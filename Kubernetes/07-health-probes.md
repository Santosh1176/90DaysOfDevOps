# Probes in kubernetes

Kubernetes employes many probes — Health checks to maintain the actual state of the cluster in sync with the desired state. Such probes are mainly integrated with th Pod to check at some frequesnt intervals to chek the health of the Pods. 

## Liveness Probe
A Liveness probe allows the Kubelet to reshedule the Pod incase of a running Pod but unable to make further progress in its funtioning. This usualy happens when a Pod is ruuning for a long period and due to some reason gets in a non-recoverable broken state, ehich can not recover unles restarted. However whenever a *Startup probe* is configured on a Pod, to avoid interferences of different configurations, the Kubelet disables the Liveness probe untill the Statup probe succeeds. Thus avoiding any accidental killing ofthe Pod befor the Pod is Up and Running.

```yaml
example-liveness-probe.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:  # Commands to execute during Liveness Probe. If the COmmands run succeeds with a return vale of 0 (Success)
        - cat     # a Non-zero return is resulted in a non-healthy Pod and a signal to Kublet to Kill the Pod and create a New one.
        - /tmp/healthy
      initialDelaySeconds: 5  # Delay to allow Pod to get into running state, to perform first probe
      periodSeconds: 5   # Interval for recurring Liveness probe to be conducted
```
### Liveness of a Http service.
```yaml
   livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```
- Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.

### TCP liveness probe
```yaml
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
- The above example uses both readiness and liveness probes. First the checks for Readiness is executed for 5 seconds after the Container starts. This will attempt to connect to the goproxy container on port 8080. If the probe succeeds, the Pod will be marked as ready. Then the Liveness probe will run the first probe after 15 seconds after Container starts, this will attempt to connect to the `goproxy` container on port 8080 for results of teh probe. 


## Readiness Probe
AS discussed above, Readiness probe check for the Pod to be in a Ready condition when its first created, so as to serve traffic. A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services. When the probe succeeds enough times (threshold), it means that the pod can serve traffic.

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
  ```

  - Readiness and liveness probes can be used in parallel for the same container. Using both can ensure that traffic does not reach a container that is not ready for it, and that containers are restarted when they fail.



# Resources:
- [Kubernetes Readiness Probes | Practical Guide](https://komodor.com/learn/kubernetes-readiness-probes-a-practical-guide/)
- [Kubernetes Docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes)
