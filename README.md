# kubee

This repository contains the codes I used to learn Kubernetes. The aim is to learn Kubernetes from the basic setup using Deployments, ReplicaSets and Service to full production quality deployment integrated with Amazon EKS. 

# Prerequisites

* Docker Installed

# Basics

Learn basics of Kubernetes deployment and kubectl commands.

## Install a Kubernetes Cluster for local development

Minikube is a developer version of a production grade Kubernetes clusters like Amazon EKS. This is meant  for local development only.

1. Install Docker https://docs.docker.com/get-docker/
2. Install kubectl https://kubernetes.io/docs/tasks/tools/
   ```
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin/kubectl
   sudo chown root: /usr/local/bin/kubectl
   kubectl version --client
   ```
3. Install Minikube
    ```
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
    sudo install minikube-darwin-amd64 /usr/local/bin/minikube
    ```
4. Start Minikube
    ```
    minikube start
    ```
5. Start a nginx application in your Minikube cluster
    ```
    # deploy nginx
    kubectl create deployment hello-minikube --image=docker.io/nginx:1.23
    # expose port 80 of nginx
    kubectl expose deployment hello-minikube --type=NodePort --port=80
    # forward localport 7080 to your nginx 80 port
    kubectl port-forward service/hello-minikube 7080:80
    ```
6. Check your local nginx instance at http://localhost:7080/.
7. Destroy deployment and service (clean-up)
    ```
    kubectl delete service hello-minikube
    kubectl delete deployment hello-minikube
    kubectl get all
    ```
   
* Reference
  * [minikube start](https://minikube.sigs.k8s.io/docs/start/)
  * [minikube start issue](https://github.com/kubernetes/minikube/issues/8770)
  
### Deploy Nginx using Declarative method

In the previous section, we installed Minikube and deployed a nginx web server using imperative method of deployment. In imperative deployment, we use kubectl commands to specify the details of the artifacts we want to deploy. This method of deployment is only meant for adhoc kind of deployment like to test stuff. There is an alternative approach where the definitions of the artifacts (or the parameters of your commands in imperative way) are defined in a YAML file. They call this declarative method of deployment. In this section, we will deploy nginx using declarative method.

1. Define deployment YAML [deployment.yml](basics/declarative/deployment.yml)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-declarative
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.16
          name: nginx
```
2. Define service YAML [service.yml](basics/declarative/service.yml)
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-declarative
  labels:
    app: nginx-lb-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
  selector:
    app: nginx
```
3. Deploy the nginx resources to your Kubernetes cluster
```
kubectl apply -f basics/declarative/deployment.yml 
kubectl apply -f basics/declarative/service.yml   
```
4. Open a new terminal and start Minikube tunnel to enable us to browse the homepage of the deployed nginx
```
minikube tunnel
```
4. In another terminal get the IP address of the nginx service. Copy the IP Address under "EXTERNAL-IP" column.
```
kubectl get services nginx-declarative
```
5. Browse your nginx homepage at http://[EXTERNAL-IP]
6. Clean-up resources. Don't forget to stop your minikube tunnel process first before running commands below.
```
kubectl delete services nginx-declarative
kubectl delete deployment nginx-declarative
```

