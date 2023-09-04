# Lesson 3 - Understanding Kubernetes

- [Lesson 3 - Understanding Kubernetes](#lesson-3---understanding-kubernetes)
    - [3.1 Understanding Kubernetes Core Functions](#31-understanding-kubernetes-core-functions)
      - [What is Kubernetes?](#what-is-kubernetes)
      - [Understanding Kubernetes](#understanding-kubernetes)
      - [Understanding Kubernetes Competition](#understanding-kubernetes-competition)
      - [Understanding Kubernetes Distributions](#understanding-kubernetes-distributions)
      - [Understanding Kubernetes Releases](#understanding-kubernetes-releases)
    - [3.2 Understanding Kubernetes Origins](#32-understanding-kubernetes-origins)
      - [Kubernetes Origins](#kubernetes-origins)
    - [3.3 Using Kubernetes in Google Cloud](#33-using-kubernetes-in-google-cloud)
    - [3.4 Understanding Kubernetes Management Interfaces](#34-understanding-kubernetes-management-interfaces)
      - [The Role of the Kubernetes API](#the-role-of-the-kubernetes-api)
      - [Options for Working with Kubernetes](#options-for-working-with-kubernetes)
      - [Using kubectl](#using-kubectl)
    - [3.5 Understanding Kubernetes Architecture](#35-understanding-kubernetes-architecture)
    - [3.6 Exploring Essential API Resources](#36-exploring-essential-api-resources)
      - [Understanding Essential API Resources](#understanding-essential-api-resources)
      - [Investigating API Resources](#investigating-api-resources)
    - [Lesson 3 Lab Exploring Kubernetes API Resources](#lesson-3-lab-exploring-kubernetes-api-resources)
    - [Lesson 3 Lab Solution Exploring Kubernetes API Resources](#lesson-3-lab-solution-exploring-kubernetes-api-resources)

### 3.1 Understanding Kubernetes Core Functions

#### What is Kubernetes?

- Kubernetes is "an open-source system for automating deployment, scaling and managing of containerized applications" (https://kubernetes.io)
- The word Kubernetes comes from the Greek language, and means "pilot of the ship".
- Kubernetes is currently owned by Cloud Native Computing Foundation (CNCF), an open-source foundation within the Linux Foundation.
- Kubernetes is ecosystem, providing a core solution, with many third-party add-ons provided by approved projects, focusing on different areas.
  - Networking
  - Ingress
  - Monitoring
  - Packaging
  - and much more

#### Understanding Kubernetes

- Kubernetes implements a platform to run container-based applications in a Cloud Native Computing environment.
- In Cloud Native Computing, applications are hosted by the cloud, and have no direct relation to a server.
- As a result, all that is needed by the application needs to be stored in the cloud.
- To successfully run applications in a Cloud Native environment, specific properties must be provided.
- These properties are defined in Kubernetes Resources.
- Resources are defined in the Kubernetes APIs.

#### Understanding Kubernetes Competition

- Kubernetes is open source, and as such available to everyone and any company.
- The open source nature of Kubernetes allows companies to use a very strong common code base and focus of features the specific company wants to add to that.
- For that reason, currently there is no significant competition to Kubernetes.
- There are Kubernetes distributions though.
- Docker Swarm can be considered a competing solution, but Kubernetes has make it less relevant.

#### Understanding Kubernetes Distributions

- Vanilla Kubernetes is Kubernetes directly created from the source code hosted by CNCF.
- Kubernetes distributions are adding specific funtionality and selection of solutions from the ecosystem.
  - Google Anthos
  - Red Hat OpenShift
  - SUSE Rancher
  - Canonical Kubernetes

#### Understanding Kubernetes Releases

- A new release of Kubernetes is published every 3 months.
- When a new release is published, new versions of the APIs may become available, and old features may get deprecated.
- If a feature is deprecated, adopt the new method, as the feature will go away within the next 2 releases.

### 3.2 Understanding Kubernetes Origins

#### Kubernetes Origins

- Kubernetes is based on Google Borg, the internal system that Google has been using for many years to offer access to its applications.
  - https://ai.google/research/pubs/pub4348
- Borg was alos the origin of important Linux kernel features used to containers nowadays, such an namespaces.
- Google donated the Borg specification to the Cloud Native Computing Foundation (CNCF) which was the start of Kubernetes.

### 3.3 Using Kubernetes in Google Cloud

- Navigtae to **Google Cloud Platform Console**
- Choose **Kubernetes Engine** under **COMPUTE**
- Choose **Clusters**
- Choose **CREATE**
- GKE Standard 
  - Choose **CONFIGURE**
  - Choose **My first cluster**
  - Choose **CREATE NOW**
- Choose **Actions** of **My first cluster**
- Choose **Connect**
- Click on **RUN IN CLOUD SHELL**
- Choose **Enable ephemeral mode**
- Hit **Enter** key
- Choose **Authorize**
- **kubectl get all**

### 3.4 Understanding Kubernetes Management Interfaces

#### The Role of the Kubernetes API

- An Application programming interface (API) is the interface between a client and a server.
- An API is used to standardize how to access items provided by the server.
- The Kubernetes API provides access to all that is needed to run containers in a cloud-native environment.

#### Options for Working with Kubernetes

- The **kubectl** command line utility provides convenient developer access, allowing you to run many tasks related to your applications.
- **kubectl** is the only interface that matters for CKAD.
- Direct API access using commands, such as **curl**, allows developers to address the cluster using API calls from custom scripts.
- Using direct API requests is uncommon, but can be useful for getting more insight.
- The Kubernetes Dashboard can be installed to run on the Kubernetes master node.
- Do NOT focus on Dashboard if you want to pass the CKAD exam.

#### Using kubectl

- **kubectl** is the only interface to Kubernetes that you need.
- It has a lot of options, allowing you to manage everything your application needs in a Kubernetes environment.
- Use **kubectl --help** for an overview of its options.
- For cluster administration tasks (not in CKAD), there's also the **kubeadm** utility.
- Use **kubectl create deploy myapp --image=myimage** to run your first application.
- Use **kubectl get all** to see all that this command has created.
- Get back to the **Google Cloud Shell** to execute the below commands.
```bash
kubectl --help;
kubectl create -h | less
kubectl create deploy myfirstapp --image=nginx --replicas=3
kubectl get all;
``` 

### 3.5 Understanding Kubernetes Architecture

- Explain about the Kubernetes architecture.

### 3.6 Exploring Essential API Resources

#### Understanding Essential API Resources

- After using the **kubectl create deploy myapp --image=nginx --replicas=3** command, the myapp application is started in Kubernetes.
- To run an application in Kubernetes, different API resources are used.
- Each API resource adds functionality that is required to run an application in a cloud native environment.
- Remember, in a cloud native environment the application is not directly related to any server, so all information the application needs is stored in the cloud.

#### Investigating API Resources

- The Kubernetes APIs provide different resources to run applications in a cloud-native environment.
  - Deployment: used to represent the application itself.
  - ReplicaSet: takes care of scalebility by managing application replicas (instances)
  - Pods: adds features required in cloud to run the actual containers that contain the application.
- Many more resources are provided.
- Use **kubectl api-resources** for an overview.
```bash
kubectl api-resources | less
```
- Draw a diagram of api-resources and explain about it.

### Lesson 3 Lab Exploring Kubernetes API Resources

- Use **kubectl create deploy mydeploy --image=nginx** to run an nginx application.
- Use **kubectl get all** to see which resources have been created.
- Use **kubectl create api-resources** to get a list of all of the API resources that are available.

### Lesson 3 Lab Solution Exploring Kubernetes API Resources

```bash
kubectl create deploy mydeploy --image=nginx
kubectl get all
kubectl delete <POD NAME>
kubectl get all # See new pod is creating.
kubectl create api-resources
```
