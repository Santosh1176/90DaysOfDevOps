# Pod Lifecycle and Phases

## Pod Lifecycle
Pods are considered to be relatively ephemeral (rather than durable) entities. Pods are created, assigned a unique ID (UID), and scheduled to nodes where they remain until termination (according to restart policy) or deletion.

Pods do not, by themselves, self-heal, if a Pod fails, it is deleted and a new one is created in its place. As its assigned a UID, a Pod is never rescheduled on a different node. Instead that Pod can be replaced by a new identical configuration *but* with a different UID, if rescheduling on a different node is desired for some reason.




## Pod Phase
Pod status object includes a phase field. This phase-field tells Kubernetes and us that wherein the execution cycle a pod is:

*Pending*: Accepted by the cluster, containers are not set up yet.
*Running*: At least one container is in a running, starting, or restarting state.
*Succeeded*: All of the containers exited with a status code of zero; the pod will not be restarted.
*Failed*: All containers have terminated and at least one container exited with a status code of non-zero.
*Unknown*: The state of the pod can not be determined.

*The phase is not intended to be a comprehensive rollup of observations of container or Pod state*


# Resources:
- [Kubernetes: Lifecycle of a Pod](https://dzone.com/articles/kubernetes-lifecycle-of-a-pod)
- [Kubernetes in Action Book - Chapter 6](https://livebook.manning.com/book/kubernetes-in-action-second-edition/chapter-6/v-4/)