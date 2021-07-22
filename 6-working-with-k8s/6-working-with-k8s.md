# Kubernetes - Simple App

Kubernetes, k8s, or kube, is an open source platform that automates container operations. It eliminates most of the existing manual processes, which involve the deploying, scaling, and managing of containerized applications. With Kubernetes, you can cluster groups of hosts running containers together. Kubernetes helps you manage those clusters. These clusters can span the public, private, and hybrid clouds similarly to how Docker Swarm does it.

The state of the entities in the system at any given point of time is represented by Kubernetes Objects. Kubernetes Objects also act as an additional layer of abstraction over the traditional container interface. You can now directly interact with instances of Kubernetes Objects instead of interacting with containers. The basic Kubernetes objects are as follows:

* Pod - the smallest deployable unit on a Node. It’s a group of containers which must run together. (Often, though not necessarily, a Pod contains one container);
* Service - defines a logical set of Pods and related policies used to access them;
* Volume - a directory accessible to all containers running in a Pod;
* Namespaces - virtual clusters backed by the physical cluster.

There are a number of Controllers provided by Kubernetes. These Controllers are built upon the basic Kubernetes Objects and provide additional features. The Kubernetes Controllers include:

* ReplicaSet - a specified number of Pod replicas are running at any given time;
* Deployment - changes the current state to the desired state;
* StatefulSet - ensures control over the deployment ordering and access to volumes, etc.;
* DaemonSet - runs a copy of a Pod on all the nodes of a cluster or on specified nodes.
* Job - performs task and exit after successfully completing their work or after a given period of time.

On Kubernetes, containers are not run directly, but through ReplicaSet managed by a Deployment. Below is an example of a .yml file describing a Deployment. A ReplicaSet will ensure that two replicas of a Pod using Nginx are running.

As we will see below, in order to create a Deployment we need to use the kubectl command line tool.
To define a whole micro-services application in Kubernetes we need to create a Deployment file for each service. We can do this manually or we can use Kompose to help us in this task.

# Deployment of the application

Using kubectl, we will create all the components defined in the descriptor files. We indicate the files that are located in the current folder.

`	$ kubectl create -f .	`

The output should be something similar to the one below:

```bash
service/redis created
deployment.apps/redis created
service/db created
deployment.apps/db created
persistentvolumeclaim/postgres-pv-claim created
service/result created
deployment.apps/result created
service/vote created
deployment.apps/vote created
service/worker created
deployment.apps/worker created
```

The commands below show the services and deployments created. It will show the allocated cluster-ip (if one was allocated) and also the mapped external-ip (if one was allocated):

`	$ kubectl get services	`

The output should be something similar to this:

```bash
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
db           ClusterIP      None             <none>        5432/TCP         26s
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          92m
redis        ClusterIP      None             <none>        6379/TCP         26s
result       LoadBalancer   10.101.237.168   localhost     5001:30670/TCP   26s
vote         LoadBalancer   10.104.193.181   localhost     5000:32341/TCP   25s
worker       ClusterIP      None             <none>        <none>           25s
```

To get deployment details, we have to literally use the command below:

`	$ kubectl get deployment	`

The output should show us the service-names, the number of desired replicas, the current amount of replicas, how many are up-to-date with the configuration, and how many are responding:

```bash
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
db           1         1         1            1           3m
redis        1         1         1            1           3m
result       1         1         1            1           3m
visualizer   1         1         1            1           3m
vote         1         1         1            1           3m
worker       1         1         1            1           3m
```

To get the rollout status, we use the same get type command:

`	$ kubectl get rs	`

The output should be very similar to the deployment, as it details the number of pods currently running:

```bash
NAME                DESIRED   CURRENT   READY   AGE
db-7c894b85ff       1         1         1       13m
redis-86b47f8678    1         1         1       13m
result-59457678cf   1         1         1       13m
vote-7f5749bf95     2         2         2       13m
worker-76ff58f548   1         1         1       13m
```

To see the labels automatically generated for each Pod, we can run:

`	$ kubectl get pods --show-labels	`

The output is similar to:

```bash
NAME                      READY   STATUS    RESTARTS   AGE   LABELS
db-7c894b85ff-fkfhx       1/1     Running   0          13m   app=db,pod-template-hash=7c894b85ff
redis-86b47f8678-jsk57    1/1     Running   0          13m   app=redis,pod-template-hash=86b47f8678
result-59457678cf-lkg47   1/1     Running   0          13m   app=result,pod-template-hash=59457678cf
vote-7f5749bf95-8vmrg     1/1     Running   0          13m   app=vote,pod-template-hash=7f5749bf95
vote-7f5749bf95-jt9db     1/1     Running   0          13m   app=vote,pod-template-hash=7f5749bf95
worker-76ff58f548-5bd5r   1/1     Running   3          13m   app=worker,pod-template-hash=76ff58f548
```

Lastly, to get a complete view of your current deployment, use the command:

`	$ kubectl describe deployments	`

This will create a rather lengthy detailed view of all services included in the deployment. You should have multiple sections similar to the one below:

```bash
Name:                   db
Namespace:              default
CreationTimestamp:      Tue, 28 Jul 2020 21:55:06 +0300
Labels:                 app=db
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=db
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=db
  Containers:
   db:
    Image:      postgres:9.4
    Port:       5432/TCP
    Host Port:  0/TCP
    Environment:
      PGDATA:             /var/lib/postgresql/data/pgdata
      POSTGRES_USER:      postgres
      POSTGRES_PASSWORD:  postgres
    Mounts:
      /var/lib/postgresql/data from db-data (rw)
  Volumes:
   db-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  postgres-pv-claim
    ReadOnly:   false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   db-7c894b85ff (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  16m   deployment-controller  Scaled up replica set db-7c894b85ff to 1
```

# Scaling a service

Kubernetes is a bit more complex in terms of its scaling potential compared to Docker Swarm. It allows for both manual scaling, but also has a rather complex autoscaling feature. Let's test the basic scaling by running the following command:

`	$ kubectl scale deployment.v1.apps/vote --replicas=5	`

The output should only inform us the deployment was scaled:

```bash
deployment.apps/vote scaled
```
To see if the replicas have been updated, we can use the command:

`	$ kubectl get deployments	`

and we should actually see something similar to what we have below (please note the number of vote replicas):

```bash
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
db       1/1     1            1           27m
redis    1/1     1            1           27m
result   1/1     1            1           27m
vote     5/5     5            5           27m
worker   1/1     1            1           27m
```

Assuming horizontal Pod autoscaling is enabled, you can also setup an autoscaler for your Deployment and choose the minimum and maximum number of Pods you want to run based on the CPU utilization of your existing Pods.

`	$ kubectl autoscale deployment.v1.apps/vote --min=2 --max=5 --cpu-percent=80	`

The output again informs us of the autoscaling system being enabled:

```bash
horizontalpodautoscaler.autoscaling/vote autoscaled
```

# Pausing & resuming a deployment

You can pause a Deployment before triggering one or more updates and then resume it. This allows you to apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.

For example, with a Deployment that was just created:

`	$ kubectl get deploy	`

The output is similar to this:

NAME     READY   UP-TO-DATE   AVAILABLE   AGE
db       1/1     1            1           33m
redis    1/1     1            1           33m
result   1/1     1            1           33m
vote     2/2     2            2           33m
worker   1/1     1            1           33m

Get the rollout status:

`	$ kubectl get rs	`
The output is similar to this:

```bash
NAME                DESIRED   CURRENT   READY   AGE
db-7c894b85ff       1         1         1       34m
redis-86b47f8678    1         1         1       34m
result-59457678cf   1         1         1       34m
vote-7f5749bf95     2         2         2       34m
worker-76ff58f548   1         1         1       34m
```

Pause the deployment by running the following command:

`	$ kubectl rollout pause deployment.v1.apps/vote	`

The output is similar to this:

```
deployment.apps/vote paused
```

You can make as many updates as you wish, for example, update the resources that will be used:

`	$ kubectl set resources deployment.apps/vote -c=vote --limits=cpu=200m,memory=512Mi	`

The output is similar to this:

```bash
deployment.apps/vote resource requirements updated
```

The initial state of the Deployment prior to pausing it will continue its function, but new updates to the Deployment will not have any effect as long as the Deployment is paused. Eventually, we can resume the Deployment and observe a new ReplicaSet coming up with all the new updates:

`	$ kubectl rollout resume deployment.v1.apps/vote	`

The output is similar to this:

```
deployment.apps/vote resumed
```

# Deleting the deployment

If a deployment needs to be removed permanently, it will not be possible to do so by removing services or pods. As the replica-set dictates a specific amount of expected instances, if it finds that one was deleted, it will re-create it on the spot. The fastest way to do so, is to delete the Deployment.

For that we can first use:

`	$ kubectl get deployments	`

Then we can delete a specific deployment by using the command:

`	$ kubectl delete deployment worker --namespace=default	`

and this will remove the worker deployment. If we want to clear all of our deployments created from the .yml file, we can use the command:

`	$ kubectl delete -f voteapp-stack.yml	`

This should have the following output:

```
service "redis" deleted
deployment.apps "redis" deleted
service "db" deleted
deployment.apps "db" deleted
persistentvolumeclaim "postgres-pv-claim" deleted
service "result" deleted
deployment.apps "result" deleted
service "vote" deleted
deployment.apps "vote" deleted
service "worker" deleted
Error from server (NotFound): error when deleting "voteapp-stack.yml": deployments.apps "worker" not found
```