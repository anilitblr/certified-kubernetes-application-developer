# Lesson 5 - Managing Pod Basic Features

- [Lesson 5 - Managing Pod Basic Features](#lesson-5---managing-pod-basic-features)
    - [5.1 Understanding Pods](#51-understanding-pods)
      - [What is Pod?](#what-is-pod)
      - [Understanding Naked Pod Disadvantages](#understanding-naked-pod-disadvantages)
      - [Managing Pods with kubectl](#managing-pods-with-kubectl)
    - [5.2 Understanding YAML](#52-understanding-yaml)
      - [What is YAMl](#what-is-yaml)
      - [Basic YAML Manifest Ingredients](#basic-yaml-manifest-ingredients)
      - [Container Components](#container-components)
      - [Example YAML File](#example-yaml-file)
    - [5.3 Generating YAML Files](#53-generating-yaml-files)
      - [Using Kubernetes the Declarative Way](#using-kubernetes-the-declarative-way)
      - [Generating YAML Files](#generating-yaml-files)
      - [Using Kubernetes the Declarative Way](#using-kubernetes-the-declarative-way-1)
    - [5.4 Understanding and Configuring Multi-container Pods](#54-understanding-and-configuring-multi-container-pods)
      - [Understanding Multi-container Pods](#understanding-multi-container-pods)
      - [Understanding the Sidecar Scenario](#understanding-the-sidecar-scenario)
      - [Demo: Running a Sidecar Container](#demo-running-a-sidecar-container)
    - [5.5 Working with Init Containers](#55-working-with-init-containers)
      - [Understanding Init Containers](#understanding-init-containers)
    - [5.6 Using Namespaces](#56-using-namespaces)
      - [Understanding NameSpace](#understanding-namespace)
      - [Using NameSpace](#using-namespace)
      - [Demo: Using NameSpaces](#demo-using-namespaces)
    - [Lesson 5 Lab Managing Pods](#lesson-5-lab-managing-pods)
    - [Lesson 5 Lab Solution Managing Pods](#lesson-5-lab-solution-managing-pods)

### 5.1 Understanding Pods

#### What is Pod?

- A Pod is an abstraction of a server.
  - It can run multiple containers within a single NameSpace, exposed by a single IP address.
- The Pod is the minimal entity that can be managed by Kubernetes.
- From a container perspective, a Pod is an entity that runs typically one or more containers by using container images.
- Typically, Pods are only started through a Deployment, because "naked" Pods are not rescheduled in case of a node failure.
- In CKAD, you will have to deal with naked Pods.

#### Understanding Naked Pod Disadvantages

- Naked Pods are not rescheduled in case of failure.
- Rolling updates don't apply to naked Pods; you can only bring it down and bring it up again with the new settings.
- Naked Pods connot be scaled.
- Naked Pods cannot be replaced automatically.

#### Managing Pods with kubectl

- Use **kubectl run** to run a Pod based on a default image.
  - **kubectl run mynginx --image=nginx**
- **kubectl get pods [-o yaml]**
  - The **-o yaml** option provides insight in all the Pod parameters.
- **kubectl describe pods** shows all details about a Pod, including information about containers running within.
  - For instance, **kubectl describe pods mynginx**
```bash
kubectl run -h
kubectl run myfirstnginx --image=nginx
kubectl get pods
kubectl get pods myfirstnginx -o yaml
kubectl describe pods myfirstnginx
```

### 5.2 Understanding YAML

#### What is YAMl

- YAML is a human-readable data-serialization language.
- It is perfect for DevOps to create input files to manage objects into different systems like Kubernetes and Ansible.
- It uses indentation to identation to dentify relations.
- When combined with GitHub, its an excellent way to manage and update configurations in a structured way.
- To make working with YAML in **vim** easier, include the following in ~/.vimrc
  - **autocmd FileType yaml setlocal ai ts=2 sw=2 et**

#### Basic YAML Manifest Ingredients

All of the YAML manifest Ingredients are defined in the API.
- **apiVersion**: specifies which version of the API to use for this object.
- **kind**: indicates the type of object (Deployment, Pod, etc...)
- **metadata**: contains administrative information about the object.
- **spec**: contains the specifics for the object.
- Use **kubectl explain** to get more information about the basic properties.

#### Container Components

In the containers spec, different parts are needed.
- **name**: the name of the container.
- **image**: the image that should be used.
- **command**: the command the container should run.
- **args**: arguments that are used by the command.
- **env**: environment variables that should be used by the container.
- These are all a part of the pod.spec.container.spec, which can be checked with **kubectl explain**

#### Example YAML File

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox
    command:
      - sleep
      - "3600"
  - name: nginx
    image: nginx
```
```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain --recursive pod.spec

cd labs/
vim busybox.yaml
kubectl create -f busybox.yaml
kubectl get pods
kubectl delete -f busybox.yaml
kubectl apply -f busybox.yaml # apply creates if doesn't exist, updates if exists.
```

### 5.3 Generating YAML Files

#### Using Kubernetes the Declarative Way

- The imperative way of working with Kubernetes is where you create everything from the command line.
- In the declarative way, YAML files are typically stored in a Git repository, which fits well into a DevOps strategy.
- The declarative approach is nice, as it allows you to create multiple resources from a single YAML file.

#### Generating YAML Files

- Do NOT write YAML files, generate them.
- To generate YAML files, use **--dry-run=client -o yaml > my.yaml** as an argument to the **kubectl run** and **kubectl create** commands (covered later)
- **kubectl run mynginx --image=nginx --dry-run=client -o yaml > mynginx.yaml**

#### Using Kubernetes the Declarative Way

- **kubectl create -f my.yaml** is used to create resources from YAML.
- **kubectl apply -f my.yaml** will create a resource if it doesn't exist yet, and modify if it already exists and has been created with **kubectl apply** earlier.
- **kubectl replace -f my.yaml** will replace a resource with the new configuration as in the YAML file.
- **kubectl delete -f my.yaml** will remove everything specified in the YAML file.
```bash
kubectl run mynginx --image=nginx -o yaml --dry-run=client > mynginx.yaml
kubectl run -h | less
kubectl run mybusybox --image=busybox -- sleep 3600
kubectl run mybusybox --image=busybox --dry-run=client -o yaml -- sleep 3600 > mybox.yaml
kubectl create -f mybox.yaml
kubectl get pods
```

### 5.4 Understanding and Configuring Multi-container Pods

#### Understanding Multi-container Pods

- The one-container Pod is the standard.
- Single container Pods are easier to build and maintain.
- Specific settings can be used that are good for a specific workload.
- To create applications that consist of multiple containers, micorservices should be defined.
- In a micorservices, different independently managed Pods are connected by resources that can be provided by Kubernetes.
```bash
cd labs/
vim multicontainer.yaml
kubectl create -f multicontainer.yaml
```
- There are some defined cases where you might want to run multiple containers in a Pod.
  - Sidecar container: a container that enhances the primary application, for instance for logging.
  - Ambassador container: a container that represents the primary container to the outside world, such as a proxy.
  - Adapter container: used to adopt the traffic or data pattern to match the traffic or data pattern in other application in the cluster.
- Sidecar container, etc, are not defined by specific Pod prperties; from a Kubernetes API resource, it's just a multi-container Pod.

#### Understanding the Sidecar Scenario

- A sidecar container is providing additional functionality to the main container, where it makes no sense running the functionality in a separate Pod.
- Think of logging, monitoring, and syncing.
- The essence is that the main container and the sidecar container have access to shared resources to exchange information.
- Istio service mesh injects sidecar containers in Pods to enable traffic management.
- Often, shared volumes are used for this purpose (discussed in-depth later)

#### Demo: Running a Sidecar Container

```bash
cd labs/
vim sidecar.yaml
kubectl create -f sidecar.yaml
kubectl exec -it sidecar-pod -c sidecar -- /bin/bash
yum install -y curl
curl http://localhost/date.txt
```

### 5.5 Working with Init Containers

#### Understanding Init Containers

- An init container is an additional container in a Pod that completes a task before the "regular" container is started.
- The regular container will only be started once the init container has been started.
- If the init container has not run to completion, the main container is not started.
```bash
cd labs/
vim init-example1.yaml
kubectl create -f init-example1.yaml
kubectl get pods

vim init-example2.yaml
kubectl create -f init-example2.yaml
kubectl get pods
kubectl describe pods init-demo2

kubectl delete -f init-example2.yaml
```

### 5.6 Using Namespaces

#### Understanding NameSpace

- A Linux NameSpace implements kernel-level resource isolation.
- Kubernetes offers NameSpace resources that provide the same functionality.
- Different NameSpace can be used to strictly separate between customer resources.
- NameSpaces are used to apply different security-related settings.
  - Role-Based Access Control (RBAC)
  - Quota

#### Using NameSpace

- Use **kubectl create namespace mynamespace** to create a NameSpace.
- Use **kubectl ... -n namespace** to work in a specific NameSpace.
- Use **kubectl get ... --all-namespaces** to see resources in all NameSpaces.

#### Demo: Using NameSpaces

```bash
kubectl get ns
kubectl get all -A
kubectl create ns secret
kubectl create -f busybox-ns.yaml
kubectl run secretnginx --image=nginx -n secret

kubectl run testnginx --image=nginx -n secret --dry-run=client -o yaml > test.yaml
kubectl create -f test.yaml

kubectl describe ns secret
```

### Lesson 5 Lab Managing Pods

- Create a NameSpace with the name **production**
- In this NameSpace, run an nginx web server as a Pod using the name **nginx-prod**
- Create a YAML file to make it easier to manage the resources just created.

### Lesson 5 Lab Solution Managing Pods

```bash
kubectl create ns production --dry-run=client -o yaml > lesson-5-lab.yaml
kubectl run nginx-prod --image=nginx -n production --dry-run=client -o yaml >> lesson-5-lab.yaml
kubectl create -f lesson-5-lab.yaml
kubectl get ns
kubectl get pod -n production
kubectl delete -f lesson-5-lab.yaml
```
