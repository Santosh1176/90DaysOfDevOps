# Authorization in Kubernetes

In order to interact with the Kubernetes cluster, a user must be first [Authenticated](./11-tls-authentication.md) before the request can be authorized (granted permission to access). Once, the user is authenticated, the requests are further processed for **Authorization** of the request.

As we are aware, the `kube-apiserver` is the component that handles all the API calls made to the cluster. It evaluates the request attributes based on the available policies, then decides whether to allow or deny the request. 

> requests on specific fields of specific kinds of objects are handled by another component in Kubernetes known as  **Admission Controllers**.

To learn more about the API request attributes reviewed by Kubernetes follow this [official link](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#review-your-request-attributes) :

Kubernetes supports mainly four types of Authorization:
- [Node Authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/): It specifically authorizes the requests made by the `kubelet`.
- [Attribute-based access control (ABAC)](https://kubernetes.io/docs/reference/access-authn-authz/abac/): This method defines access control to users through the use of policies that combine attributes together like (user attributes, resource attributes, and object, environment attributes, etc).
- [Role-based access control **(RBAC)**](https://kubernetes.io/docs/reference/access-authn-authz/rbac/): This method is used to grant access to resources based on the roles of individual users within an organization. RBAC uses the `rbac.authorization.k8s.io` API group to drive authorization decisions, allowing admins to dynamically configure permission policies through the Kubernetes API.
- [Webhook](https://kubernetes.io/docs/reference/access-authn-authz/webhook/): In this method, an application implementing WebHooks will POST a message to a URL when certain things happen. 

In this article, we will be focusing on the RBAC method.

Kubernetes provides an interesting concept to check our access to various resources in the cluster by `kubectl auth can-i` commands:
for example: 
```bash
santosh@~*$:k auth can-i create pods  -n kube-system 
yes  # The answer by the API server
```
This shows, I ( as a user of this cluster) can create pods in the `kube-system` namespace. Similarly, you can use this command to check for other resources with different verbs. like:
```bash
santosh@~*$:k auth can-i patch secret -n kube-system 
yes   # The answer by the API server
```
# Role-Based Access Control — RBAC

The RBAC, as outlined above grant access to resources based on the roles of individual users within an organization. RBAC API declares four kinds of Kubernetes objects: `Role`, `ClusterRole`, `RoleBinding`, and `ClusterRoleBinding`. You can describe objects, or amend them just like any other Kubernetes object, using `kubectl`.

RBAC Role or ClusterRole contains rules that represent a set of purely additive permissions ( There are no *Deny* permissions). While `Roles` are namespaced resources and `CLusterRoles` are Cluster-wide resources. However, `ClusterRoles` can be employed to define permissions on any `namespaced` object also.

## Examples of Role / ClusterRoles from the manifests in a Kind Cluster.

- An example of a manifest for a `Role`:


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: kubeadm:bootstrap-signer-clusterinfo
  namespace: kube-public
rules:
- apiGroups:
  - ""  # "" indicates the core API group
  resourceNames:
  - cluster-info  # list of names that the rule applies to.
  resources:
  - configmaps
  verbs:
  - get   # Verbs 
```

An example of a manifest of a `ClusterRole` 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
rules:
- apiGroups:
  - '*'
  resources:
  - '*'    #  '*' wildcard defines All resources
  verbs:
  - '*'    #  '*' wildcard defines All verbs
- nonResourceURLs:  # NonResourceURLs is a set of partial urls that a user should have access to.
  - '*'
  verbs:
  - '*'
```

A More complex ClusterRole:
```yaml
k get clusterrole system:node-problem-detector -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:node-problem-detector
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - ""
  - events.k8s.io
  resources:
  - events
  verbs:
  - create
  - patch
  - update
```
Once the Role / ClusterRole is created, apply these roles to a user or a group of users. An object named `RoleBinding` is employed. Similar to Roles and ClusterRoles, `RoleBinding` is Namespaced and for Cluster-scoped resources, `ClusterRoleBinding` is used.

A RoleBinding may reference any Role in the same namespace. Alternatively, a RoleBinding can reference a ClusterRole and bind that ClusterRole to the namespace of the RoleBinding. If you want to bind a ClusterRole to all the namespaces in your cluster, you use a ClusterRoleBinding.

An Example of `RoleBinding`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects: # Subjects holds references to the objects the role applies to.
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

An Example from a Kind Cluster:

```yaml
$ k get rolebindings -n kube-system kube-proxy -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kube-proxy
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kube-proxy
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:kubeadm:default-node-token
```

and for `ClusterroleBinding`

```yaml
$:k get clusterrolebindings system:coredns -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:coredns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
```
RoleBinding is an immutable object. That means, whenever we bind a role we cannot change the Role or ClusterRole that it refers to. If you try to change a binding's `roleRef`, you get a validation error. If you do want to change the `roleRef` for binding, you need to remove the binding object and create a replacement. 

We can also RoleBindings to Kubernetes objects as well:

```yaml
subjects:
- kind: ServiceAccount
  name: test-1
  namespace: dev-1
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```
Above example, Binds a ClusterRole `secret-reader`. Thus, ServiceAccount named `test-1` in `dev-1` namespace can "get", "watch", "list" Secret.

Default RBAC policies grant scoped permissions to control-plane components, nodes, and controllers, but grant **no permissions** to service accounts outside the kube-system namespace.

## Imperative way of creating a Role / ClusterRole:

- `kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods`

- `kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods`

## Imperitive way of creating a RoleBinding / ClusterRoleBinding:

`kubectl create rolebinding bob-admin-binding --clusterrole=admin --user=bob --namespace=acme`

`kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=cluster-admin --user=root`



> To know more about the fields used in any of the manifests, one can use `kubectl explain` command. For example in the Role / RoleBinding scenario. We can use `kubectl explain clusterrole.rules.nonResourceURLs`.





# Resource:
- [Kubernetes Authorization Overview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)