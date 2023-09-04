# Certified Kubernetes Application Developer (CKAD)

- [Certified Kubernetes Application Developer (CKAD)](#certified-kubernetes-application-developer-ckad)
- [Module 1 Container Fundamentals](#module-1-container-fundamentals)
    - [Lesson 1 - Understanding and Using Containers](#lesson-1---understanding-and-using-containers)
    - [Lesson 2 - Managing Container Images](#lesson-2---managing-container-images)
    - [Lesson 3 - Understanding Kubernetes](#lesson-3---understanding-kubernetes)
    - [Lesson 4 - Creating a Lab Environment](#lesson-4---creating-a-lab-environment)
- [Module 2 Kubernetes Essentials](#module-2-kubernetes-essentials)
    - [Lesson 5 - Managing Pod Basic Features](#lesson-5---managing-pod-basic-features)
    - [Lesson 6 - Managing Pod Advanced Features](#lesson-6---managing-pod-advanced-features)
- [Module 3 Building and Exposing Scalable Applications](#module-3-building-and-exposing-scalable-applications)
    - [Lesson 7 - Managing Deployments](#lesson-7---managing-deployments)
    - [Lesson 8 - Managing Networking](#lesson-8---managing-networking)
    - [Lesson 9 - Managing Ingress](#lesson-9---managing-ingress)
    - [Lesson 10 - Managing Kubernetes Storage](#lesson-10---managing-kubernetes-storage)
    - [Lesson 11 - Managing ConfigMaps and Secrets](#lesson-11---managing-configmaps-and-secrets)
- [Module 4 Advanced CKAD Tasks](#module-4-advanced-ckad-tasks)
    - [Lesson 12 - Using the API](#lesson-12---using-the-api)
    - [Lesson 13 - Deploying Applications the DevOps Way](#lesson-13---deploying-applications-the-devops-way)
    - [Lesson 14 - Troubleshooting Kubernetes](#lesson-14---troubleshooting-kubernetes)
- [Module 5 Sample Exam](#module-5-sample-exam)
    - [Lesson 15 - Sample CKAD Exam](#lesson-15---sample-ckad-exam)

# Module 1 Container Fundamentals

### [Lesson 1 - Understanding and Using Containers](lessons/Lesson-1-Understanding-and-Using-Containers.md)

- 1.1 What is a Container
- 1.2 Understanding Registries
- 1.3 Starting Containers
- 1.4 Managing Containers
- 1.5 Managing Container Images
- 1.6 Understanding Container Logging
- Lesson 1 Lab Using Containers
- Lesson 1 Lab Solution Using Containers

### [Lesson 2 - Managing Container Images](lessons/Lesson-2-Managing-Container-Images.md)

- 2.1 Understanding Image Architecture
- 2.2 Tagging Container Images
- 2.3 Understanding Image Creation Options
- 2.4 Building Custom Container Images
- 2.5 Using Dockerfile/Containerfile
- 2.6 Creating Images with Docker Commit
- Lesson 2 Lab Creating Custom Container Images
- Lesson 2 Lab Solution Creating Custom Container Images

### [Lesson 3 - Understanding Kubernetes](lessons/Lesson-3-Understanding-Kubernetes.md)

- 3.1 Understanding Kubernetes Core Functions
- 3.2 Understanding Kubernetes Origins
- 3.3 Using Kubernetes in Google Cloud
- 3.4 Understanding Kubernetes Management Interfaces
- 3.5 Understanding Kubernetes Architecture
- 3.6 Exploring Essential API Resources
- Lesson 3 Lab Exploring Kubernetes API Resources
- Lesson 3 Lab Solution Exploring Kubernetes API Resources

### [Lesson 4 - Creating a Lab Environment](lessons/Lesson-4-Creating-a-Lab-Environment.md)

- 4.1 Understanding Kubernetes Deployment Options
- 4.2 Using Minikube
- 4.3 Installing Minikube on Ubuntu
- 4.4 Installing Minikube on Windows
- 4.5 Installing Minikube on macOS
- 4.6 Verifying Minikube is Working
- 4.7 Running your First Application
- Lesson 4 Lab Setting up a Lab Environment
- Lesson 4 Lab Solution Setting up a Lab Environment

# Module 2 Kubernetes Essentials

### [Lesson 5 - Managing Pod Basic Features](lessons/Lesson-5-Managing-Pod-Basic-Features.md)

- 5.1 Understanding Pods
- 5.2 Understanding YAML
- 5.3 Generating YAML Files
- 5.4 Understanding and Configuring Multi-container Pods
- 5.5 Working with Init Containers
- 5.6 Using Namespaces
- Lesson 5 Lab Managing Pods
- Lesson 5 Lab Solution Managing Pods

### [Lesson 6 - Managing Pod Advanced Features](lessons/Lesson-6-Managing-Pod-Advanced-Features.md)

- 6.1 Exploring Pod State with kubectl describe
- 6.2 Exploring Pod Logs for Application Troubleshooting
- 6.3 Using Port Forwarding to Access Pods
- 6.4 Understanding and Configuring SecurityContext
- 6.5 Managing Jobs
- 6.6 Managing CronJobs
- 6.7 Managing Resource Limitations and Quota
- 6.8 Cleaning up Resources
- Lesson 6 Lab Managing Pod Advanced Features
- Lesson 6 Lab Solution Managing Pod Advanced Features

# Module 3 Building and Exposing Scalable Applications

### [Lesson 7 - Managing Deployments](lessons/Lesson-7-Managing-Deployments.md)

- 7.1 Understanding Deployments
- 7.2 Managing Deployment Scalability
- 7.3 Understanding Deployment Updates
- 7.4 Understanding Labels, Selectors and Annotations
- 7.5 Managing Update Strategy
- 7.6 Managing Deployment History
- 7.7 Understanding DaemonSet
- 7.8 Bonus topic Understanding AutoScaling
- Lesson 7 Lab Managing Deployments
- Lesson 7 Lab Solution Managing Deployments

### [Lesson 8 - Managing Networking](lessons/Lesson-8-Managing-Networking.md)

- 8.1 Understanding Kubernetes Networking
- 8.2 Understanding Services
- 8.3 Creating Services
- 8.4 Using Service Resources in Microservices
- 8.5 Understanding Services and DNS
- 8.6 Understanding and Configuring NetworkPolicy
- Lesson 8 Lab Managing Services
- Lesson 8 Lab Solution Managing Services

### [Lesson 9 - Managing Ingress](lessons/Lesson-9-Managing-Ingress.md)

- 9.1 Understanding Ingress
- 9.2 Configuring the Minikube Ingress Controller
- 9.3 Using Ingress
- 9.4 Configuring Ingress Rules
- 9.5 Understanding IngressClass
- 9.6 Troubleshooting Ingress
- Lesson 9 Lab Using Ingress
- Lesson 9 Lab Solution Using Ingress

### [Lesson 10 - Managing Kubernetes Storage](lessons/Lesson-10-Managing-Kubernetes-Storage.md)

- 10.1 Understanding Kubernetes Storage Options
- 10.2 Configuring Pod Volume Storage
- 10.3 Configuring PV Storage
- 10.4 Configuring PVCs
- 10.5 Configuring Pod Storage with PV and PVC
- 10.6 Understanding StorageClass
- Lesson 10 Lab Setting up Storage
- Lesson 10 Lab Solution Setting up Storage

### [Lesson 11 - Managing ConfigMaps and Secrets](lessons/Lesson-11-Managing-ConfigMaps-and-Secrets.md)

- 11.1 Providing Variables to Kubernetes Applications
- 11.2 Understanding Why Decoupling is Important
- 11.3 Providing Variables with ConfigMaps
- 11.4 Providing Configuration Files Using ConfigMaps
- 11.5 Understanding Secrets
- 11.6 Understanding How Kubernetes Uses Secrets
- 11.7 Configuring Applications to Use Secrets
- 11.8 Configuring the Docker Registry Access Secret
- Lesson 11 Lab Using ConfigMaps and Secrets
- Lesson 11 Lab SolutionUsing ConfigMaps and Secrets

# Module 4 Advanced CKAD Tasks

### [Lesson 12 - Using the API](lessons/Lesson-12-Using-the-API.md)

- 12.1 Understanding the Kubernetes API
- 12.2 Using curl to Work with API Objects
- 12.3 Understanding API Deprecations
- 12.4 Understanding Authentication and Authorization
- 12.5 Understanding API Access and ServiceAccounts
- 12.6 Understanding Role Based Access Control
- 12.7 Configuring a ServiceAccount
- Lesson 12 Lab Configuring a Service Account
- Lesson 12 Lab Solution Configuring a Service Account

### [Lesson 13 - Deploying Applications the DevOps Way](lessons/Lesson-13-Deploying-Applications-the-DevOps-Way.md)

- 13.1 Using the Helm Package Manager
- 13.2 Working with Helm Charts
- 13.3 Using Kustomize
- 13.4 Implementing Blue/Green Deployment
- 13.5 Implementing Canary Deployments
- 13.6 Understanding Custom Resource Definitions
- 13.7 Using Operators
- 13.8 Using StatefulSets
- Lesson 13 Lab Deploying Kubernetes Applications the DevOps Way
- Lesson 13 Lab Solution Deploying Kubernetes Applications the DevOps Way

### [Lesson 14 - Troubleshooting Kubernetes](lessons/Lesson-14-Troubleshooting-Kubernetes.md)

- 14.1 Determining a Troubleshooting Strategy
- 14.2 Analyzing Failing Applications
- 14.3 Analyzing Pod Access Problems
- 14.4 Monitoring Cluster Event Logs
- 14.5 Troubleshooting Authentication Problems
- 14.6 Using Probes
- Lesson 14 Lab Troubleshooting Kubernetes
- Lesson 1 Lab Solution Troubleshooting Kubernetes

# Module 5 Sample Exam

### [Lesson 15 - Sample CKAD Exam](lessons/Lesson-15-Sample-CKAD-Exam.md)

- 15.1 Exam Tips
- 15.2 Exam Question Overview
- 15.3 Working with Namespaces
- 15.4 Using Secrets
- 15.5 Creating Custom Images
- 15.6 Using Sidecars
- 15.7 Fixing a Deployment
- 15.8 Using Probes
- 15.9 Creating a Deployment
- 15.10 Exposing Applications
- 15.11 Using NetworkPolicies
- 15.12 Using Storage
- 15.13 Using Quota
- 15.14 Creating Canary Deployments
- 15.15 Managing Pod Permissions
- 15.16 Using ServiceAccount
