## Kubernetes Dashboard Ui - Installation process
Url: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Prereq Commands
```bash
kubectl get namespace
```
## Install Kubernetes Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

## Get all in kubernete-dashboard namespace
kubectl -n kubernetes-dashboard get all

## Get the dashboard port for the service kubernetes-dashboard
kubectl -n kubernetes-dashboard describe service kubernetes-dashboard

## There two methods to connect to the Dashboard
### - PortForwarding
### - to use NodePort

## Connect to Dashboard using Port Forwarding
kubectl -n kubernetes-dashboard port-forward kubernetes-dashboard-5996555fd8-vl4bh 8000:8443

## PortForward method: Access Dashboard
## Open Broswer and enter "https://localhost:8000

## NodePort Method:
## Edit the kubernetes-dashboard service and change the type from "ClusterIP" to "NodePort" and also
## add "nodePort: 32323" under targetPort
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard

## Verify the changes
kubectl -n kubernetes-dashboard describe service kubernetes-dashboard
kubectl -n kubernetes dashboard get service

## Get the ip address of the worker nodes
kubectl get nodes -o wide

## Access the Kubernetes Dashboard through the Browser using the IP of one of the Worker Nodes
## for example URL: https://192.168.0.8:32323

## Login to the Kubernetes Dashboard using Token
## Get the Token for the service account
kubectl -n kubernetes-dashboard get sa
kubectl -n kubernetes-dashboard describe sa kubernetes-dashboard

## With the token from the below account you can access the Dashboard, but it hasn't got any priviledges
kubectl -n kubernetes-dashboard describe secret kubernetes-dashboard-token-q5vcg

## To fix this we need to deploy the ServiceAccount and ClusterRoleBinding in "sa_cluster_admin.yam"
kubectl create -f sa_cluster_admin.yaml

## Get the Token for the Service Account "dashboard-admin" which we just created
kubectl -n kube-system get sa
kubectl -n kube-system describe sa dashboard-admin
kubectl -n kube-system describe secret dashboard-admin-token-t9fxb

## Copy the Token from the command above into the WebUI under Token and you login to the
## Kubernetes Dashboard

