# Basic Networking in Kubernetes.



## Kubernets has laid out requirements for Pod networking

 

Each pod has its own IP address, and Kubernetes expects that all pods within the cluster can communicate with each other using that IP address. This is true regardless of the range and subnet assigned to each pod.

  

How do we address these issues with the above-mentioned requirements?



In the NETWORKING section, we discussed  each pod needs an IP address, a bridge, NAT rules, and routes in the routing table. In order to simplify and standardize this process. Kubernetes can come up wityha a concept called [Container Networking Interface](cni.org) to implemnet the same networking configurations as desired by Kubernetes to operate effiiently.  



# Service in Kubernets



However, we would rarely want a Pod to communicate with another pod directly. Instead, we would always use a concept called **Service** for inter pods communication within a Node. As discussed above the Kubernetes requirements with Pods, Pods running in one moment in time could be different from the set of Pods running that application a moment later. Thus, a concept called **Service** is employed, which is an abstraction and a policy by which to access them. A Services provide discovery and routing between pods.



One of the Configurations of a Service is a Cluseter-wide accessible entity, Known as **CLUSTER IP**. This Type of Service suites well for enabling the communication between pods within the Cluster.



To enable the traffic outside the Cluster to access the Pods within the CLuster, we configure a Service of type **NodePort**. Similar to the Cluster IP, even this Service gets its own IP Address and all the Pods with the cluster can access this service, additionally, It exposes a Port on all nodes ( HighPort: 31767 to 36000). A Service of NodePort enables the Traffic from outside the cluster to access the Pods inside our cluster.

This is possible in Kubernetes by the works of components named: `Kubelet` and `Kube-Proxy`. `Kubelet` is responsible for creating a new Pod in communication with `Kube-apiserver` and invoking the pre-configured **CNI** plugin to configure all the networking configurations on that Pod. Similarly, each Node runs another componet known as `KubeProxy`. Kubeproxy watches for the chanes in the cluster through Kube-apiserver and swings into action whenever an new `Service` is create inside a Cluster. Unlike Pods, Services are not estricted to Nodes and are Clusetr-wide resouce and are accessible from all the Nodes.

> In rality Service in Kubernetes are not like Containers and Pods, as they are associated with any Interfaces, Namespaces or even the IP address. It can be termed as an Virtual object inside the Kubernetes.

Whenever we define an `Service` object, its defined an Pre-defined IP address from a range. To view the IP range assigned by the `kube-proxy, run the following command:
```bash
ps aux | grep kube-api-server

# Expect an output similar to 
kube-apiserver --authorization-mode=Node,RBAC --service-cluster-op-range=10.93.0.0/12
```
Which effectively gives an Ip range from 10.93.0.0 to 10.111.255.255.



, The `Kube-Proxy` component running on each Node gets those addresses and configures the IP forwording rules on each cluster. Interpreting, that any traffic landing on the Ip address and its Port of that perticular Service should be forwqrded to a *defined Pod*, making it accessible from any Node in tyhe Cluster. 

`Kube-proxy` component works with different concepts for forming rules for directing the traffic from a Service to a sepcif Pod connected to that Service.

Some common ways of routing the traffics are based on `Userspacfe` where `kube-proxy` listens of a different Ports, by creating `ipvs` rules or the third and the most commonly used ways is the useof `iptables` 


> if `kube-proxy --proxy-mode [userspace | iptables | ipvs]` is not set to our desired state. `kube-proxy` defaults to `iptables` mode. for redirecting the traffic.



> An abstract way to expose an application running on a set of Pods as a network service.

## Creating a Service
A Service in Kubernetes is a REST object, similar to a Pod. Like all of the REST objects.
An example of creatinga servicxe using definition file is as follows:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp   # Selects a Pod Pod with the app.kubernetes.io/name=MyApp label.
  ports:
    - protocol: TCP
      port: 80   # Port on which the Service listens to.
      targetPort: 9376   # Redirects the traffic to the TargetPort of the Pod
```

Port definitions in Pods have names, and you can reference these names in the targetPort attribute of a Service. For example, we can bind the targetPort of the Service to the Pod port in the following way:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        *name: http-web-svc*

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    *targetPort: http-web-svc*
```



# Resources fr Service:
- [Kubernetes Documents](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Cloud Native Wiki](https://www.aquasec.com/cloud-native-academy/kubernetes-101/kubernetes-services/)
- [Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)




# Shortfalls of Service



# Ingress Controllers as a remedy 



# confogMaps for storing various Ingress controller-required config data.





# Service is still the main interface



# Service Resources



# Ingress Rules for Routing