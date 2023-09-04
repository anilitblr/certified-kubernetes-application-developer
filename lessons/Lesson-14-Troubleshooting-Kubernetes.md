# Lesson 14 - Troubleshooting Kubernetes

- [Lesson 14 - Troubleshooting Kubernetes](#lesson-14---troubleshooting-kubernetes)
    - [14.1 Determining a Troubleshooting Strategy](#141-determining-a-troubleshooting-strategy)
      - [Understanding Pod Startup](#understanding-pod-startup)
    - [14.2 Analyzing Failing Applications](#142-analyzing-failing-applications)
      - [Understanding Pod States](#understanding-pod-states)
      - [Troubleshooting Failed Applications](#troubleshooting-failed-applications)
      - [Demo: Investigating Failing Applications: busybox](#demo-investigating-failing-applications-busybox)
      - [Demo: Investigating Failing Applications: mariadb](#demo-investigating-failing-applications-mariadb)
    - [14.3 Analyzing Pod Access Problems](#143-analyzing-pod-access-problems)
      - [Understanding Services](#understanding-services)
      - [Understanding Ingress](#understanding-ingress)
      - [Understanding NetworkPolicy](#understanding-networkpolicy)
      - [Understanding the Network add-on](#understanding-the-network-add-on)
      - [Demo: Troubleshooting Services](#demo-troubleshooting-services)
    - [14.4 Monitoring Cluster Event Logs](#144-monitoring-cluster-event-logs)
      - [Monitoring Cluster Events](#monitoring-cluster-events)
    - [14.5 Troubleshooting Authentication Problems](#145-troubleshooting-authentication-problems)
      - [Troubleshooting Authentication Problems](#troubleshooting-authentication-problems)
      - [Demo: Recovering .kube/config from Minikube](#demo-recovering-kubeconfig-from-minikube)
    - [14.6 Using Probes](#146-using-probes)
      - [Understanding Probes](#understanding-probes)
      - [Understanding Probe Type](#understanding-probe-type)
      - [Demo: Using Probs](#demo-using-probs)
    - [Lesson 14 Lab Troubleshooting Kubernetes](#lesson-14-lab-troubleshooting-kubernetes)
    - [Lesson 14 Lab Solution Troubleshooting Kubernetes](#lesson-14-lab-solution-troubleshooting-kubernetes)

### 14.1 Determining a Troubleshooting Strategy

#### Understanding Pod Startup

- After using **kubectl create** **kubectl run**, the requested resources are written to the etcd database.
- Next, the scheduler will look up a node eligible to run the application.
- After finding an eligible node, the Pod image is fetched.
- Next, the Pod container is started and runs its entrypoint application.
- Based on the success or failure of the entrypoint application, the Pod RestartPolicy is applied to determine further action.

### 14.2 Analyzing Failing Applications

#### Understanding Pod States

- In its lifetime, a Pod will go through different states.
  - **Pending**: the Pod has been validated by the API server and an entry has been created in the etcd, but some prerequisite conditions have not been met.
  - **Running**: the Pod currently is successfully running.
  - **Completed**: the Pod has completed its work.
  - **Failed**: the Pod has finished but something has gone wrong.
  - **CrashLoopBackOff**: the Pod has failed, and the cluster has restarted it.
  - **Unknown**: the Pod status could not be obtained.
- Current Pod status can be observed using **kubectl get pods**

#### Troubleshooting Failed Applications

- Use **kubectl describe** to investigate the application state.
- First, look at the events, next have a look at the application state.
- Check the last state, and in particular the application exit code.
- If the exit code is 0(zero), the application has completed successfully and no further investigation is needed.
- If the exit code is other than 0(zero), there is a problem with the entrypoint application and you need to use **kubectl logs** to investigate the application logs.

#### Demo: Investigating Failing Applications: busybox

```bash
kubectl create deploy failure1 --image=busybox
kubectl get pods
kubectl describe pods failure1-xxx-yyy
kubectl delete deploy failure1
kubectl create deploy failure1 --image=busybox -- sleep 3600
kubectl get pods
kubectl describe pods failure1-xxx-yyy
```

#### Demo: Investigating Failing Applications: mariadb

```bash
kubectl create deploy failure2 --image=mariadb
kubectl get pods
kubectl describe pods failure2-xxx-yyy
kubectl logs failure2-xxx-yyy
kubectl set env -h | less 
kubectl set env deployment/failure2 MYSQL_ROOT_PASSWORD=password
```

### 14.3 Analyzing Pod Access Problems

#### Understanding Services

- To access an application, a Service is used to load balance between available Pods.
- To connect to the backend Pods, the Service is using a Selector label that matches the Pod label.
- If services don't work as expected, you should check these labels first.
- Also use **kubectl get endpoints** to check Service and corresponding Pod endpoints.

#### Understanding Ingress

- Ingress uses a Service to forward traffic to Pods.
- To do so, the Pod labels are used.
- Ingress also requires an Ingress controller to be configured and running.
- Troubleshooting the Ingress controller is beyond the scope of CKAD.

#### Understanding NetworkPolicy

- NetworkPolicy can be applied to Pods, NameSpaces and IP address ranges to restrict traffic in both directions.
- Use **kubectl get netpol -A** to check for the existence of any NetworkPolicy.

#### Understanding the Network add-on

- Different network add-ons are available for Kubernetes.
- Changing a network add-on may break or fix network related problems.
- The flannel add-on for instance doesn't support NetworkPolicy
- Consider using the Calico add-on for rich feature support.

#### Demo: Troubleshooting Services

```bash
kubectl create deploy trouble --image=nginx
kubectl expose deploy trouble --port=80 --type=NodePort
kubectl get endpoints
kubectl get svc # make note of the nodeport.
curl $(minikube ip):NODEPORT # replace NODEPORT with actual port.
kubectl edit svc trouble # change the selector label.
kubectl get endpoints
kubectl get svc # make note of the nodeport.
curl $(minikube ip):NODEPORT # should now fail.
```

### 14.4 Monitoring Cluster Event Logs

#### Monitoring Cluster Events

- **kubectl get events** provides an overview of cluster-wide events.
- **kubectl get events -o wide** provides an overview of cluster-wide events with more details.
- Use **kubectl get events** if it's unclear to which resource an error conditionis related.
- If you know which resource is involved, better use **kubectl describe** on that resource.
```bash
kubectl get events
kubectl get events | less
kubectl get events | grep Warning
kubectl get events -o wide
kubectl describe nodes minikube
```

### 14.5 Troubleshooting Authentication Problems

#### Troubleshooting Authentication Problems

- RBAC configurations are not a part of CKAD.
- Access to the cluster is configured through a **~/.kube/config** file.
- This file is copied from the control node is the cluster, where it is stored as /etc/kubernetes/admin.conf
- Use **kubectl config view** to check contents of this file.
- For additional authorization based problems, use **kubectl auth can-i**: **kubectl auth can-i create pods** 
- Troubleshooting RBAC based access is covered in Certified Kubernetes Security (CKS).

#### Demo: Recovering .kube/config from Minikube

```bash
cd ~/.kube
mv config .config
kubectl get pods
minikube ssh
sudo -i
cp /etc/kubernetes/admin.conf /tmp
chmod 644 /tmp/admin.conf
exit;
exit
scp -i $(minikube ssh-key) docker@$(minikube ip):/tmp/admin.conf ~/.kube/config
kubectl get pods
```

### 14.6 Using Probes

#### Understanding Probes

- Probes can be used to test access to Pods.
- They are a part of the Pod specification.
- A readinessProbe is used to make sure a Pod is not published as available until the readinessProbe has been able to access it.
- The livenessProbe is usd to continue checking the availability of a Pod.
- The startupProbe was introduced for legacy applications that require additional startup time on first initialization.

#### Understanding Probe Type

- The probe itself is a simple test, which is often a command.
- The following probe types are defined in pods.spec.container
  - **exec**: a command is executed and returns a zero exit value.
  - **httpGet**: an HTTP request returns a response code between 200 and 399
  - **tcpSocket**: connectivity to a TCP socket (available port) is successful.
- The Kubernetes API provides 3 endpoints that indicate the current status of the API server.
  - /healtz
  - /livez
  - /readyz
- These endpoints can be used by different porbs.

#### Demo: Using Probs

```bash
kubectl create -f busybox-ready.yaml
kubectl get pods # note the READY state, which is set to 0/1, which means that the Pod has successfully started, but is not considered ready.
kubectl describe pods busybox-ready
kubectl edit pods busybox-ready # Change /tmp/nothing to /etc/hosts. Notice this is not allowed.
kubectl exec -it busybox-ready -- touch /tmp/nothing
kubectl get pods # At this point we have a Pod that is started.
```
- nginx-probe

```bash
cd labs/
vim nginx-probes.yaml
kubectl create -f nginx-probes.yaml
kubectl get pods
```

### Lesson 14 Lab Troubleshooting Kubernetes

- Create a Pod that is running the Busybox container with the sleep 3600 command. Configure a Probe that checks for the existence of the file /etc/hosts on that Pod.

### Lesson 14 Lab Solution Troubleshooting Kubernetes

- vim lesson-14-lab.yaml 
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - sleep 3600
    livenessProbe:
      exec:
        command:
        - cat
        - /etc/hosts
      initialDelaySeconds: 5
      periodSeconds: 5
```

```bash
vim lesson-14-lab.yaml 
kubectl create -f lesson-14-lab.yaml 
kubectl get all
```
