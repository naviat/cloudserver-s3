**In Dev environnment with k8s in cluster minikube**

*In production, we expect a cluster n nodes (3 nodes minimum) installed with kubeadm or provided by a cloud provider
we also expect that data will be persistent.
That you will use the multiple backends capabilities of Zenko CloudServer, and that you will have a custom endpoint for your local storage, and custom credentials for your local storage:*

*Many object k8s will be created, in logical order :*

**ConfigMap that stores ENV variables**

```shell
$ kubectl apply -f configmap-s3server.yml

$ kubectl get cm -o wide

  NAME            DATA   AGE
  config-map-s3   3      1d
```

**Persistent Volumes for storage local or on any other types of storage suported by kubernetes (Volume Plugin)**

```shell
$ kubectl apply -f pv-s3server.yml
$ kubectl apply -f pv2-s3server.yml

$ kubectl get pv -o wide

  NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   REASON   AGE
  s3-pv    1000Mi     RWO,ROX        Recycle          Bound    default/s3server-claim                            3d
```

**PersistentVolumeClaim for binding the PersistentVolume and then use refered this in pods**

```shell
$ kubectl apply -f pvc-s3server.yml
$ kubectl apply -f pvc2-s3server.yml

$ kubectl get pvc -o wide

  NAME              STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  s3server-claim    Bound    s3-pv    1000Mi     RWO,ROX                       3d
```

*At the hight level, the Deployment which including replicas, pods, labels, environment config, strategy of rolling update, claim for pvc...*

```shell
$ kubectl apply -f deploy-s3server-pv.yml --record

$ kubectl get deploy -o wide

  NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                    SELECTOR
  s3server          3         3         3            3           1d    s3server     scality/s3server:latest   app=s3server

$ kubectl get rs -l app=s3server -o wide

  NAME                  DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                    SELECTOR
  s3server-6ffc956c55   3         3         3       30d   s3server     scality/s3server:latest   app=s3server,pod-template-hash=2997512711

$ kubectl get pods -l app=s3server -o wide

  NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE
  s3server-6ffc956c55-hp5cc   1/1     Running   1          1d    172.17.0.11   minikube
  s3server-6ffc956c55-llld7   1/1     Running   1          1d    172.17.0.10   minikube
  s3server-6ffc956c55-xhxbm   1/1     Running   1          1d    172.17.0.9    minikube

```

*And finally the Service "NodeType" bring a stable and reliable network, DNS, load-balancing and expose your app outside of the kubernetes cluster, Cluster IP and Port (in our case : 10.109.72.126:30042 over TCP)*

>DNS-Based Service Recovery requires the DNS cluster--add-on,
The DNS add-on implements a POD-based DNS service in the cluster and configures all kubelets (nodes) to use it for DNS
ie: minikube addons list

```shell
$ kubectl apply -f svc-nodeport.yml

$ kubectl get svc -l app=s3server -o wide


  NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
  s3server-svc   NodePort   10.109.72.126   <none>        8000:30042/TCP   5d    app=s3server
```
