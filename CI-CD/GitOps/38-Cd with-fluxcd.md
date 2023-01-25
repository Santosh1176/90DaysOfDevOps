# Hands-on tutorial on CI/CD integrating GitHub Actions for CI and FluxCD for CD

In this session a part of implementing GitOps functionality to our application, I will be working on applying a Continuous Integration strategy using IthHUb Actions and for Continuous Deployment, I'll be using [FluxCD](https://fluxcd.io/flux/), a CNCF-hosted GitOps tool. I already have a repository of the [application](https://Santosh1176/bookstore-api) hosted on GitHub which I will be working on.

Our goal in this two-step exercise would be:
- Build a GitHub Action, like the last one. That will push a new versioned image to the Docker hub when a new commit is pushed to the repo.
- Install Flux on the Kubernetes Cluster using the FluxCD's bootstrap command.
- Apply Flux Image Automation tools required to watch for any changes in the Image.
- Configure our Deployment to pull the latest versioned image from the Docker hub.

# Build a GitHub Action to build a versioned image

This is very similar to the GitHub Action, we worked on in the [earlier session](../github-actions/35-docker-push.md). We will be tweaking some of the functionality of our Action to build and push a tagged image to DockerHub.

![Script to produce a version](./images/action-ver.png)

This action gets triggered whenever a commit is pushed to **any branch**. The script you see from lines 18 to 30 does is collect the *Branch Name*, *Commit SHA*, and the *UNIX Timestamp*. This is used for tagging when the *Docker Build and Push* action is triggered. 

![Docker Build and Push](./images/docker-build-push.png)

It's just that simple. Now, if we push a commit to this repository, the action will get triggered and a new image with a tag formatted in `BRANCH-GIT_SHA-TIMESTAMP`.

![GitHUb Action](./images/action-succuss.png)

From the above image, we can see the action was run successfully and a newly tagged image was pushed to DockerHub as desired. Let's check the Docker Hub for the new Image with the new tag.

![New Image](./images/new-image.png)

You can see, the image was tagged with `main-94e4d51a-1674303038` as we had configured in the script that ran in our GitHub Action. 

Now, as we are getting a new image with a tag corresponding with a UNIX timestamp. we can leverage this and use it in our FluxCD configuration.

This is the first step of our GitOps implememntation. The artifact built and pushed to an OCI registry will be used in the Continuous Delivery workflow, which we shall be implementing in the next session levereging the features of Flux2.

