# Kubernetes Examples

This repo is a selection of example Kubernetes configurations. I'll assume you have a Kubernetes cluster already running on your local machine or one of the managed cloud providers. I'll also assume you have the `kubectl` command installed locally and configured to connect to your cluster.


## Simple web service

In this example we have a container running a webserver on port 80. We use a [Deployment Controller](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) which will ensure there are always 2 pods (and therefore 2 cointainers) running via a [Replica Set Controller](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). The Replica Set does not need to be configured manually as the Deployment Controller manages this. The other thing we need to configure is a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) with a [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer).

This is our Kubernetes cluster before we run anything on it. It has just the default `kubernetes` service running.

```
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.245.0.1   <none>        443/TCP   4h26m
```

We apply the two config files in the `simple-web-service` directory. 

```
$ kubectl apply -f simple-web-service
deployment.apps/simple-deployment created
service/simple-service created
```

The cluster will start to initialise workloads and services. Running this at first shows there is a Deployment `deployment.apps/simple-deployment` with 0/2 pods ready. The Replica Set is also visible and show the same info. In the Pods section you can see a line for each pod in the Replica Set with the status `ContainerCreating`. There is intended to be one container for each pod (0/1).

```
$ kubectl get all
NAME                                    READY   STATUS              RESTARTS   AGE
pod/simple-deployment-6566784d5-2zkw8   0/1     ContainerCreating   0          3s
pod/simple-deployment-6566784d5-cxs2v   0/1     ContainerCreating   0          3s

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes       ClusterIP      10.245.0.1       <none>        443/TCP        4h29m
service/simple-service   LoadBalancer   10.245.233.203   <pending>     80:32487/TCP   3s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-deployment   0/2     2            0           3s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-deployment-6566784d5   2         2         0       3s
```

You can run `watch kubectl` to see changes to the cluster as they happen. After a little while you should see something like this. 2/2 pods are running and an external IP address has been aquired on the Load Balancer. You should now be able to go to that IP address in your browser and see a webpage.


```
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/simple-deployment-6566784d5-2zkw8   1/1     Running   0          3m42s
pod/simple-deployment-6566784d5-cxs2v   1/1     Running   0          3m42s

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
service/kubernetes       ClusterIP      10.245.0.1       <none>           443/TCP        4h33m
service/simple-service   LoadBalancer   10.245.233.203   188.166.139.55   80:32487/TCP   3m42s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-deployment   2/2     2            2           3m42s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-deployment-6566784d5   2         2         2       3m42s
```