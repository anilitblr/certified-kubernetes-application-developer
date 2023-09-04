# Lesson 7 - Managing Deployments

- [Lesson 7 - Managing Deployments](#lesson-7---managing-deployments)
    - [7.1 Understanding Deployments](#71-understanding-deployments)
      - [Understanding Deployments](#understanding-deployments)
      - [Demo: Creating Deployments](#demo-creating-deployments)
    - [7.2 Managing Deployment Scalability](#72-managing-deployment-scalability)
      - [Understanding Deployment Scalability](#understanding-deployment-scalability)
      - [Manually Managing Scalability](#manually-managing-scalability)
      - [Demo: Managing Scalability](#demo-managing-scalability)
    - [7.3 Understanding Deployment Updates](#73-understanding-deployment-updates)
      - [Understanding Deployment Updates](#understanding-deployment-updates)
      - [Demo: Applying Applications Updates](#demo-applying-applications-updates)
    - [7.4 Understanding Labels, Selectors and Annotations](#74-understanding-labels-selectors-and-annotations)
      - [Understanding Labels](#understanding-labels)
      - [Understanding Auto-created Labels](#understanding-auto-created-labels)
      - [Understanding Annotations](#understanding-annotations)
      - [Demo labels](#demo-labels)
    - [7.5 Managing Update Strategy](#75-managing-update-strategy)
      - [Understanding Update Strategy](#understanding-update-strategy)
      - [Understanding Rolling Updates](#understanding-rolling-updates)
      - [Using RollingUpdate Options](#using-rollingupdate-options)
      - [Demo: Investigating Current Settings](#demo-investigating-current-settings)
    - [7.6 Managing Deployment History](#76-managing-deployment-history)
      - [Understanding Deployment History](#understanding-deployment-history)
      - [Demo: Managing Rollout History](#demo-managing-rollout-history)
    - [7.7 Understanding DaemonSet](#77-understanding-daemonset)
      - [Understanding DaemonSet](#understanding-daemonset)
      - [Demo: Creating a DaemonSet](#demo-creating-a-daemonset)
    - [7.8 Bonus topic Understanding AutoScaling](#78-bonus-topic-understanding-autoscaling)
      - [Understanding Autoscaling](#understanding-autoscaling)
      - [Demo: Setting up Autoscaling](#demo-setting-up-autoscaling)
    - [Lesson 7 Lab Managing Deployments](#lesson-7-lab-managing-deployments)
    - [Lesson 7 Lab Solution Managing Deployments](#lesson-7-lab-solution-managing-deployments)

### 7.1 Understanding Deployments

#### Understanding Deployments

- The Deployments is the standard for running applications in Kubernetes.
- If offers features that add to the scalability and reliability of the application.
  - Scalability: scaling the number of application instances.
  - Updates and update Strategy: zero-downtime application updates.
- It protects Pods and will automatically restart them if anything goes wrong.
- Use **kubectl create deploy** to create Deployments.

#### Demo: Creating Deployments

```bash
kubectl create deployment myweb --image=nginx --replicas=3
kubectl describe deploy myweb
kubectl get all
kubectl delete pod myweb-xxx-yyy
kubectl get all
kubectl run pod mypod --image=nginx
kubectl delete pod mypod
kubectl get all
```

### 7.2 Managing Deployment Scalability

#### Understanding Deployment Scalability

- In very old versions of Kubernetes, the Deployment did not exist.
- To manage scalability, the ReplicaSet resource was used.
- With the introduction of the apps API extension, the Deployment was introduced.
- From that moment, scalability was manged in the Deployment and no longer in the ReplicaSet.
- To manage scalability however, the Deployment creates a ReplicaSet.
- Do NOT manage ReplicaSet directly, manage through the Deployment only.

#### Manually Managing Scalability

- Use kubectl scale deployment my-deployment --replicas=4 to scale the number of currently running replicas.
- Alternatively, use **kubectl edit deployment my-deployment** to edit the number of replicas manually.
  - Notice that **kubectl edit** allows modification of a limited number of options only.
- To start a deployment with a specific number of replicas, use **kubeclt create deploy ... --replicas=3**

#### Demo: Managing Scalability

```bash
kubectl api-resources | less
kubectl create -f redis-deploy.yaml
kubectl api-versions | less
# Edit redis-deploy.yaml and change apiVersion to apps/v1
kubectl create -f redis-deploy.yaml
kubectl get all
kubectl edit deploy redis # change number of replicas
kubectl get all
kubectl get all --selector app=redis
kubectl delete rs redis-xxx
kubectl get all --selector app=redis
```

### 7.3 Understanding Deployment Updates

#### Understanding Deployment Updates

- Using Deployments allows for zero-downtime application updates.
- Updates are used change different parts of the Deployment.
  - Image versions.
  - Any parameter that can be used with **kubectl set**
- When an update is applied, a new ReplicaSet is created with the new properties.
  - Pods with the new properties are started in the new ReplicaSet.
  - After successfull updates, the old ReplicaSet is no longer used and may be deleted.
  - The deployment.sepc.revisionHistoryLimit is set to keep the 10 last ReplicaSets.
- The updateStragety deployment property defines how to handle updates.

#### Demo: Applying Applications Updates

```bash
kubectl create deploy nginxup --image=nginx:1.14
kubectl get all --selector app=nginxup
kubectl get all --selector app=nginxup -o wide
kubectl describe deploy nginxup

kubectl set image deploy nginxup nginx=nginx:1.17
kubectl get all --selector app=nginxup -o wide
kubectl get all --selector app=nginxup

# Rollback to the old version, in casy if new version of image not working properly.
kubectl set image deploy nginxup nginx=nginx:1.14
kubectl get all --selector app=nginxup -o wide
kubectl get all --selector app=nginxup
```

### 7.4 Understanding Labels, Selectors and Annotations

#### Understanding Labels

- A lable is a key-value pair that provides additional information about Kubernetes resources.
- Labels are used a lot in Kubernetes.
- Labels are set in resources like Deployments and Services, and use selectors to connect to related resources.
- The Deployment finds its Pods using a selector.
- The Service finds tis endpoint Pods using a selector.
- Administrator can manually set labels to facilitate resource management and selection.
- To find all resources matching a specific label, use **--selector key=value**

#### Understanding Auto-created Labels

- Deployments created with **kubectl create** automatically get a label **app=appname**
- Pods started with **kubectl run** automatically get a label **run=podname**

#### Understanding Annotations

- Annotations were originally used to provide detailed metadata in an object.
- Annotations can not be used in queries.
- Think of information about licenses, maintainer, and more.
- Recently introduced resources may use annotations to provide additional functional information about the resources.

#### Demo labels

```bash
kubectl create deploy bluelabel --image=nginx
kubectl label deployment bluelabel state=demo
kubectl get deployments --show-labels
kubectl get deployments --selector state=demo
kubectl get all --show-labels
kubectl get all --show-labels --selector state=demo

kubectl describe deploy bluelabel # Look for Labels
kubectl label pod bluelabel-xxx app-
kubectl get all
kubectl get all --show-labels
```

### 7.5 Managing Update Strategy

#### Understanding Update Strategy

When a Deployment changes, the Pods are immediately updated according to the update strategy.
- Recreate: all Pods are killed, and new Pods are created. This will lead to temprary unavailable. Useful if you cannot simultaneously run different versions of an application.
- RollingUpdate: updates Pods one at a time to guarantee availability of the application. This is the preferred approach, and you can further tune its behaviour.

#### Understanding Rolling Updates

- It is the task of the Deployment to ensure that enough Pods are running at all times.
- So, when making a change, this change is applied as a rolling update: the changed version is deployed in a new ReplicaSet.
- After the update has been confirmed as successful, the old version ReplicaSet is scaled to 0 to deactivate.
- You can use **kubectl rollout history** to get details about recent transactions.
- Use **kubectl rollout undo** to undo a previous change.

#### Using RollingUpdate Options

The RollingUpdate options are used to guarantee a certain minimal and maximal number of Pods to be always available.
- maxUnavailable: determines the maximum number of Pods that are upgraded at the same time.
- maxSurge: the number of Pods that can run beyond the desired number of Pods as specified in a replica to guarantee minimal availability.

#### Demo: Investigating Current Settings

```bash
kubectl create deploy bluelabel --image=nginx
kubectl get deploy bluelabel -o yaml
kubectl edit deploy bluelabel
# set maxUnavailable: 2 and maxSurge: 4
kubectl scale deploy bluelabel --replicas=4
kubectl get all --selector app=bluelabel
kubectl set env deploy bluelabel type=blended
kubectl get all --selector app=bluelabel
```

### 7.6 Managing Deployment History

#### Understanding Deployment History

- In Deployment updates, the Deployment creates a new ReplicaSet that uses the new properties.
- The old ReplicaSet is kept, but the number of Pods will be bet to 0.
- This makes it easy to undo a change.
- **kubectl rollout history** will show the rollout history of a specific deployment, which can easily be reverted as well.
- Use **kubectl rollout history deployment mynginx --revision=1** to observe changes between versions.

#### Demo: Managing Rollout History

```bash
cd labs/
vim rolling.yaml
kubectl create -f rolling.yaml
kubectl get deploy
kubectl rollout history deployment
kubectl edit deployment rolling-nginx # change version to 1.15
kubectl rollout history deployment rolling-nginx
kubectl describe deployments rolling-nginx
kubectl rollout history deployment rolling-nginx --revision=2
kubectl rollout history deployment rolling-nginx --revision=1
kubectl rollout -h
kubectl rollout undo deployment rolling-nginx --to-revision=1
kubectl get all

kubectl scale deployment bluelabel --replicas=0
kubectl get all
```

### 7.7 Understanding DaemonSet

#### Understanding DaemonSet

- A DaemonSet is a Deployment that starts one Pod instance on every node in the cluster.
- This is useful in cases where a software component like an agen needs to be available on all cluster nodes.
- When nodes are added or removed, the DaemonSet automatically changes the number of Pods accordingly.
- Use YAMl code to create DaemonSet.

#### Demo: Creating a DaemonSet

```bash
cd labs/
vim daemon.yaml
kubectl apply -f daemon.yaml
kubectl get ds,pods
kubectl get ds,pods -n kube-system
```

### 7.8 Bonus topic Understanding AutoScaling

#### Understanding Autoscaling

- For CKAD, you need to know how to manually scale Pods using **kubectl scale** etc.
- In real clusters, Pods are often automatically scaled based on resource usage properties that are collected by the Metrics Server.
- The Horizontal Pod Autoscaler observes usage statistics and after passing a threshold, will add additional replicas.
- Setting up AutoScaling is NOT required of CKAD, but as it is so important in real clusters you will find a demo how to do it.
- This demo requires the use of Metrics Server, which is set up easily in Minikube.
- Setting up Metrics Server in other cluster types is covered in CKA.

#### Demo: Setting up Autoscaling

```bash
cd labs/autoscaling
docker build -t php-apache .
kubectl apply -f hpa.yaml # using the image from the Dockerfile as provided by the image registry.
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
kubectl api-resources | grep auto
kubectl get hpa

# From another terminal.
kubectl run -it load-nenerator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

# Back to the original terminal
sleep 60; kubectl get hpa
minikube addons enable metrics-server
minikube addons list
kubectl get hpa
kubectl get deploy php-apache
kubectl delete pod load-nenerator
kubectl get hpa
kubectl get deploy php-apache
```

### Lesson 7 Lab Managing Deployments

- Create YAML file that starts a deployment for Nginx. It should use image version 1.9 and start 5 replicas. The deployment should have the label type=proxy
- Also, configure the deployment such that when it is upgraded, no more then 2 Pods will be unavailable at the same time.

### Lesson 7 Lab Solution Managing Deployments

```bash
kubectl create deploy lab-7-nginx --image=nginx:1.9 --replicas=5 --dry-run=client -o yaml > lab7.yaml
kubectl explain --recursive deployment.spec | less
vim lab7.yaml
kubectl apply -f lab7.yaml
kubectl get all --selector app=lab-7-nginx
kubectl set image deploy lab-7-nginx nginx=nginx:latest
kubectl get all --selector app=lab-7-nginx
```

- vim lab7.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: lab-7-nginx
    type: proxy
  name: lab-7-nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: lab-7-nginx
  strategy: 
    rollingUpdate:
      maxUnavailable: 2
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: lab-7-nginx
        type: proxy
    spec:
      containers:
      - image: nginx:1.9
        name: nginx
        resources: {}
status: {}
```
