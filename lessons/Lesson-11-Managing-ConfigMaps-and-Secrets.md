# Lesson 11 - Managing ConfigMaps and Secrets

- [Lesson 11 - Managing ConfigMaps and Secrets](#lesson-11---managing-configmaps-and-secrets)
    - [11.1 Providing Variables to Kubernetes Applications](#111-providing-variables-to-kubernetes-applications)
      - [Providing Variables to Kubernetes Applications](#providing-variables-to-kubernetes-applications)
      - [Demo: Generating a YAML File with Variables](#demo-generating-a-yaml-file-with-variables)
    - [11.2 Understanding Why Decoupling is Important](#112-understanding-why-decoupling-is-important)
      - [Understanding the Need for Decoupling](#understanding-the-need-for-decoupling)
      - [Understanding ConfigMap](#understanding-configmap)
    - [11.3 Providing Variables with ConfigMaps](#113-providing-variables-with-configmaps)
      - [Providing Variables with ConfigMaps](#providing-variables-with-configmaps)
      - [Demo: Providing Variables with CongigMaps](#demo-providing-variables-with-congigmaps)
    - [11.4 Providing Configuration Files Using ConfigMaps](#114-providing-configuration-files-using-configmaps)
      - [Using ConfigMaps for ConfigFiles](#using-configmaps-for-configfiles)
      - [Mounting a ConfigMap in an Application](#mounting-a-configmap-in-an-application)
      - [Demo: Using a CongigMap with a Configuration File](#demo-using-a-congigmap-with-a-configuration-file)
    - [11.5 Understanding Secrets](#115-understanding-secrets)
      - [Understanding Secrets](#understanding-secrets)
      - [Understanding Secret Types](#understanding-secret-types)
    - [11.6 Understanding How Kubernetes Uses Secrets](#116-understanding-how-kubernetes-uses-secrets)
      - [Understanding Kubernetes Secret Use](#understanding-kubernetes-secret-use)
      - [Demo: Exploring Kubernetes Secret Use](#demo-exploring-kubernetes-secret-use)
    - [11.7 Configuring Applications to Use Secrets](#117-configuring-applications-to-use-secrets)
      - [Using Secrets in Applications](#using-secrets-in-applications)
      - [Using Secrets in Applications](#using-secrets-in-applications-1)
      - [Demo: Using a Secret to Provide Passwords](#demo-using-a-secret-to-provide-passwords)
    - [11.8 Configuring the Docker Registry Access Secret](#118-configuring-the-docker-registry-access-secret)
      - [Understanding Registry Access](#understanding-registry-access)
      - [Demo: Configuring a docker-registry Secret](#demo-configuring-a-docker-registry-secret)
    - [Lesson 11 Lab Using ConfigMaps and Secrets](#lesson-11-lab-using-configmaps-and-secrets)
    - [Lesson 11 Lab SolutionUsing ConfigMaps and Secrets](#lesson-11-lab-solutionusing-configmaps-and-secrets)

### 11.1 Providing Variables to Kubernetes Applications

#### Providing Variables to Kubernetes Applications

- Kubernetes does not offer a command line option to provide variables while running a Deployment with **kubectl create deploy**
  - First, use **kubectl create deploy mydb --image=mariadb**
  - Next, use **kubectl set env deploy mydb MYSQL_ROOT_PASSWORD=password**
- While running a Pod, environment variables can be provided, but you shouldn't run naked Pods.
  - **kubectt run mydb --image=mysql-- env="MYSQL_ROOT_PASSWORD=password"**

#### Demo: Generating a YAML File with Variables

```bash
kubectl create deploy mydb --image=mariadb
kubectl get all
kubectl describe pods mydb-xxx-yyy
kubectl logs mydb-xxx-yyy
kubectl set env deploy mydb MYSQL_ROOT_PASSWORD=password
kubectl get all
kubectl get deploy mydb -o yaml > mydb.yaml # don't forget to clean it up!
vim mydb.yaml 
1. Delete 4 lines from annotations
2. Delete resourceVersion: and uid
3. Delete complete status: information 
4. Save and close the file.

# Dry run
kubectl create deploy newdb --image=mariadb --replicas=3 --dry-run=client -o yaml > mydb.yaml
kubectl create -f mydb.yaml
kubectl set env deploy newdb MYSQL_ROOT_PASSWORD=password --dry-run=client -o yaml >> mydb.yaml # don't forget to clean it up!
```

### 11.2 Understanding Why Decoupling is Important

#### Understanding the Need for Decoupling

- It's a good idea to separate site-specific information from code.
- Code should be static, which makes it portable, so that it can be used in other environments.
- Variables are site-specific, and for that reason should not be provided in the Deployment configuration.
- Kubernetes provides ConfigMaps to deal with this issue.
- A ConfigMap can be used to define variables, and the Deployment just points to the ConfigMap.
- While using ConfigMaps, site-specific information is included in the ConfigMap, which allows you to keep the Deployment manifest files generic.

#### Understanding ConfigMap

- ConfigMaps are used to provide different types of configuration to an application.
- ConfigMaps can be used for three different types of configuration 
  - Variables
  - Configuration Files
  - Command line arguments
- Note that according to these different types of configurations, ConfigMaps are created and used in a very different way.
- If an application refers to a ConfigMap, the ConfigMap should exist in the cluster before running the application.

### 11.3 Providing Variables with ConfigMaps

#### Providing Variables with ConfigMaps 

- While creating a ConfigMap with **kubectl create cm**, variables can be provided in two ways.
  - Using **--from-env-file: kubectl create cm --from-env-file=dbvars**
  - Using **--from-literal: kubectl create vm --from-literal=MYSQL_USER=anna** 
- Notice that is's possible to use multiple **--from-literal**, you cannot use mulitple **--from-env-file**
- After creating the ConfigMap, use **kubectl set env -- from=configmap/mycm deploy/myapp** to use the ConfigMap in your Deployment
- Use **--dry-run=client** and command line redirection on **kubectl create deploy** and **kubectl set env** to generate a YAML file using ConfigMap.

#### Demo: Providing Variables with CongigMaps

```bash
vim varsfile
    MYSQL_ROOT_PASSWORD=password
    MYSQL_USER=anil
kubectl create cm -h | less
kubectl create cm mydbvars --from-env-file=varsfile
kubectl get configmaps
kubectl describe configmaps mydbvars

kubectl create deploy mydb --image=mariadb --replicas=3 
kubectl get all --selector app=mydb

kubectl set env deploy mydb --from=configmap/mydbvars
kubectl get all --selector app=mydb
kubectl get deploy mydb -o yaml
```

### 11.4 Providing Configuration Files Using ConfigMaps

#### Using ConfigMaps for ConfigFiles

- Configuration files are typically used to provide site-specific information to Applications.
- To store configuration files in cloud, CongigMap can be used.
- Use **kubectl create cm myconf --from=file=/my/file.conf**
- If a ConfigMap is created from a directory, all files in that directory are included in the CongigMap.
- To use the configuration file in an application, the CongigMap must be mounted in the application.
- There is no easy imperative way to mount CongigMaps in applications.

#### Mounting a ConfigMap in an Application

- Tip: generate the base YAML code, and add the ConfigMap mount to it later.
- In the application manifest, define a volume using the CongigMap type.
- Mount this volume on a specific directory.
- The configuration file will apear inside the directory.

#### Demo: Using a CongigMap with a Configuration File

```bash
echo "hello world" > index.html
kubectl create cm myindex --from-file=index.html
kubectl describe cm myindex

kubectl create deploy myweb --image=nginx
kubectl get all --selector app=myweb

kubectl edit deploy myweb
spec.template.spec
volumes:
- name: cmvol
  configMap:
    name: myindex
spec.template.spec.containers
volumeMounts:
- name: cmvol
  mountPath: /usr/share/nginx/html
  
kubectl get all --selector app=myweb
kubectl describe pod myweb-xxx-yyy
kubectl exec myweb-xxx-yyy -- cat /usr/share/nginx/html/index.html
```

### 11.5 Understanding Secrets

#### Understanding Secrets

- Secrets allow for storage of sensitive data such as passwords, Auth tokens and SSH keys.
- Using Secrets makes it so that data doesn't have to be put in a Pod, and reduces the risk of accidental exposure.
- Some Secrets are automatically created by the system, users can also use secrets.
- System-created Secrets are important for Kubernetes resources to connect to other cluster resources.
- Secrets are not encrypted, they are base64 encoded.

#### Understanding Secret Types

- Three types of Secret are offered.
  - docker-registry: used for connecting to a Docker registry.
  - TLS: used to store TLS key material.
  - generic: creates a secret from a local file, directory or leteral value.
- When defining the Secret, the type must be specified: **kubectl create secret generic ...**

### 11.6 Understanding How Kubernetes Uses Secrets

#### Understanding Kubernetes Secret Use

- To access the Kubernetes API, all Kubernetes resources need access to TLS keys.
- These keys are provided by Secrets and used through ServiceAccounts.
- By using the ServiceAccount, the application gets access to its Secret.

#### Demo: Exploring Kubernetes Secret Use

```bash
kubectl get pods -n kube-system corredns-xxx-yyy -o yaml # looks for ServiceAccount
kubectl get sa -n kube-system coredns -o yaml # look for secrets
kubectl get secret -n kube-system coredns-token-xxxx -o yaml # look at the TLS materials provided.
```

### 11.7 Configuring Applications to Use Secrets

#### Using Secrets in Applications

- There are different use cases for using Secrets in applications.
  - To provide TLS keys to the application: **kbuectl create secret tls my-tls-keys --cert=tls/my.crt --key=tls/my.key**
  - To provide security to passwords: **kubectl create secret generic my-secret-pw --from-literal=password=verysecret**
  - To provide access to an SSH private key: **kubectl create secret generic my-ssh-key --from-file=ssh-private-key=.ssh/id_rsa**
  - To provide access to sensitive files, which would be mounted in the application with root access only: **kubectl create secret generic my-secret-file --from-file=/my/secretfile**

#### Using Secrets in Applications

- As a Secret basically is an encoded ConfigMap, it is used in a way similar to using ConfigMaps in applications.
- If it contains variables, use **kubectl set env**
- If it contains files, mount the Secret
- While mounting the Secret in the Pod spec, consider using **defaultMode** to set the permissionsmode: **defaultMode: 0400**
- Notice that mounted Secrets are automatically updated in the application when the Secret is updated.

#### Demo: Using a Secret to Provide Passwords

```bash
kubectl create secret generic -h | less
kubectl create secret generic dbpw --from-literal=ROOT_PASSWORD=password
kubectl get secret dbpw
kubectl describe secret dbpw
kubectl get secret dbpw -o yaml
kubectl create deploy mynewdb --image=mariadb
kubectl set env deploy mynewdb --from=secret/dbpw --prefix=MYSQL_
kubectl get all
kubectl exec mynewdb-xxx-yyy -- env
```

### 11.8 Configuring the Docker Registry Access Secret

#### Understanding Registry Access

- To get access to container registries, additional authentication may be required.
- On a stand-alone container node, these credentials would be stored in a local config file.
- In a cloud native environment, the information should be decoupled from the nodes and stored in the cloud.
- Use the **docker-registry** Secret type to store the authentication credentials in a Secret.

#### Demo: Configuring a docker-registry Secret

```bash
export DOCER_USERNAME=
export DOCER_PASSWORD=
export DOCER_EMAIL=

kubectl create secret docker-registry my-docker-credentials --docker-username=anil --docker-password=secretpw --docker-email=anil@gmail.com --docker-server=docker.io

kubectl create secret docker-registry my-docker-credentials --docker-username=$DOCER_USERNAME --docker-password=$DOCER_PASSWORD --docker-email=$DOCER_EMAIL --docker-server=docker.io

kubectl get secrets
kubectl describe secrets my-docker-credentials
kubectl get secret my-docker-credentials -o yaml
```

### Lesson 11 Lab Using ConfigMaps and Secrets

- Create a simple index.html file, with the contents "hello world"
- Create a ConfigMap that contains the contents of the file index.html.
- Create a Secret that contains the variable defination **MYPASSWORD=verysecret**
- Create a Manifest file that starts an Nginx Deployment that mounts the index.html file from the ConfigMap on /usr/share/nginx/html, and sets the environment variable from the Secret.

### Lesson 11 Lab SolutionUsing ConfigMaps and Secrets

```bash
# Create index.html file.
echo "hello world" > index.html

# Create ConfigMap.
kubectl create cm lab11cm --from-file=index.html
kubectl describe cm lab11cm
kubectl get cm lab11cm -o yaml

# Create secret.
kubectl create secret generic lab11secret --from-literal=MYPASSWORD=verysecret
kubectl get secrets
kubectl get secret lab11secret -o yaml

# Create deployment.
kubectl create deploy lab11deploy --image=nginx
kbuectl set env deploy lab11deploy --from=secret/lab11secret 
kubectl get deploy
kubectl describe deployments.apps lab11deploy

# Create a manifest file.
kubectl edit deploy lab11deploy
spec.template.spec
volumes:
- name: lab11
  configMap: 
    name: lab11cm
spec.template.spec.containers
volumeMounts:
- name: lab11
  mountPath: /usr/share/nginx/html

kubectl get deploy lab11deploy -o yaml > lab11deploy.yaml
kubectl delete deploy lab11deploy
vim lab11deploy.yaml
1. Delete 4 lines from annotations
2. Delete resourceVersion: and uid
3. Delete complete status: information 
4. Save and close the file.

# Create deployment from the manifest file.
kubectl create -f lab11deploy.yaml
kbuectl exec lab11deploy-xxx-yyy -- cat /usr/share/nginx/html/index.html
kbuectl exec lab11deploy-xxx-yyy -- env
```
