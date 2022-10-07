# kubee

This repository contains the codes I used to learn Kubernetes. The aim is to learn Kubernetes from the basic setup using Deployments, ReplicaSets and Service to full production quality deployment integrated with Amazon EKS. 

# Prerequisites

* AWS Account 
* kubectl
* eksctl
* Kotlin

# Setup

Learn basics of Kubernetes deployment and kubectl commands.

## Install a Kubernetes Cluster for local development

Minikube is a developer version of a production grade Kubernetes clusters like Amazon EKS. This is meant  for local development only.

1. Install Minikube
    ```
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
    sudo install minikube-darwin-amd64 /usr/local/bin/minikube
    ```
2. Start Minikube
    ```
    minikube start
    ```
3. Start a nginx application in your Minikube cluster
    ```
    # deploy nginx
    kubectl create deployment hello-minikube --image=docker.io/nginx:1.23
    # expose port 80 of nginx
    kubectl expose deployment hello-minikube --type=NodePort --port=80
    # forward localport 7080 to your nginx 80 port
    kubectl port-forward service/hello-minikube 7080:80
    ```
4. Check your local nginx instance at http://localhost:7080/.
5. Destroy deployment and service (clean-up)
    ```
    kubectl delete service hello-minikube
    kubectl delete deployment hello-minikube
    kubectl get all
    ```
   
* Reference
  * [minikube start](https://minikube.sigs.k8s.io/docs/start/)
  * [minikube start issue](https://github.com/kubernetes/minikube/issues/8770)
  
### 
