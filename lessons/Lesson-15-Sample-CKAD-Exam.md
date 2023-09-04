# Lesson 15 - Sample CKAD Exam

- [Lesson 15 - Sample CKAD Exam](#lesson-15---sample-ckad-exam)
    - [15.1 Exam Tips](#151-exam-tips)
      - [Examp Tips](#examp-tips)
    - [15.2 Exam Question Overview](#152-exam-question-overview)
      - [Question 1 - Working with Namespaces](#question-1---working-with-namespaces)
      - [Question 2 - Using Secrets](#question-2---using-secrets)
      - [Question 3 - Creating Custom Images](#question-3---creating-custom-images)
      - [Question 4 - Using Sidecars](#question-4---using-sidecars)
      - [Question 5 - Fixing a Deployment](#question-5---fixing-a-deployment)
      - [Question 6 - Using Probes](#question-6---using-probes)
      - [Question 7 - Creating a Deployment](#question-7---creating-a-deployment)
      - [Question 8 - Exposing Applications](#question-8---exposing-applications)
      - [Question 9 - Using NetworkPolicies](#question-9---using-networkpolicies)
      - [Question 10 - Using Storage](#question-10---using-storage)
      - [Question 11 - Using Quota](#question-11---using-quota)
      - [Question 12 - Creating Canary Deployments](#question-12---creating-canary-deployments)
      - [Question 13 - Managing Pod Permissions](#question-13---managing-pod-permissions)
      - [Question 14 - Using a ServiceAccount](#question-14---using-a-serviceaccount)
    - [15.3 Working with Namespaces](#153-working-with-namespaces)
    - [15.4 Using Secrets](#154-using-secrets)
    - [15.5 Creating Custom Images](#155-creating-custom-images)
    - [15.6 Using Sidecars](#156-using-sidecars)
    - [15.7 Fixing a Deployment](#157-fixing-a-deployment)
    - [15.8 Using Probes](#158-using-probes)
    - [15.9 Creating a Deployment](#159-creating-a-deployment)
    - [15.10 Exposing Applications](#1510-exposing-applications)
    - [15.11 Using NetworkPolicies](#1511-using-networkpolicies)
    - [15.12 Using Storage](#1512-using-storage)
    - [15.13 Using Quota](#1513-using-quota)
    - [15.14 Creating Canary Deployments](#1514-creating-canary-deployments)
    - [15.15 Managing Pod Permissions](#1515-managing-pod-permissions)
    - [15.16 Using ServiceAccount](#1516-using-serviceaccount)

### 15.1 Exam Tips

#### Examp Tips

- Be fast and use the most efficient tools.
- Don't write YAML from scratch but generate it.
- Train using kubernetes.io/docs
- Don't forget the **kubectl** cheat sheet https//kubernetes.io/docs/reference/kubectl/cheatsheet/
- Use **kubectl explain** a lot
- Make sure you know how to deal with Kubernetes Namespaces.

### 15.2 Exam Question Overview

#### Question 1 - Working with Namespaces
Create a Namespace ckad-ns1 in your cluster. In this Namespace, run the following Pods.
- A Pod with the name pod-a, running the httpd server image.
- A Pod with the name pod-b, running the nginx server image as well as the alpine image

#### Question 2 - Using Secrets

- Create a Secret that defines the variable password=secret. Create a Deployment with the name secretapp, which starts the nginx image and uses this variable.

#### Question 3 - Creating Custom Images

- Create a Dockerfile that runs an alpine image with the command "echo hello world" as the default command. Build the image, and export it in OCI format to a tar file with the name "greetworld"

#### Question 4 - Using Sidecars

Create a Multi-container Pod with the name sidecar-pod, that runs in the ckad-ns3 Namespace.
- The primary container is running busybox, and writes the ouput of the **date** command to the /var/log/date.log file every 5 seconds.
- The second container should run as a sidecar and provide nginx web-access to this file, using an hostPath shared volume. (mount on /usr/share/nginx/html).
- Make sure the image for this container is only pulled if it's not available on the local system yet.

#### Question 5 - Fixing a Deployment

- Start the Deployment from the redis.yaml file in the course Git repository. Fix any problems that may occur while starting it.

#### Question 6 - Using Probes

Create a Pod that runs the nginx webserver.
- The webserver should be offering its services on port 80 and run in the ckad-ns3 Namespace
- This Pod should check the /healthz path on the API-server before starting the main container.

#### Question 7 - Creating a Deployment

Write a manifest file with the name nginx-exam.yaml that meet's the following requirements.
- It starts 5 replicas that run the nginx:1.18 image.
- Each Pod has the label type=webshop
- Create the Deployment such that while updating, it can temporarily run 8 application instances at the same time, of which 3 should always be avaialbe.
- The Deployment itself should use the label service=nginx
- Update the Deployment to the latest version of the nginx image.

#### Question 8 - Exposing Applications

In the ckad-ns6 Namespace, create a Deployment that runs the nginx 1.19 image and give it the name nginx-deployment
- Ensure it runs 3 replicas.
- After verfying that the Deployment runs successfully, expose it such that users that are external to the cluster can reach it by addressing the Node Port 32000 on the Kubernetes Cluster node.
- Configure Ingress to access the application at mynginx.info

#### Question 9 - Using NetworkPolicies

Create a YAML file with the name my-nw-policy that runs two Pods and a NetworkPolicy.
- The first Pod should run an Nginx server with default settings.
- The second Pod should run a busybox image with the sleep 3600 command.
- Use a NetworkPolicy to restrict traffic between Pods in the following way.
  - Access to the nginx server is allowed for the busybox Pod.
  - The busybox Pod is not restricted in any way.

#### Question 10 - Using Storage

All objects in this assignment should be created in the ckad-1311 Namespace.
- Create a PersistentVolume with the name 1311-pv. It should provide 2 GiB of storage and read/write access to multiple clients simultaneously. use the hostPath storage type.
- Next, create a PersistentVolumeClaim that requests 1 GiB from any PersistentVolume that allows multiple clients simultaneously read/write access. The name of the object should be 1311-pvc
- Finally, create a Pod with the name 1311-pod that uses this PersistentVolume. It should run an nginx image and mount the volume on the directory /webdata

#### Question 11 - Using Quota

- Create a Namespace with the name limited, in which 5 Pods can be started and a total amount of 1000 millicore and 2 GiB of RAM is available.
- Run a Deployment with the name restrictnginx in this Namespace, with 3 Pods where every Pod initially requests 64 MiB RAM, with an upper limit of 256 MiB RAM.

#### Question 12 - Creating Canary Deployments

- Run a Deployment with the name myweb, using the nginx:1.14 image and 3 replicas. Ensure this Deployment is accessible through a Service with the name canary, which uses the NodePort Service type.
- Update the Deployment to the latest version of Nginx, using the canary Deployment update strategy, in such a way that 40% of the application offers access to the updated application and 60% still uses the old application.

#### Question 13 - Managing Pod Permissions

- Create a Pod manifest file to run a Pod with the name sleepybox. It should run the latest version of busybox, with the sleep 3600 command as the default command. Ensure the primary Pod user is a member of the supplementary group 2000 while this Pod is started.

#### Question 14 - Using a ServiceAccount

- Create a Pod with the name allaccess. Also create a ServiceAccount with the name allaccess and ensure that the Pod is using the ServiceAccount. Notice that no further RBAC setup is required.

### 15.3 Working with Namespaces

Create a Namespace ckad-ns1 in your cluster. In this Namespace, run the following Pods.
- A Pod with the name pod-a, running the httpd server image.
- A Pod with the name pod-b, running the nginx server image as well as the alpine image

```bash
# Create Namespace - ckad-ns1
kubectl create ns ckad-ns1
kubectl get ns

# Run pod-a with httpd server image in Namespace ckad-ns1 
kubectl run pod-a -n ckad-ns1 --image=httpd 
kubectl get pods -n ckad-ns1

# Run pod-b with nginx server and alpine image in Namespace ckad-ns1
kubectl run pod-b -n ckad-ns1 --image=alpine --dry-run=client -o yaml -- sleep 3600 > task1.yaml
# Edit task1.yaml to add nginx server image
vim task1.yaml
# 1. Add below content under resources: {}
- name: nginx
  image: nginx
# 2. Save and close the file.
kubectl create -f task1.yaml
kubectl get pods -n ckad-ns1

# Verify the images alpine and nginx.
kubectl describe -n ckad-ns1 pod pod-b
```

### 15.4 Using Secrets

- Create a Secret that defines the variable password=secret. Create a Deployment with the name secretapp, which starts the nginx image and uses this variable.

```bash
# Create a secret 
kubectl create secret generic -h | less
kubectl create secret generic my-secret --from-literal=password=secret
kubectl get secrets
kubectl describe secret my-secret

# Create a Deployment with the name secretapp
kubectl create deploy secretapp --image=nginx
kubectl get deploy

# Add secret to deployment.
kubectl set env -h | less
kubectl set env --from=secret/my-secret deployment/secretapp
kubectl describe deploy secretapp

# Verify
kubectl get pods # make sure secretapp is running.
kubectl exec secretapp-54df8db4d-969pt -- env # make sure PASSWORD=secret is present.
```

### 15.5 Creating Custom Images

- Create a Dockerfile that runs an alpine image with the command "echo hello world" as the default command. Build the image, and export it in OCI format to a tar file with the name "greetworld"

```bash
# Create a Dockerfile
vim Dockerfile

    FROM alpine
    CMD ["echo", "hello world"]

# Build docker image.
docker build -t greetworld .
docker images

# save the docker image.
docker save --help
docker save -o greetworld.tar greetworld
ls -l
```

### 15.6 Using Sidecars

Create a Multi-container Pod with the name sidecar-pod, that runs in the ckad-ns3 Namespace.
- The primary container is running busybox, and writes the ouput of the **date** command to the /var/log/date.log file every 5 seconds.
- The second container should run as a sidecar and provide nginx web-access to this file, using an hostPath shared volume. (mount on /usr/share/nginx/html).
- Make sure the image for this container is only pulled if it's not available on the local system yet.

```bash
# Create Namespace - ckad-ns3
kubectl create ns ckad-ns3
kubectl get ns

# Create a Multi-container Pod with the name sidecar-pod
# ref: https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
# hostPath
#

vim task156.yaml

---
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
  namespace: ckad-ns3
spec:

  restartPolicy: Never

  volumes:
  - name: shared-data
    hostPath:
      path: /mydata

  containers:

  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: busybox-container
    image: busybox
    volumeMounts:
    - name: shared-data
      mountPath: /var/log
    args:
      - sh
      - -c
      - while sleep 5; do date >> /var/log/date.log; done  
...

# Generate command args.
kubectl run busybox --image=busybox --dry-run=client -o yaml -- sh -c "while sleep 5; do date >> /var/log/date.log; done"

# Create sidecar-pod
kubectl create -f task156.yaml
kubectl get po -n ckad-ns3

# Verify
kubectl exec -it sidecar-pod -n ckad-ns3 sh
cat /usr/share/nginx/html/date.log
```

### 15.7 Fixing a Deployment

- Start the Deployment from the redis.yaml file in the course Git repository. Fix any problems that may occur while starting it.

```bash
vim redis.yaml
# 1. Change apps/v1beta1 to apps/v1
# 2. Number of replicas is missing, set it to `1`

# Create Deployment
kubectl create -f redis.yaml
kubectl get deploy
kubectl get pods --selector app=redis
```

### 15.8 Using Probes

Create a Pod that runs the nginx webserver.
- The webserver should be offering its services on port 80 and run in the ckad-ns3 Namespace
- This Pod should check the /healthz path on the API-server before starting the main container.

```bash
# Open https://kubernetes.io/docs
# Search for `healthz api`
# Click `Kubernetes API health endpoints`
# Copy the curl command and 
# 1. Change the localhost to minkikube ip address 
# 2. Change the port from 6443 to 8443 
# 3. Change livez to healthz
curl -k https://192.168.49.2:8443/healthz?verbose
echo $?

# Find kube-apiserver published ip address and port.
minikube ssh
ps aux | grep api # look for --advertise-address=
exit

# Another way to find out the kube-apiserver ip address.
kubectl get pods -n kube-system -o wide # Look for the IP for kube-apiserver.

# Open https://kubernetes.io/docs
# Search for probes
# Click Configure Liveness, Readiness and Startup Probes

vim task158.yaml

---
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
  namespace: ckad-ns3
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 80
    readinessProbe:
      exec:
        command:
        - curl
        - -k
        - https://192.168.49.2:8443/healthz
      initialDelaySeconds: 5
      periodSeconds: 5
...
kubectl explain pod.spec.containers.ports

kubectl create -f task158.yaml
kubectl get po -n ckad-ns3
```

### 15.9 Creating a Deployment

Write a manifest file with the name nginx-exam.yaml that meet's the following requirements.
- It starts 5 replicas that run the nginx:1.18 image.
- Each Pod has the label type=webshop
- Create the Deployment such that while updating, it can temporarily run 8 application instances at the same time, of which 3 should always be avaialbe.
- The Deployment itself should use the label service=nginx
- Update the Deployment to the latest version of the nginx image.

```bash
# Create the initial deployment manifest file.
kubectl create deploy nginx-exam --image=nginx:1.18 --dry-run=client -o yaml > nginx-exam.yaml

# Modify the manifest file as per the requirements.
vim nginx-exam.yaml

# 1. Change the replicas to 5
replicas: 5
# 2. Add labels for pods. spec.labels type=webshop
type: webshop
# 3. Temporarily run 8 application 
kubectl explain --recursive deployment.spec.strategy

# Copy below lines from the above command output.
  rollingUpdate	<RollingUpdateDeployment>
    maxSurge	<IntOrString>
    maxUnavailable	<IntOrString>
# Modify it as below and put it spec.strategy.
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
# 4. Add service label for deployment metadata.labels service=nginx
type: nginx

# Final YAML code looks like below.
vim nginx-exam.yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-exam
    type: nginx
  name: nginx-exam
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-exam
  strategy:
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-exam
        type: webshop
    spec:
      containers:
      - image: nginx:1.18
        name: nginx
        resources: {}
status: {}
...

# Create deployment.
kubectl create -f nginx-exam.yaml
kubectl get all --selector app=nginx-exam

# Update the deployment to the latest version.
kubectl set image -h | less
kubectl set image deployment/nginx-exam nginx=nginx:latest
kubectl get all --selector app=nginx-exam
```

### 15.10 Exposing Applications

In the ckad-ns6 Namespace, create a Deployment that runs the nginx 1.19 image and give it the name nginx-deployment
- Ensure it runs 3 replicas.
- After verfying that the Deployment runs successfully, expose it such that users that are external to the cluster can reach it by addressing the Node Port 32000 on the Kubernetes Cluster node.
- Configure Ingress to access the application at mynginx.info

```bash
sudo vim /etc/hosts # No need to do this in the exam.
192.168.49.2   mynginx.info # it is a minikube IP address.

# Note: No need to instatll Ingress in the exam.
minikube addons list # Install ingress addon if it is not enabled.
minikube addons enable ingress
minikube addons list # Make sure ingress is enabled now.

# Create namespace ckad-ns6
kubectl create ns ckad-ns6
kubectl get ns

# Create deployment with nginx:1.19, replicas 3, namespace ckad-ns6.
kubectl create deploy nginx-deployment --image=nginx:1.19 -r 3 -n ckad-ns6
kubectl get deploy -n ckad-ns6
kubectl get all -n ckad-ns6

# Expose
kubectl expose -n ckad-ns6 deployment nginx-deployment --port=80
#kubectl expose -n ckad-ns6 deployment nginx-deployment --port=80 --target-port=32000
kubectl get svc -n ckad-ns6

# Change the cluster type from ClusterIP to NodePort and add nodePort: 32000 under "port: 80"
kubectl edit -n ckad-ns6 svc nginx-deployment

# Check the cluster type.
kubectl get svc -n ckad-ns6
kubectl get ep -n ckad-ns6

# Create ingress in namespce ckad-ns6
kubectl create ingress -h | less
kubectl create -n ckad-ns6 ingress nginxdeploy --class=default --rule="mynginx.info/=nginx-deployment:80"
kubectl get ingress -n ckad-ns6

# Verify the ingress
curl mynginx.info # TODO: Should work on port 80, instead of port 32000 
```

### 15.11 Using NetworkPolicies

Create a YAML file with the name my-nw-policy that runs two Pods and a NetworkPolicy.
- The first Pod should run an Nginx server with default settings.
- The second Pod should run a busybox image with the sleep 3600 command.
- Use a NetworkPolicy to restrict traffic between Pods in the following way.
  - Access to the nginx server is allowed for the busybox Pod.
  - The busybox Pod is not restricted in any way.

```bash
# Open https://kubernetes.io/docs
# Search for `Network Policies`
# Click `Network Policies`
# Copy the sample YAML code.
# vim my-nw-policy.yaml # Paste the code into this file.

# Open https://kubernetes.io/docs
# Search for Pod`
# Click `Pod`
# Copy the sample YAML code
# Paste twice into my-nw-policy.yaml by separating with ---
# Delete these two lines in nginx container.
    ports:
    - containerPort: 80
# Modify the next section to run busybox container.
# Generate command args to run sleep 3600 command.
kubectl run busyboxy --image=busybox --dry-run=client -o yaml -- sleep 3600
# Copy these lines and paste it the file.
  - args:
    - sleep
    - "3600"
# Modify it as below.
  args:
  - sleep
  - "3600"
# Add label for nginx.
metadata.labels.app: nginx

# Add label for busybox.
metadata.labels.access: allowed

# Change nework policy.
spec.role: db to spec.app: nginx

# policyType:
Delete Egress, need only - Ingress

# Delete ipBlock and namespaceSelector sections.
# Delete all content under role: frontend
# Save and close the file.

# Create 
kubectl create -f my-nw-policy.yaml

# Create service
kubectl expose pod nginx --port=80

# Test it.
kubectl exec -it busybox -- wget --spider --timeout=1 nginx
# Ouput:
Connecting to nginx (10.111.10.143:80)
remote file exists

# Add label to nginx pod
kubectl label pod nginx role=frontend

# Test it again.
kubectl exec -it busybox -- wget --spider --timeout=1 nginx
# Ouput:
Connecting to nginx (10.111.10.143:80)
remote file exists
```

### 15.12 Using Storage

All objects in this assignment should be created in the ckad-1311 Namespace.
- Create a PersistentVolume with the name 1311-pv. It should provide 2 GiB of storage and read/write access to multiple clients simultaneously. use the hostPath storage type.
- Next, create a PersistentVolumeClaim that requests 1 GiB from any PersistentVolume that allows multiple clients simultaneously read/write access. The name of the object should be 1311-pvc
- Finally, create a Pod with the name 1311-pod that uses this PersistentVolume. It should run an nginx image and mount the volume on the directory /webdata

```bash
# Open https://kubernetes.io/docs
# Search for `Persistent volumes`
# Click `Configure a Pod to use a PersistentVolume for Storage`
# Copy the Create a PersistentVolume code and paste it into the task1512.yaml file and put ---
# Copy the PersistentVolumeClaim code and paste it into the task1512.yaml file and put ---
# Copy the Create a Pod code and paste it into the task1512.yaml file and put

# Modify the code as per the requirement.
# PersistentVolume
metadata.namespace: ckad-1312 # Do it in all 3 sections.
metadata.name: 1312-pv
spec.capacity.storage: 2Gi
accessModes: ReadWriteMany

# PersistentVolumeClaim
name: 1312-pvc
accessModes: ReadWriteMany
storage: 1Gi

# Pod
name: 1312-pod
claimName: 1312-pvc
mountPath: "/webdata"

# Create a namespace ckad-1312
kubectl create ns ckad-1312
kubectl get ns

# Create a Pod
kubectl create -f task1512.yaml
kubectl get pod,pvc,pv -n ckad-1312

# Create a test file in side Pod
kubectl exec -n ckad-1312 -it 1312-pod -- touch /webdata/testfile
kubectl exec -n ckad-1312 -it 1312-pod -- ls /webdata/testfile

# Verify the testfile in minikube server.
minikube ssh
ls /mnt/data/ # testfile should be present
exit
```

### 15.13 Using Quota

- Create a Namespace with the name limited, in which 5 Pods can be started and a total amount of 1000 millicore and 2 GiB of RAM is available.
- Run a Deployment with the name restrictnginx in this Namespace, with 3 Pods where every Pod initially requests 64 MiB RAM, with an upper limit of 256 MiB RAM.

```bash
# Create a namespace limited
kubectl create ns limited

# Create a quota
kubectl create quota -h | less
kubectl create quota limitedquota -n limited --hard=cpu=1,memory=2G,pods=5
kubectl describe ns limited

# Create deployment
kubectl create deploy restrictnginx -n limited --image=nginx --replicas=3

# Set resources
kubectl set resources -h | less
kubectl set resources -n limited deployment restrictnginx --limits=memory=256Mi --requests=memory=64Mi
kubectl set resources -n limited deployment restrictnginx --limits=cpu=1 --requests=cpu=1
kubectl describe -n limited rs restrictnginx-xxx-yyy

kubectl describe ns limited # Used shoud be in millicore, not 1.
kubectl set resources -h | less
kubectl set resources -n limited deployment restrictnginx --limits=cpu=200m --requests=cpu=200m
kubectl describe ns limited # If Used is not in millicore, delete deployment and recreate it.

# Delete deployment
kubectl delete deploy restrictnginx -n limited

# Recreate deployment and resources
kubectl create deploy restrictnginx -n limited --image=nginx --replicas=3
kubectl set resources -n limited deployment restrictnginx --limits=memory=256Mi,cpu=200m --requests=memory=64Mi,cpu=200m
kubectl describe ns limited
```

### 15.14 Creating Canary Deployments

- Run a Deployment with the name myweb, using the nginx:1.14 image and 3 replicas. Ensure this Deployment is accessible through a Service with the name canary, which uses the NodePort Service type.
- Update the Deployment to the latest version of Nginx, using the canary Deployment update strategy, in such a way that 40% of the application offers access to the updated application and 60% still uses the old application.

```bash
# Create deployment
kubectl create deploy myweb --image=nginx:1.14 --replicas=3

# Add labels
kubectl edit deploy myweb
metadata.labels.type: canary
spec.template.labels.type: canary
# - Save and close the file.

kubectl get all --selector type=canary

# Create a service
kubectl expose deploy myweb --selector type=canary --port=80
kubectl describe svc myweb

# Update the deployment to latest version. 40% of the application offers access to the updated application and 60% still uses the old application.
kubectl create deploy mynewweb --image=nginx --replicas=2

# Add labels
kubectl edit deploy mynewweb
metadata.labels.type: canary
spec.template.labels.type: canary

# Veify
kubectl describe svc myweb # In the Endpoints, now it show as + 2 more...
```

### 15.15 Managing Pod Permissions

- Create a Pod manifest file to run a Pod with the name sleepybox. It should run the latest version of busybox, with the sleep 3600 command as the default command. Ensure the primary Pod user is a member of the supplementary group 2000 while this Pod is started.

```bash
# Generate manifest file.
kubectl run sleepybox --image=busybox --dry-run=client -o yaml -- sleep 3600 > task1515.yaml
kubectl explain pod.spec.securityContext | less

# Modify the manifest file.
vim task1515.yaml # Add below two lins under dnsPolicy:

securityContext:
    fsGroup: 2000

# Create a Pod
kubectl create -f task1515.yaml

# Verify
kubectl get pods sleepybox -o yaml
```

### 15.16 Using ServiceAccount

- Create a Pod with the name allaccess. Also create a ServiceAccount with the name allaccess and ensure that the Pod is using the ServiceAccount. Notice that no further RBAC setup is required.

```bash
# Create a ServiceAccount allaccess
kubectl create sa allaccess

# Create a Pod
kubectl run allaccess --image=busybox --dry-run=client -o yaml -- sleep 3600 > task1516.yaml

# Add ServiceAccount
vim task1516.yaml # Add below 2 lines to containers section.

serviceAccount: allaccess
serviceAccountName: allaccess

# Create a Pod
kubectl create -f task1516.yaml

# Verify
kubectl get pods allaccess -o yaml 
kubectl get pods allaccess -o yaml | grep serviceAccount
```
