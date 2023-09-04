# Lesson 10 - Managing Kubernetes Storage

- [Lesson 10 - Managing Kubernetes Storage](#lesson-10---managing-kubernetes-storage)
    - [10.1 Understanding Kubernetes Storage Options](#101-understanding-kubernetes-storage-options)
      - [Understanding Storage Options](#understanding-storage-options)
      - [Understanding Persistent Volumes](#understanding-persistent-volumes)
    - [10.2 Configuring Pod Volume Storage](#102-configuring-pod-volume-storage)
      - [Configuring Volumes](#configuring-volumes)
      - [Demo: Using Pod Local Storage](#demo-using-pod-local-storage)
    - [10.3 Configuring PV Storage](#103-configuring-pv-storage)
      - [Understanding Persistent Volumes](#understanding-persistent-volumes-1)
      - [Demo: Defining Persistent Volumes](#demo-defining-persistent-volumes)
    - [10.4 Configuring PVCs](#104-configuring-pvcs)
      - [Understanding Persistent Volume Claims](#understanding-persistent-volume-claims)
      - [Understanding PVC Binds](#understanding-pvc-binds)
      - [Demo: Using PVC](#demo-using-pvc)
    - [10.5 Configuring Pod Storage with PV and PVC](#105-configuring-pod-storage-with-pv-and-pvc)
      - [Using PVCs for Pods](#using-pvcs-for-pods)
      - [Demo: Using PVCs for Pods](#demo-using-pvcs-for-pods)
    - [10.6 Understanding StorageClass](#106-understanding-storageclass)
      - [Understanding StorageClass](#understanding-storageclass)
      - [Using StorageClass as a Selector](#using-storageclass-as-a-selector)
      - [Demo: Explore StorageClass](#demo-explore-storageclass)
    - [Lesson 10 Lab Setting up Storage](#lesson-10-lab-setting-up-storage)
    - [Lesson 10 Lab Solution Setting up Storage](#lesson-10-lab-solution-setting-up-storage)

### 10.1 Understanding Kubernetes Storage Options

- Explaination

#### Understanding Storage Options

- Files stored in a container will only live as long as the container itself.
- Pod Volumes are used to allocate storage taht outlives a container and stays available during Pod lifetime.
- Pod Volumes can directly bind to a specific storage type.
- Pod Volumes can also use persistent Volume Claims (PVC), to decouple the Pod from the requested site-specific storage.

#### Understanding Persistent Volumes

- A Persistent Volumes defines access to external storage available in a specific cluster.
- A wide range of Persistent Volume types is available.
- Persistent Volume Claims (PVC) are used to connect to Persistent Volumes.
- When a PVC is created, it will search for an available PV that matches the storage request.
- If a perfect match does not exist, StorageClass can automatically allocate it.
- In this scenario, the Pod uses the PVC storage type.

### 10.2 Configuring Pod Volume Storage

#### Configuring Volumes

- To configure Pod local volumes, the volume is defined in **pod.spec.volumes**
- The volume points to a specific volume type.
  - For testing purpose **emptyDir** and **hostPath** are common.
  - A wide range of other storage types is also available.
- The volume is mounted through.
  - **pod.spec.containers.volumeMounts**

#### Demo: Using Pod Local Storage

```bash
kubectl explain pod.spec.volumes | less
cat morevolumes.yaml
kubectl create -f morevolumes.yaml
kubectl get pods morevol
kubectl describe pods morevol | less # verify there are two containers in the Pod.
kubectl exec -it morevol -c centos1 -- touch /centos1/test
kubectl exec -it morevol -c centos2 -- ls -l /centos2
```

### 10.3 Configuring PV Storage

#### Understanding Persistent Volumes

- A PV is an independent resource that connects to external storage.
  - It can point to all of the storage types.
  - Use PVC to connect to it.
  - They PVC talks to the available backend storage provider and dynamically uses volumes that storage type.
- The PVC will bind to a PV according to the available of the requested volume accessModes and capacity.

#### Demo: Defining Persistent Volumes

```bash
vim pv.yaml
kubectl create -f pv.yaml
kubectl get pv
describe pv pv-volume
```

### 10.4 Configuring PVCs

#### Understanding Persistent Volume Claims

- The VPV requests access to the Persistent Volume according to specific properties.
  - accessModes
  - availability of resources.
- To use the storage requested by a PVC, the Pod Volume type specification can use the name of the PVC.

#### Understanding PVC Binds

- After connecting to a PV, the PVC will show as bound.
- A PVC bind is exclusive: after successfully binding to a PV, the PV cannot be used by other PVC anymore.
- If a perfect match is not available, StorageClass may kick in and dynamically create a PV that does exactly match the request.

#### Demo: Using PVC

```bash
vim pvc.yaml
kubectl create -f pvc.yaml
kubectl get pvc
kubectl describe pvc pv-claim
kubectl get pv
kubectl describe pv pv-xxx
```

### 10.5 Configuring Pod Storage with PV and PVC

#### Using PVCs for Pods

- The purpose of configuring a Pod with PVC storage is to decouple site-specific information from a generic Pod specification.
- The pod.volume.spec is set to PersistentVolumeClaim, and it's up to the PVC to find availalbe PV storage.
- When a Pod specification is destributed with a PVC specification, it will always to available site-specific storage, without knowing anything about this site-specific storage.

#### Demo: Using PVCs for Pods

```bash
cat pvc-pod.yaml
kubectl create -f pvc-pod.yaml
kubectl get pvc,pv
kubectl get pv
kubectl describe pv pvc-xxx-yyy
kubectl exec nginx-pvc-pod -- touch /usr/share/nginx/html/testfile
minikube ssh
ls /tmp/hostpath-provisioner/default/nginx-pvc/
```

### 10.6 Understanding StorageClass

#### Understanding StorageClass

- Cluster environments typically have their own specific storage type.
- Kubernetes StorageClass allows for automatic provisioning of PVs when a PVC request for storage comes in.
- StorageClass must be backed by a storage provisioner, which takes care of the volume configuration.
- Kubernetes comes with some internal provisioners, alternatively external provisioners can be installed using Operators.
  - Typical lab environment storage is not backed by a Kubernetes provisioner.
  - Minikube does come with a provisioner.

#### Using StorageClass as a Selector

- A second use of StorageClass, is as a selector label.
- Normally, PVC binding to a PV is based on best match.
- While using StorageClass in PV as well as PVC, a PVC will only bind to a PV that has the same StorageClass setting.
- If such a PV is not available, the PVC will stay in a status of pending.

#### Demo: Explore StorageClass

```bash
kubectl create -f pvc.yaml
kubectl get pv
kubectl get storageclass
kubectl describe pv pvc-xxx-yyy
cat pv-pvc-pod.yaml
kubectl create -f pv-pvc-pod.yaml
kubectl get pv
```

### Lesson 10 Lab Setting up Storage

- Configure a PV that is using HostPath storage.
- Set up the storage accessMode such that only multiple Pods may access the storage at the same time.
- Configure a Pod that runs an httpd web server to mount the default web server documentroot /var/www/html on the HostPath storage.

### Lesson 10 Lab Solution Setting up Storage

- Open **https://kubernetes.io/docs**
- Search for **persistent volume** and hit **Enter** key
- Click **Configure a Pod to Use a PersistentVolume for Storage | Kubernetes**
- ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

```bash
cd labs/
vim lesson-10-lab.yaml
kubectl create -f lesson-10-lab.yaml
kubectl get pods,pvc,pv
kubectl describe pod task-pv-pod
kubectl delete all --all
```
