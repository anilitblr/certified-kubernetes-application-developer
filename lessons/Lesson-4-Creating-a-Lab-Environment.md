# Lesson 4 - Creating a Lab Environment

- [Lesson 4 - Creating a Lab Environment](#lesson-4---creating-a-lab-environment)
    - [4.1 Understanding Kubernetes Deployment Options](#41-understanding-kubernetes-deployment-options)
      - [Understanding Deployment Options](#understanding-deployment-options)
    - [4.2 Using Minikube](#42-using-minikube)
      - [What is Minikube?](#what-is-minikube)
      - [Configuration Requirements](#configuration-requirements)
    - [4.3 Installing Minikube on Ubuntu](#43-installing-minikube-on-ubuntu)
      - [Installing a Lab Environment](#installing-a-lab-environment)
    - [4.4 Installing Minikube on Windows](#44-installing-minikube-on-windows)
    - [4.5 Installing Minikube on macOS](#45-installing-minikube-on-macos)
    - [4.6 Verifying Minikube is Working](#46-verifying-minikube-is-working)
      - [Testing Minikube](#testing-minikube)
    - [4.7 Running your First Application](#47-running-your-first-application)
    - [Lesson 4 Lab Setting up a Lab Environment](#lesson-4-lab-setting-up-a-lab-environment)
    - [Lesson 4 Lab Solution Setting up a Lab Environment](#lesson-4-lab-solution-setting-up-a-lab-environment)

### 4.1 Understanding Kubernetes Deployment Options

#### Understanding Deployment Options

- Kubernetes can be deployed in many environments.
  - In a public cloud.
  - On-permise in a datacenter.
  - Using Minikube or any other compact Kubernetes distribution for testing and developing.
- As Minikube is a rich and easy to use platform, studetns are recommended to install Minikube for this course.
- In other platforms some components may be  not available.

### 4.2 Using Minikube

#### What is Minikube?

- Minikube is an all-in-one Kubernetes solution.
- Typically, container and worker functionality are offered in one virtual machine.
- To run minikube in a virtual machine, you need a virtualization layer to be present and configured.
- Configuration of the virtualization layer is not discussed in the course as it is operating system dependent and doesn't have to do with CKAD.
- As an easy alternative, Minikube can also be started in a container.
- This course recommended installation method will show you how to install Minikube as a container using a dedicated Ubuntu virtual machine.

#### Configuration Requirements

- Minikube offers a complete test environment that runs on Linux, OS-X or Windows.
- In this course, we will focus on Minikube as it is easy to set up and has no further dependencies.
- Also, Minikube comes as a ready-to-use Kubernetes distribution with many add-ons installed.
- For best possible flexibility, it's a good idea to install Minikube in a virtual machine.
- Alternatively, you may run it on top of any operating system.
- To use Minikube, you should have at least 2 GiB (4 GiB recommended) RAM and 2 vCPUs.

### 4.3 Installing Minikube on Ubuntu

#### Installing a Lab Environment

- Easy installation on top of recent Ubuntu is provided through a lab setup script.
- You will need to provide an Ubuntu virtual (or physical) machine to use the lab set up script.
```bash
cd labs/
sh minikube-docker-setup.sh
minikube start --vm-driver=docker --cni=calico
minikube status;
```

### 4.4 Installing Minikube on Windows

- Install Minikube on Windows
  - ref: https://minikube.sigs.k8s.io/docs/start/
- Install kubectl on Windows 
  - ref: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
```bash
kubectl create deploy winner --image=nginx
kubectl get all
```

### 4.5 Installing Minikube on macOS

ref: https://minikube.sigs.k8s.io/docs/start/

### 4.6 Verifying Minikube is Working

#### Testing Minikube

- **minikube status**: shows current status.
- **kubectl get all**: vrifies kubectl client operation.
- **minikube start**: starts Minikube
- **minikube stop**: stops Minikube
- **minikube ssh**: logs in to the Minikube host.
- **docker ps**: shows all Docker processes on the Minikube host.

The **minikube** command has different options.
- **dashboard**: opens the K8s dashboard in the local browser.
- **delete**: deletes a cluster.
- **ip**: shows the currently used IP address.
- **start**: starts a cluster.
- **stop**: stops a cluster.
- **status**: gives current status.
- **version**: shows current version.
- **ssh**: connects to the Minikube cluster. 

### 4.7 Running your First Application

```bash
kubectl create deploy myapp --image=nginx --replicas=3
kubectl get all;
kubectl delete pod <POD NAME>
kubectl delete myapp;
kubectl get all;
kubectl svc myapp;

# Kubectl shell completion
kubectl completion -h

kubectl completion bash > ~/.kube/completion.bash.inc

printf "
# Kubectl shell completion
source '$HOME/.kube/completion.bash.inc'
" >> $HOME/.bash_profile

source $HOME/.bash_profile

kubectl run -h | less
kubectl run nginx --image=nginx
kubectl get all
kubectl delete pod nginx
kubectl get all
```

### Lesson 4 Lab Setting up a Lab Environment

- Set up a Minikube-based lab environment.
- Use the Kubernetes Dashboard to install a simple app that runs the Nginx web server, and uses the name nging-app

### Lesson 4 Lab Solution Setting up a Lab Environment

- Follow the above doc to set up the lab.

- Use the Kubernetes Dashboard to install a simple app that runs the Nginx web server, and uses the name nginx-app
```bash
# Start Kubernetes dashboard.
minikube dashboard;
```

- Click on **Plus**
- Chose **Create from form**
- App name: **nginx-app**
- Container image: **nginx**
- Number of pods: **3**
- Service: Choose **Internal**
- Port: **80** Target port: **80**
- Namespace: **default**
- Choose **Deploy**
- Go back to the terminial, Press **Ctrl+z** and the **bg**
```bash
kubectl get all;
kubectl delete deploy nginx-app
```
