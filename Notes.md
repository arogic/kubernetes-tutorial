# Create Service Account User
kubectl create -f create-service-account.yaml

# Create Cluster Role Binding for Service Account
kubectl create -f create-cluster-role-binding.yaml

# Command to find the token to use for the service account
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

# Command clusterrolebinding
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

# Prereq for Kubernetes Cluster Setup
- Disable swap

# Kubernetes setup using kubeadm (https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
><><><><><><><><<<><><><><><><><><><><><><><><><><><

# Initializing your control-plane node (10.125.117.7 is the static Ip of the Master)
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.125.117.7

# To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# Alternatively, if you are the root user, you can run:
export KUBECONFIG=/etc/kubernetes/admin.conf

# Install Pod Network on the Master Node
# For flannel to work correctly, you must pass --pod-network-cidr=10.244.0.0/16 to kubeadm init.
# Set /proc/sys/net/bridge/bridge-nf-call-iptables to 1 by running sysctl net.bridge.bridge-nf-call-iptables=1
# to pass bridged IPv4 traffic to iptables’ chains. This is a requirement for some CNI plugins to work, for more information please see here.
# Make sure that your firewall rules allow UDP ports 8285 and 8472 traffic for all hosts participating in the overlay network. see here .
# Note that flannel works on amd64, arm, arm64, ppc64le and s390x under Linux.
# Windows (amd64) is claimed as supported in v0.11.0 but the usage is undocumented.
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

# Once a Pod network has been installed, you can confirm that it is working by checking that the CoreDNS Pod is Running in the output of
kubectl get pods --all-namespaces
# And once the CoreDNS Pod is up and running, you can continue by joining your nodes.
# If your network is not working or CoreDNS is not in the Running state, checkout our troubleshooting docs.

# Joining your nodes
# The nodes are where your workloads (containers and Pods, etc) run. To add new nodes to your cluster do the following for each machine:
# - SSH to the machine
# - Become root (e.g. sudo su -)
# - Run the command that was output by kubeadm init. For example:
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

# If you do not have the token, you can get it by running the following command on the control-plane node:
kubeadm token list

# Test a Deployment
kubectl run nginx --image=nginx

# What is the difference between ReplicationController and ReplicaSet??

# Scale
# Update value for replicas in the defintions yaml File
kubectl replace -f replicaset-definition.yaml
# or scale adhoc without updating the definition File
kubectl scale --replicas=6 -f replicaset-definition.yaml
# or run it with the type and name format
kubectl scale --replicas=6 replicaset myapp-replicaset

# Commands
kubectl create -f replicaset-definition.yaml
kubectl get replicaset
kubectl delete replicaset myapp-replicaset # Also deletes all underliying PODs
kubectl replace -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml


# Rollback Commands
kubectl rollout undo deployment/myapp-deployment

# kubectl run (deploys a pod, but creates a deployment)
kubectl run nginx --image=nginx
> deployment "nginx" created

# Summarize Commands
## Create
kubectl create -f deployment-definition.yaml
kubectl create -f deployment-definition.yaml --record # Record the change

## Get
kubectl get deployments

## Update
kubectl apply -f deployment-definition.yaml
kubectl set image deployment/myapp-deployment nginx=nginx:1.17.8

## Status
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment

## Rollback
kubectl rollout undo deployment/myapp-deployment

Kubernetes:
- Prometheus (monitoring/logging)
- nginx proxy (loadbalancing)
- helm package manager
- role based access control (rbac) with Auth0
