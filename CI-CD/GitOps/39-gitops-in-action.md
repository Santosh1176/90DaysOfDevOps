# GitOps in Action

In this session, which is a continuation from the laste one, we are going to enable a fully automated deployment model based on **GitOps** principles. We shall be using a Opensource GitOps tool called **Flux2** in this workflow.

## Installing Flux2
Inorder to implement GitOps, we need to install **Flux** on our system 


Installing Flux becomes a smooth ride with some help from the [Flux getting started guide](https://fluxcd.io/flux/get-started/). After checking for all prerequisites, like having the Kubernetes cluster running, installing FluxCLI, and exporting the GitHub credentials. We can use the Flux Bootstrap command to install Flux on our system.

We will use the following FLUX CLI command to Bootstrap the Flux components on our system:

```bash
$ flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=gitops-demo \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
  --components-extra=image-automation-controller,image-reflector-controller
  ```
This will create a GitHub repository by the name `gitops-demo`, generate manifests, and install various FluxCD components in the `Flux-system` namespace. The `--components-extra` argument here, will install additional Flux components, `image-automation-controller` and `image-reflector-controller` to our cluster that are required for performing some Image updates and Automation required for our CD workflow.

The output would be like:

  ```bash
  ► connecting to github.com
✔ repository created
✔ repository cloned
✚ generating manifests
✔ components manifests pushed
.
.
.
✔ sync manifests pushed
► applying sync manifests
◎ waiting for cluster sync
✔ bootstrap finished
```
Check for all the Flux components installed:

```bash
$ k get deploy -n flux-system 
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
helm-controller               1/1     1            1           2d4h
image-automation-controller   1/1     1            1           2d4h
image-reflector-controller    1/1     1            1           2d4h
kustomize-controller          1/1     1            1           2d4h
notification-controller       1/1     1            1           2d4h
source-controller             1/1     1            1           2d4h
santosh@~:$ 
```
Apart from deployments, Flux installs configMaps, Secrets, ClusterRoles, ClusterRoleBindings, etc in the `flux-system` namespace. Once all the components are up and running, we can clone the `gitops-demo` repo and cd into `cluster/my-clusters` dir. Then we can move ahead by adding our application repository to Flux.

Now, we need to generate a few Custom Resource Definitions (CRD's)  to get our GitOps workflow with Flux to work. First, we generate the yaml config creating a new component in Flux, known as `GitRepository`, which points Flux to our application repo's main branch:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: bookstore
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: main
  #This is the main application repository we want to link with FluxCD
  url: https://github.com/Santosh1176/bookstore-api/
  ```
Commit and push the changes.

Next, we need to build and apply Kustomize configurations from our application repository.

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: bookstore
  namespace: flux-system
spec:
  interval: 5m0s
  # A Directory where all application manifests with Kustomization are available
  path: "./kustomize/"
  prune: true
  sourceRef:
    kind: GitRepository
    name: bookstore
```  
I have pointed to a `./kustomize` directory on my [Bookstore application repository](https://github.com/Santosh1176/bookstore-api) in the Kustomization CRD to install the manifests. We commit and push the changes.

Now, we should see our application deployed on our cluster. So far, Flux has applied Kustomize configs available in the repo flux and is linked to my Bookstore application repository. Now, we need to configure Flux to watch for any new image build and pull the latest among the tagged images based on the timestamp we configured earlier in GitHub Action.

In order to achieve this, we need to tell Flux which image repository to look for. This we do by adding an `ImageRepository` CRD to our `gitops-demo` repository:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: bookstore
  namespace: flux-system
spec:
  #This is the registry we want to watch for
  image: docker.io/santoshdts/bookstore
  interval: 1m0s
```

In our case, it's a public repository. But, if you have a private registry, it is advised to generate a secret of type `docker-registry` and configure Flux to use it by specifying it in the above manifest by adding:

```yaml
spec:
  secretRef:
    name: <Secret Name>
```
Once we have our **Imagerepository** resources created and pushed to the `gitops-demo` repo. It's time to tell Flux which style of versioning to use while filtering for image tags. For this we create an **ImagePolicy** CRD in the `gitops-demo` repo with the following config:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata: 
  name: bookstore
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: bookstore
  filterTags:
    # The ReGex pattern matches our rule, Tagging an image with <Branch>-<Git-SHA>-<UNIX-TimeStamp>
    pattern: '^main-[a-fA-F0-9]+-(?P<ts>.*)'
    extract: '$ts'
  policy:
    numerical:
      order: asc
```flux get image policy
We can provide patterns as per our requirements in `spec.filters.pattern` field. [For example](https://fluxcd.io/flux/components/image/imagepolicies/#examples), instead of patterns, we can specify a policy like so:

```yaml
spec:
  imageRepositoryRef:
    name: bookstore
  policy:
    semver:
      range: 1.0.x
```
> But for this, we need to configure our GitHub Action to trigger a change in git Tag.

Once, the ImagePolicy is pushed to our `github-demo` repo. We can ask Flux to apply the changes locally by reconciling with the source:

`flux reconcile kustomization flux-system --with-source`.

Once the reconciliation is done, we should see the image tags from our container registry:

```bash
$ flux get image repository bookstore 
NAME      LAST SCAN                 SUSPENDED READY MESSAGE                       
bookstore 2023-01-21T21:29:45+05:30 False     True  successful scan, found 4 tags
```

We can also see the tags that match our pattern specified in the *ImagePolicy*:
```bash
$ flux get image policy bookstore 
NAME            LATEST IMAGE                                            READY   MESSAGE                                                                                     
bookstore       docker.io/santoshdts/bookstore:main-94e4d51a-1674303038 True    Latest image tag for 'docker.io/santoshdts/bookstore' resolved to: main-94e4d51a-1674303038
```

This is the matching tag we desired, `bookstore:main-94e4d51a-1674303038`. Still, we need to tell Flux to which GitRepository to update the tags to. We Do this by adding an **ImageUpdateAutomation** CRD, like so:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m0s
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: dtshbl@gmail.com
        name: fluxcdbot
      messageTemplate: '{{range .Updated.Images}}{{println .}}{{end}}'
    push:
      branch: main
  update:
    path: ./clusters/my-cluster
    strategy: Setters
```
There's a special syntax to tell our deployment manifest to watch for the specific *ImagePolicy*. For this, we need to edit the main deployment file in our `bookstore` repository. 

![Tag Deploy](./images/deploy-tag.png)

The `# {"$imagepolicy": "flux-system:bookstore"}` tag in front of the image file, tells the deployment to track a specific *ImagePolicy* resource. Here, I've configured it to look for: `flux-system: bookstore` it's a combination of `Namespace: ImagePolicy`. More on this patching mechanism is explained [here](https://fluxcd.io/flux/guides/image-update/#configure-image-update-for-custom-resources).



With these resources updated in our `gitops-demo` and `bookstore` repo. Now, our deployment manifest should be updated with the latest image tags based on the `UNIX timestamp`. But, we need to provide write access to the `gitops-demo` repository to write to our application repository. This can be achieved by generating a **Deploy key** from our management repository. We can generate the key using the `flux cli`. For this, we would need a *Flux Secret* which is available at `.spec.secretRef.name` from our GitRepository resource. And, we can generate the Secret by applying the following command:

```bash
flux create secret git \
--namespace=flux-system \
flux-system \
--url=ssh://git@github.com/Santosh1176/bookstore-api
```
This would generate a public key, which we need to enter into our `gitops-demo` repo. We need to ensure the **Allow write access is check**. 

![Deploy key](./images/deploy-key.png)

Once this deploy key for write access is configured on our repository, we can commit to our application code, and witness within minutes the newly build image being pulled from the DockerHub into our deployment namespace.

we can check this:

```bash
$ k get deploy -n frontend-api bookstore-frontend -oyaml | grep -i image
        image: docker.io/santoshdts/bookstore:main-94e4d51a-1674303038
        imagePullPolicy: IfNotPresent
```

You can clearly see the image we have mentioned in our Deployment manifest as shown above (the image used for tagging with ImagePolicy) is different from the currently deployed one, as I have pushed some minor updates to the application. Hence, the change in the Image Tag.

With this automation for our Dev/Staging teams complete, we can move in a similar fashion to put in place. A similar workflow for our Prod teams can be created. The only alterations, we need to make would be to the image tagging, which would be based on more realistic real-world scenarios like using **server**, etc. The GitHub Action here would be triggered on any changes made with Git Tags and/or any PR's merged.

> The above workflow can be built with more standardised repository structure which could be designed based on separation of concerns between Dev and Ops team. Though there isn’t a hard rule on structuring our git repository. However, this detailed [discussion](https://youtu.be/vLNZA_2Na_s) on the topic may help you on that.



# Resources:
- [Automate image updates to Git](https://fluxcd.io/flux/guides/image-update/)
- [Flux Bootstrap](https://fluxcd.io/flux/get-started/)
- [Github-Actions-Demo with Flux by Kingdon Barret](https://github.com/kingdonb/github-actions-demo)
- [GitOps: Core Concepts & Ways of Structuring Your Repos - WeaveWorks](https://youtu.be/vLNZA_2Na_s)