# Lesson 8 - Managing Networking

- [Lesson 8 - Managing Networking](#lesson-8---managing-networking)
    - [8.1 Understanding Kubernetes Networking](#81-understanding-kubernetes-networking)
    - [8.2 Understanding Services](#82-understanding-services)
      - [Understanding Services](#understanding-services)
      - [Understanding Services Decoupling](#understanding-services-decoupling)
      - [Sercices and Kube-proxy](#sercices-and-kube-proxy)
      - [Understanding Service Types](#understanding-service-types)
    - [8.3 Creating Services](#83-creating-services)
      - [Creating Services](#creating-services)
      - [Understanding Service Ports](#understanding-service-ports)
      - [Demo: Creating Services](#demo-creating-services)
      - [Accessing Applications Using Services](#accessing-applications-using-services)
    - [8.4 Using Service Resources in Microservices](#84-using-service-resources-in-microservices)
      - [Understanding Microservices](#understanding-microservices)
    - [8.5 Understanding Services and DNS](#85-understanding-services-and-dns)
      - [Understanding Services and DNS](#understanding-services-and-dns)
    - [8.6 Understanding and Configuring NetworkPolicy](#86-understanding-and-configuring-networkpolicy)
      - [Understanding NetworkPolicy](#understanding-networkpolicy)
      - [NetworkPolicy Identifiers](#networkpolicy-identifiers)
      - [Demo: Using NetworkPolicy](#demo-using-networkpolicy)
    - [Lesson 8 Lab Managing Services](#lesson-8-lab-managing-services)
    - [Lesson 8 Lab Solution Managing Services](#lesson-8-lab-solution-managing-services)

### 8.1 Understanding Kubernetes Networking

- Explain about Kubernetes network.

### 8.2 Understanding Services

#### Understanding Services

- A Service is an API resource that is used to expose a logical set of Pods.
- Services are applying round-robin load balancing to forward traffic to specific Pods.
- The set of Pods that is trageted by a Service is determined by a selector (which is a label)
- The kube-controller-manager will continuously scan for Pods that match the selector and include these in the Service.
- If Pods are added or removed, they immediately show up in the Service.

#### Understanding Services Decoupling

- Services exist independently from the applications they provide access to.
- The only thing they do, is watch for Pods that have a specific label set matching the selector that is specified in the service.
- That means that one Service can provide access to Pods in multiple Deployments, and while doing so, Kubernetes will automatically load balance between these Pods.

#### Sercices and Kube-proxy

- The kube-proxy agent on the nodes watches the Kubernetes API for new Services and endpoints.
- After creation, it opens random ports and listens for traffic to the Service port on the Cluster IP address, and next, redirects traffic to a Pod that is specified as an endpoint.
- The kube-proxy works in the background and normally doesn't require any configuration.

#### Understanding Service Types

To match networking needs in different environments, different service types are available.
- CulsterIP: the default type exposes the service on an internal cluster IP address.
- NodePort: allocates a specific node port on the node that forwards to the service cluster IP address.
- LoadBalancer: currently only implemented in public cloud.
- ExternalName: works on DNS names, redirection is happening at a DNS level, useful in migration.
- For CKAD, foucs on ClusterIP and NodePort.

### 8.3 Creating Services

#### Creating Services

- **kubeclt expose** can be used to create Services, providing access to deployments, ReplicaSets, Pods or other services.
- In most cases **kubectl exppse** exposes a Deployment, which allocates its Pods as the service endpoint.
- **kubectl create service** can be used as an alternative solution to create Services.
- While creating a Service, the **--port** argument must be specified to indicate the Service port.

#### Understanding Service Ports

- While working with Services, different ports are specified.
- **targetport**: the port on the application (container) that the service addresses.
- **port**: the port on which the Service is accessible.
- **nodeport** the port that is exposed externally while using the nodePort Service type.

#### Demo: Creating Services

```bash
kubectl create deployment nginxsvc --image=nginx
kubectl scale deployment nginxsvc --replicas=3
kubectl get all
kubectl expose deployment nginxsvc -h
kubectl expose deployment nginxsvc --port=80
kubectl get all
kubectl describe svc nginxsvc # look for endpoints
kubectl get svc nginxsvc o yaml
kubectl get svc
kubectl get endpoints
```

#### Accessing Applications Using Services

```bash
minikube ssh
curl http://svc-ip-address
exit
kubectl edit svc nginxsvc

---
  protocol: TCP
  nodePort: 32000
type: NodePort

kubectl get svc
curl http://$(minikube ip):32000 # From host machine.
```

### 8.4 Using Service Resources in Microservices

#### Understanding Microservices

- In a microservices architecture, different frontend and backend Pods are used to provide the application.
- Backend Pods (databases) can be exposed internally only using the ClusterIP Service type.
- Fronten Pods (webservers) can be exposed for external access, using the NodePort Service type.

### 8.5 Understanding Services and DNS

#### Understanding Services and DNS

- Exposed Services automatically register with the Kubernetes internal DNS.
- With Services exposing themselves on dynamic ports, resolving Service names can be challenging.
- As a solution, the courDNS service is included by default in Kubernetes, ans this DNS service is updated every time a new service is added.
- As a reslut, DNS name lookup from within one Pod to any exposed Service happens automatically.

```bash
kubectl run testpod --image=busybox -- sleep 3600
kubectl get svc
kubectl get svc,pods -n kube-system
kubectl exec -it testpod -- cat /etc/resolv.conf
kubectl exec -it testpod -- nslookup nginxsvc
```

### 8.6 Understanding and Configuring NetworkPolicy

#### Understanding NetworkPolicy

- By default, there are no restrictions to network traffic in K8s.
- Pods can always communicate, even if they're in other Namespaces.
- To limit this, NetworkPolicies can be used.
- NetworkPolicies need to be supported by the network plugin though .
  - The weave plugin does NOT support network policies!
- If in a policy there is no match, traffic will be denied.
- If no NetworkPolicy is used, all traffic is allowed.

#### NetworkPolicy Identifiers

- In NetworkPolicy, three defferent identifiers can be used.
  - **Pods**: (podSelector) node that a Pod cannot block access to ifself.
  - **Namespaces**: (namespaceSelector) to grant access to specific namespaces.
  - **IP blocks**: (ipBlock) notice that traffic to and from the node where a Pod is running is always allowed.
- When devining a Pod - or namespace-based NetworkPolicy, a selector lablel is used to specify what traffic is allowed to and from the Pods that match the selector.

#### Demo: Using NetworkPolicy

```bash
cd labs/
vim nwpolicy-complete-example.yaml
kubectl apply -f nwpolicy-complete-example.yaml
kubectl expose pod nginx --port:80
kubectl describe networkpolicy
kubectl exec it busybox -- wget --spider --timeout=1 nginx # will fail.
kubectl label pod busybox access=true
kubectl exec -it busybox -- wget --spider --timeout=1 nginx # will work.
```

### Lesson 8 Lab Managing Services

- Configure a Service for the Nginx Deployment you have created earlier. Ensure that this service makes Nginx accessible through port 80, using the ClusterIP type. Verify that it works.
- After making the Service accessible this way, change the type to NodePort and expose the Service on port 32000.
- Verify the Service is accessible on this node port.

### Lesson 8 Lab Solution Managing Services

```bash
kubectl create deploy lesson-8-lab --image=nginx --replicas=3
kubectl get deploy
kubectl expose deploy lesson-8-lab --port=80
kubectl get all --selector app=lesson-8-lab
kubectl get all --selector app=lesson-8-lab -o wide
kubectl get endpoints
minikube ssh
curl <ClusterIP>
exit
kubectl edit svc lesson-8-lab

---
  protocol: TCP
  nodePort: 32000
type: NodePort

curl $(minikube ip):32000
```
