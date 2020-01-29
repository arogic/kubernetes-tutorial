Kubernetes:
- Prometheus (monitoring/logging) https://hub.helm.sh/charts/banzaicloud-stable/prometheus
- nginx proxy (loadbalancing)
- helm package manager
- role based access control (rbac) with Auth0
- docker registry
- version control (github, gitlab)
- artifactory (jfrog)
- configMap (?? - need to explore)

- Helm
- ArgoCD Solution
- Kubeform (Terraform) - https://github.com/kubeform/kubeform

Interesting:
https://github.com/virtual-kubelet/virtual-kubelet


export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-svc -o go-template='{{(index .spec.clusterIP)}}')
echo CLUSTER_IP=$CLUSTER_IP
curl $CLUSTER_IP:80

kubectl get svc
kubectl get pods

kubectl describe svc/webapp1-clusterip-targetport-svc

export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-targetport-svc -o go-template='{{(index .spec.clusterIP)}}')
echo CLUSTER_IP=$CLUSTER_IP
curl $CLUSTER_IP:8080


kubectl get pods -n kube-system


export LoadBalancerIP=$(kubectl get services/webapp1-loadbalancer-svc -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
echo LoadBalancerIP=$LoadBalancerIP
curl $LoadBalancerIP

# Use kubectl to find out what was deployed.
kubectl get all

# Helm
https://github.com/kubernetes/helm/releases

# Install Helm
curl -LO https://storage.googleapis.com/kubernetes-helm/helm-v2.8.2-linux-amd64.tar.gz
tar -xvf helm-v2.8.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/

# Once installed, initialise update the local cache to sync the latest available packages with the environment.
helm init
helm repo update

# Search For Chart (here we are searching for "redis")
helm search redis

# We can identify more information with the inspect command.
helm inspect stable/redis

# Deploy Redis
helm install stable/redis

To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default quieting-guppy-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis server:

1. Run a Redis pod that you can use as a client:

   kubectl run --namespace default quieting-guppy-redis-client --rm --tty -i --restart='Never' \
    --env REDIS_PASSWORD=$REDIS_PASSWORD \
   --image docker.io/bitnami/redis:5.0.7-debian-10-r0 -- bash

2. Connect using the Redis CLI:
   redis-cli -h quieting-guppy-redis-master -a $REDIS_PASSWORD
   redis-cli -h quieting-guppy-redis-slave -a $REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/quieting-guppy-redis-master 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD

# Helm will now launch the required pods. You can view all packages using
helm ls

# The helm could be provided with a more friendly name, such as:
helm install --name my-release stable/redis

Output:
master $ helm install --name my-release stable/redis
NAME:   my-release
LAST DEPLOYED: Fri Jan 24 09:00:49 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME              TYPE    DATA  AGE
my-release-redis  Opaque  1     0s

==> v1/ConfigMap
NAME                     DATA  AGE
my-release-redis         3     0s
my-release-redis-health  6     0s

==> v1/Service
NAME                       TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
my-release-redis-headless  ClusterIP  None           <none>       6379/TCP  0s
my-release-redis-master    ClusterIP  10.98.241.243  <none>       6379/TCP  0s
my-release-redis-slave     ClusterIP  10.102.75.79   <none>       6379/TCP  0s

==> v1/StatefulSet
NAME                     DESIRED  CURRENT  AGE
my-release-redis-master  1        1        0s
my-release-redis-slave   2        1        0s

==> v1/Pod(related)
NAME                       READY  STATUS   RESTARTS  AGE
my-release-redis-master-0  0/1    Pending  0         0s
my-release-redis-slave-0   0/1    Pending  0         0s


NOTES:
** Please be patient while the chart is being deployed **
Redis can be accessed via port 6379 on the following DNS names from within your cluster:

my-release-redis-master.default.svc.cluster.local for read/write operations
my-release-redis-slave.default.svc.cluster.local for read-only operations


To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default my-release-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis server:

1. Run a Redis pod that you can use as a client:

   kubectl run --namespace default my-release-redis-client --rm --tty -i --restart='Never' \
    --env REDIS_PASSWORD=$REDIS_PASSWORD \
   --image docker.io/bitnami/redis:5.0.7-debian-10-r0 -- bash

2. Connect using the Redis CLI:
   redis-cli -h my-release-redis-master -a $REDIS_PASSWORD
   redis-cli -h my-release-redis-slave -a $REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/my-release-redis-master 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD

# check the status of the release
helm ls --all my-release

# Delete the deployment my-release
helm del --purge my-release

# Use helm rollback to roll back to an older version of a release with ease.
