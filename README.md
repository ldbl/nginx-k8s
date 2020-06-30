
### Requirments 

#### VirtualBox or KVM 
https://www.virtualbox.org/

#### Minikube
https://kubernetes.io/docs/setup/learning-environment/minikube/

#### Kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl/

#### Helm3 
https://github.com/helm/helm/releases/tag/v3.2.2

---
**We can control the VM types that minikube will spawn but we use default one**

### Start minikube 
```
minikube start
```

### Get minkube status
```
minikube status 
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### Add nginx helm repo
```
helm3 repo add bitnami https://charts.bitnami.com/bitnami

```

### Install nginx
```
helm3 upgrade --install my-nginx bitnami/nginx  -f my-nginx-values.yaml
```

### set values for the deployment in the values file. 
We set nodeport to access the nginx on VM IP  otherwise we need to use kubectl port-forward 

```
cat my-nginx-values.yaml 
service:
  type: NodePort

replicaCount: 1
```

it's not necessary to add all posible values here because they are inheritd by the main values file : 
https://github.com/bitnami/charts/blob/master/bitnami/nginx/values.yaml
In this file we only overwrite them .



After sucessfull instalation we can check the deployments :


```
kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx   1/1     1            1           3m54s
```

We can aslo check the service node port to see where to access the nginx.
```
kubectl get svc my-nginx
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
my-nginx   NodePort   10.109.140.81   <none>        80:30967/TCP,443:31675/TCP   4m42s
```
We can also check the minikube IP.
```
minikube ip
192.168.39.162

```

Nginx must be accessible on 192.168.39.162:30967


Let's check how many pods we have (replicas )

```
kubectl get pods 
NAME                       READY   STATUS    RESTARTS   AGE
my-nginx-555d6dff7-4f4tq   1/1     Running   0          7m29s
```

### Run some test on 1 replica 
```
hey http://192.168.39.162:30967

Summary:
  Total:	0.0308 secs
  Slowest:	0.0235 secs
  Fastest:	0.0002 secs
  Average:	0.0054 secs
  Requests/sec:	6494.0216
  

Response time histogram:
  0.000 [1]	|■
  0.003 [37]	|■■■■■■■■■■■■■■■■■■■■■■■■■
  0.005 [60]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.007 [58]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.010 [20]	|■■■■■■■■■■■■■
  0.012 [18]	|■■■■■■■■■■■■
  0.014 [2]	|■
  0.016 [1]	|■
  0.019 [1]	|■
  0.021 [0]	|
  0.023 [2]	|■


Latency distribution:
  10% in 0.0016 secs
  25% in 0.0033 secs
  50% in 0.0050 secs
  75% in 0.0065 secs
  90% in 0.0100 secs
  95% in 0.0113 secs
  99% in 0.0222 secs

Details (average, fastest, slowest):
  DNS+dialup:	0.0003 secs, 0.0002 secs, 0.0235 secs
  DNS-lookup:	0.0000 secs, 0.0000 secs, 0.0000 secs
  req write:	0.0001 secs, 0.0000 secs, 0.0019 secs
  resp wait:	0.0043 secs, 0.0002 secs, 0.0234 secs
  resp read:	0.0007 secs, 0.0000 secs, 0.0068 secs

Status code distribution:
  [200]	200 responses
```

Let's increase the replicas 
```
kubectl scale --replicas=3 deployments/my-nginx
```

Let's confirm we have 3 pods 
```
kubectl get pods 
NAME                       READY   STATUS    RESTARTS   AGE
my-nginx-555d6dff7-4f4tq   1/1     Running   0          12m
my-nginx-555d6dff7-6vm5h   1/1     Running   0          39s
my-nginx-555d6dff7-tw6zw   1/1     Running   0          39s
`

Let;s run some tests 
`
hey http://192.168.39.162:30967

Summary:
  Total:	0.0358 secs
  Slowest:	0.0280 secs
  Fastest:	0.0004 secs
  Average:	0.0064 secs
  Requests/sec:	5583.7100
  

Response time histogram:
  0.000 [1]	|■
  0.003 [73]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.006 [67]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.009 [26]	|■■■■■■■■■■■■■■
  0.011 [6]	|■■■
  0.014 [2]	|■
  0.017 [3]	|■■
  0.020 [4]	|■■
  0.023 [4]	|■■
  0.025 [10]	|■■■■■
  0.028 [4]	|■■


Latency distribution:
  10% in 0.0019 secs
  25% in 0.0026 secs
  50% in 0.0042 secs
  75% in 0.0070 secs
  90% in 0.0190 secs
  95% in 0.0241 secs
  99% in 0.0277 secs

Details (average, fastest, slowest):
  DNS+dialup:	0.0004 secs, 0.0004 secs, 0.0280 secs
  DNS-lookup:	0.0000 secs, 0.0000 secs, 0.0000 secs
  req write:	0.0001 secs, 0.0000 secs, 0.0031 secs
  resp wait:	0.0056 secs, 0.0004 secs, 0.0263 secs
  resp read:	0.0003 secs, 0.0000 secs, 0.0027 secs

Status code distribution:
  [200]	200 responses

```
Look like with 3 replicas most of the respones are bellow 0.011 .
