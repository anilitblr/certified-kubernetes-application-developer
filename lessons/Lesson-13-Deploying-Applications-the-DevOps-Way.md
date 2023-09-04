# Lesson 13 - Deploying Applications the DevOps Way

- [Lesson 13 - Deploying Applications the DevOps Way](#lesson-13---deploying-applications-the-devops-way)
    - [13.1 Using the Helm Package Manager](#131-using-the-helm-package-manager)
      - [Understanding Helm](#understanding-helm)
      - [Demo: Installing the Helm Binary](#demo-installing-the-helm-binary)
      - [Getting Access to Helm Charts](#getting-access-to-helm-charts)
      - [Demo: Managing Helm Repositories](#demo-managing-helm-repositories)
    - [13.2 Working with Helm Charts](#132-working-with-helm-charts)
      - [Installing Helm Charts](#installing-helm-charts)
      - [Demo: Installing Helm Charts](#demo-installing-helm-charts)
      - [Customizing Before Installing](#customizing-before-installing)
      - [Demo: Customizing Before Install](#demo-customizing-before-install)
    - [13.3 Using Kustomize](#133-using-kustomize)
      - [Understanding Kustomize](#understanding-kustomize)
      - [Understanding a Sample kustomization.yaml](#understanding-a-sample-kustomizationyaml)
      - [Using Kustomization Overlays - 1](#using-kustomization-overlays---1)
      - [Using Kustomization Overlays - 2](#using-kustomization-overlays---2)
      - [Demo: Using Kustomization](#demo-using-kustomization)
    - [13.4 Implementing Blue/Green Deployment](#134-implementing-bluegreen-deployment)
      - [Understanding Blue/Green](#understanding-bluegreen)
      - [Procedure Overview](#procedure-overview)
      - [Demo: Using Blue/Green Deployments](#demo-using-bluegreen-deployments)
    - [13.5 Implementing Canary Deployments](#135-implementing-canary-deployments)
      - [Understanding Canary Deployments](#understanding-canary-deployments)
      - [Demo Step 1: Running the Old Version](#demo-step-1-running-the-old-version)
      - [Demo Step 2: Creating a CongigMap](#demo-step-2-creating-a-congigmap)
      - [Demo Step 3: Preparing the New Version](#demo-step-3-preparing-the-new-version)
      - [Demo Step 4: Activating the New Version](#demo-step-4-activating-the-new-version)
    - [13.6 Understanding Custom Resource Definitions](#136-understanding-custom-resource-definitions)
      - [Understanding Custom Resource Definitions](#understanding-custom-resource-definitions)
      - [Creating Custom Resources](#creating-custom-resources)
      - [Demo: Creating Custom Resources](#demo-creating-custom-resources)
    - [13.7 Using Operators](#137-using-operators)
      - [Understanding Operators and Controllers](#understanding-operators-and-controllers)
      - [Demo: Installing the Calico Network Plugin](#demo-installing-the-calico-network-plugin)
    - [13.8 Using StatefulSets](#138-using-statefulsets)
      - [Understanding StatefulSet](#understanding-statefulset)
      - [Understanding StatefulSet Limitations](#understanding-statefulset-limitations)
      - [Demo: Using a StatefulSet](#demo-using-a-statefulset)
    - [Lesson 13 Lab Deploying Kubernetes Applications the DevOps Way](#lesson-13-lab-deploying-kubernetes-applications-the-devops-way)
    - [Lesson 13 Lab Solution Deploying Kubernetes Applications the DevOps Way](#lesson-13-lab-solution-deploying-kubernetes-applications-the-devops-way)

### 13.1 Using the Helm Package Manager

#### Understanding Helm

- Helm is used to streamline installing and managing Kubernetes applications.
- Helm consists of the **helm** tool, which needs to be installed, and a chart.
- A chart is a Helm package, which contains the following.
  - A description of the package.
  - One or more templates containing Kubernetes manifest files.
- Charts can be stored locally, or accessed from remote Helm repositories.

#### Demo: Installing the Helm Binary

```bash
# Download the binary file from and check for the latest version.
https://github.com/helm/helm/releases click **Linux amd64** to download
# or
cd ~/Downloads/
wget https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz

# Extract.
tar -xvf helm-v3.12.3-linux-amd64.tar.gz

# Move helm binary file to the local path.
sudo mv linux-amd64/helm /usr/local/bin/

# Check helm version.
helm version
```

#### Getting Access to Helm Charts

- The main site for finding Helm charts, is through **https://artifacthub.io**
- This is a major way for finding repository names.
- Search for specific software here, and run the commands to install it, for instance, to run the Kubernetes Dashboard.
  - **helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/**
  - **helm install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard**
- Note: in Minikube, use **minikube addons enable dashboard** for easire installation

#### Demo: Managing Helm Repositories

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm search repo bitnami
helm repo update
```

### 13.2 Working with Helm Charts

#### Installing Helm Charts

- After adding repositories, use **helm repo update** to ensure access to the most up-to-date information.
- Use **helm install** to install the chart with default parameters.
- After installation, use **helm list** to list currently installed charts.
- Optionally, use **helm delete** to remove currently installed charts.

#### Demo: Installing Helm Charts

```bash
helm install bitnami/mysql --generate-name
kubectl get all
helm show chart bitnami/mysql
helm show all bitnami/mysql
helm list
helm status mysql-xxxx
```

#### Customizing Before Installing

- A Helm chart consists of templates to which specific values are applied.
- The values are stored in the values.yaml file, within the helm chart.
- The easiest way to modify these values, is by first using **helm pull** to fetch a local copy of the helm chart.
- Next use your favorite editor on **chartname/values.yaml** to change any values.

#### Demo: Customizing Before Install

```bash
helm show values bitnami/nginx
mkdir ~/helm-charts && cd $_
helm pull bitnami/nginx
tar xvf nginx-xxxx
vim nginx/values.yaml # Change the replicaCount = 1 to replicaCount = 3
helm template --debug nginx
helm install -f nginx/values.yaml my-nginx nginx/
kubectl get all
```

### 13.3 Using Kustomize

#### Understanding Kustomize

- **kustomize** is a Kubernetes feature, that uses a file with the name **kustomization.yaml** to apply changes to a set of resources.
- This is convenient for applying changes to input files that the user does not corntrol himself, and which contents may change because of new versions appearing in Git.
- Use **kubectl apply -k ./** in the directory with the **kustomization.yaml** and the first is refers to apply changes.
- Use **kubectl delete -k ./** in the same directory to delete all that was created by the kustomization.

#### Understanding a Sample kustomization.yaml

```yaml
resources:         # defines which resources (in YAML files) apply
- deployment.yaml
- service.yaml
namePrefix: test-  # specifies a perfix should be added to all names.
namespace: testing # objects will be created in this specific namespace.
commonLabels:      # labels that will be added to all objects.
  environment: testing
```

#### Using Kustomization Overlays - 1

- Kustomization can be used to define a base configuration, as well as multiple deployments scenarios (overlays) as in dev, staging and prod for instance.
- In such a configuration, the main **kustomization.yaml** defines the structure.
```yaml
- base
  - deployment.yaml
  - service.yaml
  - kustomization.yaml
- overlays
  - dev
    - kustomization.yaml
  - staging
    - kustomization.yaml
  - prod
    - kustomization.yaml
```

#### Using Kustomization Overlays - 2

- In each of the overlays/{dev,staging,prod}/kustomization.yaml, users would reference the base configuration in the resources field, and specify changes for that specific environment.
```yaml
resources:
- ../../base
namePrefix: dev-
namespace: development
commonLabels:
  environment: development
```

#### Demo: Using Kustomization

```bash
cd labs/kustomization/
cat deployment.yaml
cat service.yaml
# kubectl apply -f deployment.yaml service.yaml
cat kustomization.yaml
kubectl apply -k ./
kubectl get all --selector environment=testing
kubectl delete -k ./
```

### 13.4 Implementing Blue/Green Deployment

#### Understanding Blue/Green

- Blue/Green Deployments are a strategy to accomplish zero downtime application upgrade.
- Essential is the possibility to test the new version of the application before taking it into production.
- The blue Deployment is the current application.
- The green Deployment is the new application.
- Once the green Deployment is tested and ready, traffic is re-routed to the new application version.
- Blue/Green Deployments can easily be implemented using Kubernetes Service.

#### Procedure Overview

- Start with the already running application.
- Create a new Deployment that is running the new version, and test with a temporary Service resource.
- If all tests pass, remove the temporary Service resource.
- Remove the old Service resource (pointing to the blue Deployment), and immediately create a new Service resource exposing the green Deployment.
- After successful transition, remove the blue Deployment.
- It is essential to keep the Service name unchanged, so that front-end resources such as Ingress will automatically pick up the transition.

#### Demo: Using Blue/Green Deployments

```bash
kubectl create deploy blue-nginx --image=nginx:1.14 --replicas=3
kubectl expose deploy blue-nginx --port=80 --name=bgnginx
kubectl get deploy blue-nginx -o yaml > green-nginx.yaml
vim green-nginx.yaml
# 1. Clean up dynamic generated stuff.
# 2. Change Image version.
# 3. Change "blue" to "green" throughout
kubectl create -f green-nginx.yaml
kubectl get all
kubectl expose deploy green-nginx --port=80 --name=green
kubectl get endpoints
kubectl delete svc green;
kubectl delete svc bgnginx; kubectl expose deploy green-nginx --port=80 --name=bgnginx
kubectl get pods -o wide
kubectl get endpoints
kubectl delete deploy blue-nginx
kubectl get all
```

### 13.5 Implementing Canary Deployments

#### Understanding Canary Deployments

- A canary Deployment is an update strategy where you first push the update at samll scale to see if it works well.
- In terms of Kubernetes, you could imagine a Deployment that runs 4 replicas.
- Next, you add a new Deployment that uses the same label.
- Then you create a Service that uses the same selector label for all.
- As the Service is load balancing, only 1 out of 5 requests would be serviced by the newer version.

#### Demo Step 1: Running the Old Version

```bash
kubectl create deploy old-nginx --image=nginx:1.14 --replicas=3 --dry-run=client -o yaml > ~/oldnginx.yaml
vim ~/oldnginx.yaml
# 1. set labels: type: canary in deploy metadata as well as Pod metadata under app.
kubectl create -f oldnginx.yaml
kubectl get all
kubectl expose deploy old-nginx --name=oldnginx --port=80 --selector type=canary
kubectl get svc
kubectl get endpoints
kubectl get pods -o wide
minikube ssh
curl svc-ip-address # a few times, you will see all the same.
exit
```

#### Demo Step 2: Creating a CongigMap

```bash
kubectl cp <old-nginx-pod>:/usr/share/nginx/html/index.html index.html
vim index.html # 1. Add a line that uniquely identifies this as the canary Pod.
kubectl create configmap canary --from-file=index.html
kubectl describe cm canary
```

#### Demo Step 3: Preparing the New Version

```bash
cp oldnginx.yaml canary.yaml
vim canary.yaml
# 1. image: nginx:latest
# 2. replicas: 1
# 3. :%s/old/new/g
# 4. Mount the configMap as a volume (see Git repo canary.yaml)
kubectl create -f canary.yaml
kubectl get svc
kubectl get endpoints
kubectl get pods --show-labels
minikube ssh
curl <service-ip> # and notice different rusults, this is canary in action.
exit
```

#### Demo Step 4: Activating the New Version

```bash
# Use to verify the names of the old and the new deployment
kubectl get deploy

# Use to scale the canary deployment up to the desired.
kubectl scale deploy new-nginx --replicas=3
kubectl get deploy
kubectl get pods
kubectl get svc

# Delete the old deployment or set the scale to 0.
kubectl scale deploy old-nginx --replicas=0
kubectl get all
curl <service-ip> # and notice different rusults, this is canary in action.
exit

kubectl delete deploy oldnginx
```

### 13.6 Understanding Custom Resource Definitions

#### Understanding Custom Resource Definitions

- Custom Resource Definitions (CRDs) allow users to add custom resources to clusters.
- Doing so allows anything to be integrated in a cloud-native environment.
- The CRD allows users to add resources in a very easy way.
  - The resources are added as extension to the original Kubernetes API server.
  - No programming skills required.
- The alternative ay to build custom resources, is via API integration.
  - This will build a custom API server.
  - Programming skills are required.

#### Creating Custom Resources

- Creating Custom Resources using CRDs is a two-step procedure.
- First, you will need to define the resource, using the CustomResourceDefinition API kind.
- After defining the resource, it can be added though its own API resource.

#### Demo: Creating Custom Resources

```bash
cd labs/
cat crd-object.yaml
kubectl create -f crd-object.yaml
kubectl api-resources | grep backup
cat crd-backup.yaml
kubectl create -f crd-backup.yaml
kubectl get backups
kubectl describe backups.stable.example.com mybackup
```

### 13.7 Using Operators

#### Understanding Operators and Controllers

- Operators are custom applications, based on Custom Resource Definitions.
- Operators can be seen as a way of packaging, running and managing applications in Kubernetes.
- Operators are based on Controllers, which are Kubernetes components that continuously operate dynamic systems.
- The Controller loop is the essence of any Controllers.
- The Kubernetes Controller manager runs a reconciliation loop, which continuously observes the current state, compares it to the desired state, and adjusts it when necessary.
- Operators are application-specific Controllers.
- Operators can be added to Kubernetes by developing them yourself.
- Operators are also avilable from community websites.
- A common registry for operators is found at **operatorhub.io** (which is rather OpenShift oriented)
- Many solutions from the Kubernetes ecosystem are provided as operators.
  - Prometheus: a monitoring and alerting solution.
  - Tigera: the operator that manages the calico network plugin.
  - Jaeger: used for tracking transactions between distributed services.

#### Demo: Installing the Calico Network Plugin

```bash
minikube stop
minikube delete
minikube start --network-plugin=cni --extra-config=kubeadm.pod-network-cidr=10.10.0.0/16
kubectl get all
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

kubectl get ns
kubectl get all -n tigera-operator
kubectl api-resources | grep tigera
kubectl get pods -n tigera-operator
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O
sed -i -e s/192.168.0.0/10.10.0.0/g custom-resources.yaml
kubectl create -f custom-resources.yaml
kubectl get installation -o yaml
kubectl get pods -n calico-system
```

- ref: https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

### 13.8 Using StatefulSets

#### Understanding StatefulSet

- The main purpose of a StatefulSet is to provide a persistent identity to Pods as well as the Pod-speific storage.
- Each Pod in a StatefulSet has a persistent identifier that it keeps across rescheduling.
- StatefulSet provides ordering as well.
- Using StatefulSet is valuable for applications that require any of the following.
  - Stable and unique network identifiers.
  - Stable persistent storage.
  - Ordered deployment and scaling.
  - Ordered automated rolling updates.

#### Understanding StatefulSet Limitations

- Storage Provisioning based on StorageClass must be available.
- To ensure data sefety, volumes created by the StatefulSet are not deleted while deleting the StatefulSet.
- A Headless Service must be created to provide application access.
- To guarantee removal of StatefulSet Pods, scale down the number of Pods to 0 (zero) before removing the StatefulSet.

#### Demo: Using a StatefulSet

```bash
cat sfs.yaml
kubectl get storageclass
kubectl create -f sfs.yaml
kubectl get all
kubectl get pvc
```

### Lesson 13 Lab Deploying Kubernetes Applications the DevOps Way

- Run an nginx Deployment that meets the following requirements.
  - Use a ConfigMap to provide an index.html file containing the text "Welcome to the old version"
  - Use image version 1.14
  - Run 3 replicas
- Use the canary Deployment upgrade strategy to replace with a newer version of the application.
  - Use a ConfigMap to provide an index.html in the new applicatio, containing the text "Welcome to the new version"
  - Set the image version to latest
- Complete the transition such that the old application is completely removed after verifying successful working of the updated application.

### Lesson 13 Lab Solution Deploying Kubernetes Applications the DevOps Way

```bash
# Create a index.html file for old version.
echo Welcome to the old version > index.html

# Create configMap for old version
kubectl create cm --help | less
kubectl create cm oldversion --from-file=index.html
kubectl get cm oldversion -o yaml

# Create a index.html file for new version.
echo Welcome to the new version > index.html

# Create configMap for new version
kubectl create cm --help | less
kubectl create cm newversion --from-file=index.html
kubectl get cm newversion -o yaml
kubectl get cm -o yaml

# Create old deployment
kubectl create deploy oldnginx --image=nginx:1.14 --replicas=3 --dry-run=client -o yaml > oldnginx.yaml
vim oldnginx.yaml
# 1. set labels: type: canary in deploy metadata as well as Pod metadata under app.
kubectl create -f oldnginx.yaml
kubectl get all
kubectl get all --show-labels
kubectl expose -h | less
kubectl expose deploy oldnginx --name=canary --port=80 --selector=type=canary
kubectl get svc
kubectl describe svc canary
minikube ssh
curl <service-ip>
exit

# Create new deployment
kubectl create -f newnginx.yaml
kubectl get svc
minikube ssh
curl <service-ip>
exit

# After testing new version application.
kubectl get deploy
kubectl scale deploy newnginx --replicas=3
kubectl get deploy
kubectl get pods --selector type=canary
kubectl scale deploy oldnginx --replicas=0
kubectl get pods --selector type=canary
minikube ssh
curl <service-ip>
exit
```
