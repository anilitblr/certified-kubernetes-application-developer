# Lesson 9 - Managing Ingress

- [Lesson 9 - Managing Ingress](#lesson-9---managing-ingress)
    - [9.1 Understanding Ingress](#91-understanding-ingress)
      - [Understanding Ingress](#understanding-ingress)
      - [Understanding Ingress Controllers](#understanding-ingress-controllers)
    - [9.2 Configuring the Minikube Ingress Controller](#92-configuring-the-minikube-ingress-controller)
      - [Configuring Minikube Ingress](#configuring-minikube-ingress)
      - [Demo: Running the Minikube Ingress Addon](#demo-running-the-minikube-ingress-addon)
    - [9.3 Using Ingress](#93-using-ingress)
      - [Demo: Creating Ingress](#demo-creating-ingress)
    - [9.4 Configuring Ingress Rules](#94-configuring-ingress-rules)
      - [Understanding Ingress Rules](#understanding-ingress-rules)
      - [Understanding Ingress Backends](#understanding-ingress-backends)
      - [Understanding Ingress Path Types](#understanding-ingress-path-types)
      - [Understanding Ingress Types](#understanding-ingress-types)
      - [Demo: Name-Based Virtual Hosting Ingress](#demo-name-based-virtual-hosting-ingress)
    - [9.5 Understanding IngressClass](#95-understanding-ingressclass)
      - [Understanding IngressClass](#understanding-ingressclass)
    - [9.6 Troubleshooting Ingress](#96-troubleshooting-ingress)
    - [Lesson 9 Lab Using Ingress](#lesson-9-lab-using-ingress)
    - [Lesson 9 Lab Solution Using Ingress](#lesson-9-lab-solution-using-ingress)

### 9.1 Understanding Ingress

- Explain about Ingress

#### Understanding Ingress

- Ingress is used to provide external access to internal Kubernetes cluster resources.
- To do so, Ingress uses a load balancer that is present on the external cluster.
- This load balancer is implemented by the Ingress controller.
- As an API resource, Ingress used selector labels to connecto to Pods that are used as a service endpoint.
- To access resources in the cluster, DNS must be configured to resolve to the Ingress load balancer IP.
- Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.
- Traffic routing is controlled by rules defined on the Ingress resource.
- Ingress can be configured to do the following.
  - Give Services externally-reachable URLs.
  - Load balance traffic.
  - Terminate SSL/TLS.
  - Offer name based virtual hosting.

#### Understanding Ingress Controllers

- Creating Ingress resources without Ingress controller has no effect.
- Many Ingress controllers exist:
  - nginx: https://kubernetes.github.ai/ingress-nginx/
  - haproxy: https://www.haproxy.com/blog/dissecting-the-haproxy-kubernetes-ingress-controller/
  - traefik: https://docs.traefik.io
  - kong: https://konghq.com/solutions/kubernetes-ingress/
  - contour: https://octetz.com/posts/contour-adv-ing-and-delegation

### 9.2 Configuring the Minikube Ingress Controller

#### Configuring Minikube Ingress

- Specific instructions exist to use Ingress in a full Kubernetes cluster.
- Minikube provides easy Ingress access using a Minikube addon.
- Use **minikube addon enable ingress** to enable it.
- Without it, there will be no working Ingress.

#### Demo: Running the Minikube Ingress Addon

```bash
minikube addons list
minikube addons list | grep ingress
minikube addons enable ingress
minikube addons list
kubectl get ns
kubectl get all -n ingress-ginx
kubectl get pods -n ingress-ginx
```

### 9.3 Using Ingress

#### Demo: Creating Ingress

Note: this demo continues on the demo in Lesson 8.4

```bash
kubectl get deployment
kubectl get svc nginxsvc
curl http://$(minikube ip):32000
# Create ingress
kubectl create ingress -h
kubectl create ingress nginxsvc-ingress --rule="/=nginxsvc:80" --rule="/hello=newdep:8080"
kubectl describe ingress nginxsvc-ingress
minikube ip
sudo vim /etc/hosts
    $(minikube ip) nginxsvc.info
kubectl get ingress # Wait until it shows an IP address.
curl nginxsvc.info

kubectl create deployment newdep --image=docker.io/phozzy/hello-app:latest
kubectl expose deployment newdep --port:8080
curl nginxsvc.info/hello
```

### 9.4 Configuring Ingress Rules

#### Understanding Ingress Rules

Each Ingress Rule contains the following.
- An optional host. If no host is specified, the rule applies to all inbound HTTP traffic.
- A list of paths (like /testpath). Each path has its own backend. Paths can be exposed as a POSIX regular expression.
- The backend, which consists of either a service or a resource. It is common to configure a default backend in an Ingress controller for incoming traffic that doesn't match a specific path.

#### Understanding Ingress Backends

- A service backend relates to a Kubernetes Service.
- A resource backend typically refers to cloud based object storage, where Ingress in used to provide access to storage resources.
  - See ingress-resource.yaml in the course Git repo for an example.
- Resource backends and service backends are mutually exclusive.
- A default Backend can be set to handle traffic that doesn't deal with any specific backend.

#### Understanding Ingress Path Types

- The Ingress **pathType** specifies how to deal with path requests.
- The **exact** pathType indicates that an exact match should occur.
  - If the path is set to /foo, and the request is /foo there is no match.
- The **Prefix** pathType indicates that the requested path should start with.
  - If the path is set to /, any requested path will match.
  - If the path is set to /foo, /foo as well as /foo/ will match.

#### Understanding Ingress Types

- Ingress backed by a single Service: there is a one rule that defines access to one backend Service.
  - kubectl create ingress single --rule="/files=fileservice:80"
- Simple fanout: there are two or more rules difining different paths that refer to different Services.
  - kubectl create ingress single --rule="/files=fileservice:80" --rule="/db=dbservice:80"
- Name-based virtual hosting: there are two or more rules that route requests based on the host header.
  - Make sure there is a DNS entry for each host header.
  - kubectl create ingress multihost --rule="my.example.com/files/*=fileservice:80" --rule="my.example.org/data/*=dataservice:80"

#### Demo: Name-Based Virtual Hosting Ingress

```bash
kubectl create deploy mars --image=nginx
kubectl create deploy saturn --image=httpd
kubectl get all --selector app=mars
kubectl get all --selector app=saturn

# Create service
kubectl expose deploy mars --port=80
kubectl expose deploy saturn --port=80

# Add entries to /etc/hosts
vim /etc/hosts
    $(minikube ip) mars.example.com
    $(minikube ip) saturn.example.com

# Create ingress.
kubectl create ingress multihost --rule="mars.example.com/=mars:80" --rule="saturn.example.com/=saturn:80"
kubectl edit ingress multihost # change pathType to Prefix
curl saturn.example.com
curl mars.example.com
```

### 9.5 Understanding IngressClass

#### Understanding IngressClass

- IngressClass is a relatively new resource (Kubernetes 1.22) which can be used to set a specific Ingress controller as the cluster default.
- Each ingress resource should specify a class, which refers to the cluster default IngressClass.

### 9.6 Troubleshooting Ingress

```bash
curl nginxsvc.info
kubectl get ingress
kubectl describe ingress nginxsvc-ingress
cat /etc/hosts
kubectl get ns
kubectl get all -n ingress-nginx
kubectl describe servie nginxsvc
kubectl get pods --show-labels
kubectl edit svc nginxsvc
kubectl get endpoints
curl nginxsvc.info
```

### Lesson 9 Lab Using Ingress

- Configure Ingress such that the Nginx webserver can be reached on the URL http://hostname/nginx 

### Lesson 9 Lab Solution Using Ingress

```bash
kubectl create deploy lesson-9-lab --image=nginx
kubectl get all --selector app=lesson-9-lab

# Create service
kubectl expose deploy lesson-9-lab --port=80

# Add entries to /etc/hosts
vim /etc/hosts
    $(minikube ip) lab.example.com

# Create ingress.
kubectl create ingress webserver --rule="lab.example.com/=lesson-9-lab:80" 
kubectl edit ingress webserver # change pathType to Prefix

kubectl get ingress
curl lab.example.com
```
