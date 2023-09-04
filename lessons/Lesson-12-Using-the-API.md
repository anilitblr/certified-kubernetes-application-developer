# Lesson 12 - Using the API

- [Lesson 12 - Using the API](#lesson-12---using-the-api)
    - [12.1 Understanding the Kubernetes API](#121-understanding-the-kubernetes-api)
      - [Understanding the API](#understanding-the-api)
      - [Understanding API Access](#understanding-api-access)
      - [Understanding kube-proxy](#understanding-kube-proxy)
      - [Connecting to the API](#connecting-to-the-api)
    - [12.2 Using curl to Work with API Objects](#122-using-curl-to-work-with-api-objects)
      - [Demo: Using curl to Access API Resouves](#demo-using-curl-to-access-api-resouves)
    - [12.3 Understanding API Deprecations](#123-understanding-api-deprecations)
      - [Understanding API Deprecations](#understanding-api-deprecations)
      - [Demo: Dealing with Deprecation](#demo-dealing-with-deprecation)
    - [12.4 Understanding Authentication and Authorization](#124-understanding-authentication-and-authorization)
      - [Understanding Authentication](#understanding-authentication)
      - [Understanding Authorization](#understanding-authorization)
      - [Understanding RBAC](#understanding-rbac)
      - [Demo: Showing Current Authorization](#demo-showing-current-authorization)
    - [12.5 Understanding API Access and ServiceAccounts](#125-understanding-api-access-and-serviceaccounts)
      - [Understanding ServiceAccounts](#understanding-serviceaccounts)
      - [Understanding ServiceAccount Secrets](#understanding-serviceaccount-secrets)
    - [12.6 Understanding Role Based Access Control](#126-understanding-role-based-access-control)
      - [Understanding RBAC](#understanding-rbac-1)
    - [12.7 Configuring a ServiceAccount](#127-configuring-a-serviceaccount)
      - [Managing ServiceAccount](#managing-serviceaccount)
      - [Creating Additional Service Accounts](#creating-additional-service-accounts)
      - [Demo: Using Service Accounts](#demo-using-service-accounts)
      - [ServiceAccount on the CKAD Test](#serviceaccount-on-the-ckad-test)
    - [Lesson 12 Lab Configuring a Service Account](#lesson-12-lab-configuring-a-service-account)
    - [Lesson 12 Lab Solution Configuring a Service Account](#lesson-12-lab-solution-configuring-a-service-account)

### 12.1 Understanding the Kubernetes API

#### Understanding the API

- The API defines which resources exist, and how to use these resources.
- Kubernetes uses a core API, which is extensible.
- This allows additional APIs to be added in API Groups.
- This flexibility is used in Custom Resource Definition (CRD) to add resources to Kubernetes.
- Some Kubernetes distributions have added many CRDs.
- When installing Operators, CRDs are typically involved.
- Use **kubectl api-resources** for an overview of APIs and resources defined in specific APIs.
```bash
kubectl api-resources
```
- The main API documentation is here: https://kubenetes.io/docs/reference/generated/kubernetes-api/v1.22
  - Change to match latest version
- Each API group can have its own version number.
  - See here for more information: https://kubernetes.io/docs/reference/using-api/api-overview/
- Use **kubectl api-resources** for information about resource types.
- Or **kubectl api-versions** for resource and version information.

#### Understanding API Access

- Kubernetes functionality is exposed by the **kube-apiserver** core Kubernetes process.
- The **kube-apiserver** is typically started as a systemd process.
- The **kube-apiserver** allows TLS certificate-based access only.
- **kubectl** makes secured API requests.

#### Understanding kube-proxy

- The Kubernetes APIs are RESTful
- This means that they can be accessed using direct HTTP traffic.
- **curl** can be used for direct API access, which is needed when **kubectl** is not available.
- Behind the scenes, **kubectl** uses **curl** as well.
- Use **kubectl --v=10 get pods** (or any other command) to see the underlying API request.
- To make your own API requests using **curl**, the appropriate certificates are required.
- Use **kube-proxy** to provide a secure interface for **curl**

#### Connecting to the API

- To access the API using **curl**, start the kube-proxy on the kubernetes user workstation
  - **kubectl proxy --port=8001&**
  - curl http://localhost:8001
- This shows all the available API paths and groups, providing access to all exposed functions.
```bash
kubectl -v=10 get pods
kubectl proxy --port=8001&
http://localhost:8001
fg
Ctrl+C
```

### 12.2 Using curl to Work with API Objects

#### Demo: Using curl to Access API Resouves

```bash
# On the host that runs kubectl
kubectl proxy --port=8001&

kubectl create deploy curlnginx --image=nginx --replicas=3

curl http://localhost:8001/version
curl http://localhost:8001/api/v1/namespaces/default/pods # shows the Pods.
curl http://localhost:8001/api/v1/namespaces/default/pods/curlnginx-xxx-yyy # shows direct API access to a Pod.
curl -XDELETE http://localhost:8001/api/v1/namespaces/default/pods/curlnginx-xxx-yyy # will delete the httpd Pod.
cat ~/.kube/config
```

### 12.3 Understanding API Deprecations

#### Understanding API Deprecations

- With new Kubernetes releases, old API versions may get deprecated.
- If an old version gets deprecated, it will be supported fo a minimum of two more Kubernetes releases.
- When you see a deprecation message, make sure to take action and change your YAML manifest files.

#### Demo: Dealing with Deprecation

```bash
kubectl create -f redis-deploy.yaml
kubectl api-versions
kubectl explain --recursive deploy
kubectl exec -it podname-xxx-yyy /bin/bash
```

### 12.4 Understanding Authentication and Authorization

#### Understanding Authentication

- Authentication is about where Kubernetes users come from.
- By default, a local Kubernetes admin account is used, and authentication is not required.
- In more advanced setups, you can create your own user accounts (covered in CKA)
- The **kubectl** config specifies to which cluster to authenticate.
  - Use **kubectl config view** to see current settings.
- The config is read from ~/.kube/config

#### Understanding Authorization

- Authorization is what these users can do.
- Behind authorization, there is Role-Based Access Control (RBAC) to take care of the different options.
- Use **kubectl auth can-i ...** (like **kubectl auth can-i get pods**) to find out what you can do.

#### Understanding RBAC

- RBAC is used to provide access to API Resources.
- In RBAC, 3 elements are used.
  - The Role defines access permissions to specific resources
  - The user or ServiceAccount is used as an entity in Kubernetes to work with the API.
  - The RoleBinding connects a user or ServiceAccount to a specific Role.

#### Demo: Showing Current Authorization

```bash
kubectl config view
less ~/.kube/config
kubectl auth can-i get pods
kubectl auth can-i get pods --as anil@lab.com
```

### 12.5 Understanding API Access and ServiceAccounts

#### Understanding ServiceAccounts

- All actions in a Kubernetes Cluster need to be authenticated and authorized.
- ServiceAccounts are used for basic authentication from within the Kubernetes cluster.
- Role Based Access Control (RBAC) is used to connect a ServiceAccount (SA) to a specific Role.
- Every Pod uses the default ServiceAccount to contact the API server.
- This default ServiceAccount allows a resource to get information from the API server, but not much else.
- Each ServiceAccount uses a secret to automount API credentials.

#### Understanding ServiceAccount Secrets

- ServiceAccount come with a Secret .
- This Secret contains API credentials.
- By specifiying the ServiceAccount to be used by a Pod, the ServiceAccount Secret is auto-mounted to provide API access credentials according to the ServiceAccount RBAC.
```bash
kubectl get pods busybox -o yaml
kubectl get sa -o yaml
kubectl describe pod busybox
kubectl get sa -A
```

### 12.6 Understanding Role Based Access Control

#### Understanding RBAC

- A Cluster Administrator can configure RBAC authorization to specify what exactly can be accessed by a ServiceAccount or user.
- Custom ServiceAccount can be created, and configured with additional permissions for enhanced access from Pods to cluster resources.
- For additional permissions, Namespace or cluster scope Roles and RoleBindings are used.
- The Role defines access permissions to API resources in specific Namespaces.
- The RoleBinding connects the ServiceAccount to a specific Role.
- ClusterRole and ClusterRoleBinding are used to configure access to cluster resources.

```bash
cd labs/
vim list-pods.yaml
kubectl create -f list-pods.yaml
vim list-pods-mysa-binding.yaml
kubectl create -f list-pods-mysa-binding.yaml
kubectl get sa
kubectl create sa mysa
kubectl get sa
```

### 12.7 Configuring a ServiceAccount

#### Managing ServiceAccount

- Every Namespaces has a default ServiceAccount called default.
- Additional ServiceAccount can be created to provide Additional access to resources.
- After creating the additional ServiceAccount, it needs to get permissions through RBAC.

#### Creating Additional Service Accounts

- ServiceAccount are easily created using the imperative way: **kubectl create serviceaccont mysa**
- While creating a ServiceAccount, a Secret is automatically created to have the ServiceAccount connect to the API.
- This Secret is mounted in Pods using the ServiceAccount and used as an access token.
- After creating the ServiceAccount, RBAC must be configured.

#### Demo: Using Service Accounts

```bash
# 1. Create a Pod, using the standard ServiceAccount.
cat mypod.yaml
kubectl apply -f mypod.yaml

# 2. Use to check current SA Configuration
kubectl get pods mypod -o yaml

# 3. Access the Pod using
kubectl exec -it mypod -- sh

# Try to list Pods using curl on the API.
apk add --update curl
curl https://kubernetes/api/v1 --insecure # will be forbidden

# 4. Use the Default ServiceAccount token and try again.
TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
echo $TOKEN
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/ --insecure

# 5. Try the same, but his time to list Pods - it will fail.
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
exit

# 6. Create a ServiceAccount
kubectl create -f mysa.yaml

# Describe role list-pods.
kubectl describe roles list-pods
kubectl describe rolebindings.rbac.authorization.k8s.io list-pods-mysa-binding

# 7. Define a role that allows to list all Pods in the default NameSpace.
kubectl apply -f list-pods.yaml

# 8. Define a RoleBinding that binds the mysa to the Role just created.
kubectl apply -f list-pods-mysa-binding.yaml

# 9. Create a Pod that uses the mysa SA to access this Role.
cat mysapod.yaml
kubectl create -f mysapod.yaml

# 10. Access the Pod, use the mysa ServiceAccount token and try again.
kubectl exec -it mysapod -- sh
apk add --update curl
TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
echo $TOKEN
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/ --insecure

# 11. Try the same, but his time to list Pods.
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
exit
```

#### ServiceAccount on the CKAD Test

- On the exam you will need to know how to configure an application to use a specific ServiceAccount.
- You will not have to configure any Roles or RoleBindings.
- For troubleshooting, check if additional ServiceAccounts are provided.
```bash
kubectl get sa
kubectl set serviceaccount -h | less
```

### Lesson 12 Lab Configuring a Service Account

- Create a ServiceAccount with the name **mynewsa**
- Find the easiest way to run a busybox Pod with the **sleep 3600** command that uses this ServiceAccount

### Lesson 12 Lab Solution Configuring a Service Account

```bash
kubectl create sa mynewsa
kubectl get sa
kubectl run --help | less
kubectl run busysa --image=busybox --dry-run=client -o yaml -- sleep 3600 > busysa.yaml
vim busysa.yaml # no prior service accont details, look in the documentation.
kubectl explain pods.spec | less # Look for serviceAccountName
kubectl explain pods.spec | grep serviceAccountName 

pods.spec
serviceAccountName: mynewsa

kubectl create -f busysa.yaml
kubectl get pods busysa -o yaml # Look for serviceAccountName
kubectl get pods busysa -o yaml | grep serviceAccountName
```
