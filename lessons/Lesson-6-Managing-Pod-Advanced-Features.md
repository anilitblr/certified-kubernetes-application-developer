# Lesson 6 - Managing Pod Advanced Features

- [Lesson 6 - Managing Pod Advanced Features](#lesson-6---managing-pod-advanced-features)
    - [6.1 Exploring Pod State with kubectl describe](#61-exploring-pod-state-with-kubectl-describe)
      - [Exploring Pod Configuration](#exploring-pod-configuration)
      - [Connecting to a Pod for Further Inspection](#connecting-to-a-pod-for-further-inspection)
    - [6.2 Exploring Pod Logs for Application Troubleshooting](#62-exploring-pod-logs-for-application-troubleshooting)
      - [Using the Pod Logs](#using-the-pod-logs)
      - [Troubleshooting Failing Applications](#troubleshooting-failing-applications)
    - [6.3 Using Port Forwarding to Access Pods](#63-using-port-forwarding-to-access-pods)
      - [6.3 Using Port Forwarding to Access Pods](#63-using-port-forwarding-to-access-pods-1)
      - [Demo: Using Port Forwarding](#demo-using-port-forwarding)
    - [6.4 Understanding and Configuring SecurityContext](#64-understanding-and-configuring-securitycontext)
      - [Understanding Configuring SecurityContext](#understanding-configuring-securitycontext)
      - [Using SecurityContext](#using-securitycontext)
      - [Demo: Using SecurityContext](#demo-using-securitycontext)
    - [6.5 Managing Jobs](#65-managing-jobs)
      - [Managing Jobs](#managing-jobs)
      - [Understanding Job Types](#understanding-job-types)
      - [Demo: Using Jobs](#demo-using-jobs)
    - [6.6 Managing CronJobs](#66-managing-cronjobs)
      - [Managing CronJobs](#managing-cronjobs)
      - [Demo: Managing CronJobs](#demo-managing-cronjobs)
    - [6.7 Managing Resource Limitations and Quota](#67-managing-resource-limitations-and-quota)
      - [Understanding Resource Limitations](#understanding-resource-limitations)
      - [Understanding Quota](#understanding-quota)
      - [Demo: Running a Pod with Limitations](#demo-running-a-pod-with-limitations)
      - [Demo: Using Quota](#demo-using-quota)
    - [6.8 Cleaning up Resources](#68-cleaning-up-resources)
      - [Understanding the Need to Cleanup](#understanding-the-need-to-cleanup)
      - [Demo: Cleaning up Resources](#demo-cleaning-up-resources)
    - [Lesson 6 Lab Managing Pod Advanced Features](#lesson-6-lab-managing-pod-advanced-features)
    - [Lesson 6 Lab Solution Managing Pod Advanced Features](#lesson-6-lab-solution-managing-pod-advanced-features)

### 6.1 Exploring Pod State with kubectl describe

#### Exploring Pod Configuration

- When deploying a Pod, many parameters are set to a default value.
- Use **kubectl describe pod <podname-xxx>** to see all these parameters and their current setting as they currently are in the etcd.
- Use documentation at **https://kubernetes.io/docs** for more information about these settings.
- Tip! This documentation is available on the exam as well use it.

#### Connecting to a Pod for Further Inspection

Apart from exploring a Pod externally, you can also connect to it and run commands on the primary container in a Pod.
- **kubectl exec -it nginx-xxx -- sh**
- From here, run any command to investigate.
- Use **exit** to disconnect.
```bash
kubectl get pods
kubectl get pods mynginx -o json | less
kubectl get pods mynginx -o yaml | less
kubectl explain pods.spec.enableServiceLinks
kubectl describe pod mynginx
kubectl exec -it mynginx -- sh
```

### 6.2 Exploring Pod Logs for Application Troubleshooting

#### Using the Pod Logs

- The Pod entrypoint application does not connect to any STDOUT.
- Application output is sent to the Kubernetes cluster.
- Use **kubectl logs** to connect to this ouput.
- Using **kubectl logs** is helpful in troubleshooting 

#### Troubleshooting Failing Applications

- Start by using **kubectl get pods** and check the status of your application.
- While seeing anything odd, use **kubectl describe pod <podname>** to get more information.
- If the Pod main application generates a non-zero exit code, use **kubectl logs** to figure out what is going wrong.
```bash
kubectl run mydb --image=mariadb
kubectl get pods
kubectl describe pod mydb
kubectl get pods
kubectl logs mydb
kubectl delete pod mydb
kubectl run --help | less
kubeclt run mydb --image=mariadb --env MARIADB_ROOT_PASSWORD=password
kubectl get pods
kubectl logs mydb
```

### 6.3 Using Port Forwarding to Access Pods

#### 6.3 Using Port Forwarding to Access Pods

- Pods can be accessed in multiple ways.
- A very simple way is by using port forwarding to expose a Pod port on the kubectl host that forwards to teh Pod.
  - kubectl port-forward fwnginx 8080:80
- Port forwarding is useful for testing Pod accessibility on a specific cluser node, not to expose it it external users.
- Regular user access to applications is provided through services and ingress (covered later)

#### Demo: Using Port Forwarding

```bash
kubectl get pods -o wide
kubectl run fwnginx --image=nginx
kubectl port-forward fwnginx 8080:80 &
curl http://localhost:8080
```

### 6.4 Understanding and Configuring SecurityContext

#### Understanding Configuring SecurityContext

A SecurityContext defines privilege and access control settings for a Pod or container, and includes the following.
- Discretionary Access Control which is about permissions used to access an object.
- Security Enhanced Linux, where security labels can be applied.
- Running as privileged or unprivileged user.
- Using Linux capabilities.
- AppArmor, which is an alternative to SELinux.
- AllowPrivilegeEscalation which controls if a process can gain more privilege than its parent process.
- Use **kubectl explain** for a complete overview.

#### Using SecurityContext

- Notice that SecurityContext can be applied to Pods as well as containers.
- When SecurityContext prevents a Pod from running successfully, use **kubectl describe** to get additional information from the events.
- In some cases, you will need to analyze in more depth and use **kubectl logs** on teh failing Pod as well.

#### Demo: Using SecurityContext

```bash
cd labs/
vim securitycontextdemo2.yaml
kubectl apply -f securitycontextdemo2.yaml
kubectl get pod
kubectl exec -it security-context-demo -- sh
ps
cd /data; ls -l
cd demo; echo hello > testfile
ls -l
id
```
```bash
vi securitycontextdemo.yaml
kubectl apply -f securitycontextdemo.yaml
kubectl get pod
kubectl describe pod nginxsecure
```

### 6.5 Managing Jobs

#### Managing Jobs

- Pods normally are created to run forever.
- To create a Pod that runs up to completion, use Jobs instead.
- Jobs are useful for one-shot tasks, like backup, calculation, batch processing and more.
- Use **spec.ttlSecondsAfterFinished** to clean up completed Jobs automatically.

#### Understanding Job Types

3 different Job types can be started, which is specified by the **completions** and **parallelism** 
parameters.
- Non-parallel Jobs: one Pod is started, unless the Pod fails.
  - **completions=1**
  - **parallelism=1**
- Parallel Jobs with a fixed completion count: the Job is complete after successfully running as many times as specified in job.spec.completions
  - **completions=n**
  - **parallelism=m**
- Parallel Jobs with a work queqe: multiple Jobs are started, when one completes successfully, the Job is complete.
   - **completions=1**
  - **parallelism=n**

#### Demo: Using Jobs

```bash
kubectl create -h | less
kubectl create job onejob --image=busybox -- date
kubectl get jobs
kubectl get jobs,pods
kubectl get jobs -o yaml | less
kubectl get jobs -o yaml | grep restartPolicy
kubectl get jobs -o yaml | grep -B 5 restartPolicy
kubectl delete jobs onejob
kubectl get jobs
kubectl create job mynewjob --image=busybox --dry-run=client -o yaml -- sleep 5 > mynewjob.yaml
# Edit mynewjob.yaml and include the following in job.spec
    completions: 3
    ttlSecondsAfterFinished: 60
kubectl create -f mynewjob.yaml
kubectl get jobs,pods
```

### 6.6 Managing CronJobs

#### Managing CronJobs

- Jobs are used to run a task a specific number of times.
- CronJobs are used for fasks that need to run on a regular basis.
- When running a CronJob, a Job will be scheduled.
- This Job, on its trun, will start a Pod.
- To test a CronJob, use **kubectl create job myjob --from=cronjob/mycronjob**

#### Demo: Managing CronJobs

```bash
kubectl create cronjob -h | less
kubectl create cronjob runme --image=busybox --schedule="*/2 * * * *" -- echo greetings from your cluster
kubectl get pods
kubectl create job runme --from=cronjob/runme
kubectl get cronjobs,jobs,pods
kubectl logs runme-xxx-yyy
kubectl delete cronjob runme
```

### 6.7 Managing Resource Limitations and Quota

#### Understanding Resource Limitations

- Resource requestes and limits are set as an application property.
- By default, a Pod will use as much CPU  and memory as necessary to do its work.
- This can be managed by using Memory/CPU requests and limits in pod.spec.containers.resources
- A request is an initial request for resources.
- A limit defines the upper threshold of resources a Pod can use.
- Memory as well as CPU limits can be used.
- CPU limits are expressed in millicor eor millicpu, 1/1000 of a CPU core.
  - So, 500 millicore is 0.5 CPU
- When being scheduled, the kube-scheduler ensures that the node running the Pods has all requested resources available.
- If a Pod with resource limits cannon be scheduled, it will show a status of Pending.
- Use **kubectl set resources ...** to apply resource limits to running applicatins in deployments (covered later)

#### Understanding Quota

- Quota are restrictions that are applied to namespaces.
- If Quota are set on a namespace, applications started in the namespace must have resource requests and limits set.
- Use **kubectl create quota ... -n mynamespace** to apply quota.

#### Demo: Running a Pod with Limitations

```bash
cd labs/
vim frontend-resources.yaml
kubectl create -f frontend-resources.yaml
kubectl get pods
kubectl describe pod frontend
kubeclt delete -f frontend-resources.yaml
```

#### Demo: Using Quota

```bash
kubectl create ns restricted
kubectl create quota -h | less
kubectl create quota myquota -n restricted --hard=cpu=2,--memory=1G,pods=3
kubectl describe ns restricted
kubectl run pod restrictedpod --image=nginx -n resttricted # will fail.
kubectl create deploy restricteddeploy --image=nginx -n restricted
kubectl set resources deploy restriceddeploy --limits=cpu=200m,memory=2G -n restricted
kubectl describe -n restricted deploy restricteddeploy
kubectl get all -n restricted
kubectl describe replicasets.apps <replicaset-name> -n restricted
set resources deploy restriceddeploy --limits=cpu=200m,memory=128M --requests=cpu=100m,memory=64M -n restricted 
kubectl get all -n restricted
```

### 6.8 Cleaning up Resources

#### Understanding the Need to Cleanup

- Resources are not automatically cleaned up.
- Some resources have options for automatic cleanup if they're no longer used.
- Periodic manual cleanup may be required.
- If a Pod is managed by deployments, the deployment must be removed not the Pod.
- Try not to force resource to deletion, it may bring them in an unmanageable state.

#### Demo: Cleaning up Resources

```bash
kubeclt get pods -A
kubectl delete all
kubectl delete -h | less
kubectl delete all --all
kubectl delete all --all --force --grace-period=-1
kubectl delete all --all -A --force --grace-period=-1 # Don't do this
```

### Lesson 6 Lab Managing Pod Advanced Features

Create a manifest file that defies the secret-app applicatin, which should meet the following requirements.
- It runs in the namespace **secret**
- It uses the **busybox** image to run the command **sleep 3600**
- It should restart only if it fails.
- It should make an initial memory request of 64 MiB, with an upper threshold of 128 MiB.

### Lesson 6 Lab Solution Managing Pod Advanced Features

- ref: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

```bash
kubectl create ns secret --dry-run=client -o yaml > lesson-6-lab.yaml
kubectl run secret-app --image=busybox --dry-run=client -o yaml -- sleep 3600 >> lesson-6-lab.yaml
kubectl explain pod.spec | less
kubectl apploy -f lesson-6-lab.yaml
kubectl get pods -n secret
```

- vim lesson-6-lab.yaml

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: secret
spec: {}
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-app
  name: secret-app
  namespace: secret
spec:
  restartPolicy: OnFailure
  containers:
  - args:
    - sleep
    - "3600"
    image: busybox
    resources:
      requests:
        memory: "64Mi"
      limits:
        memory: "128Mi"
    name: secret-app
    resources: {}
  dnsPolicy: ClusterFirst
status: {}
```
