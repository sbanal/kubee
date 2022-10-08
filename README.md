# kubee

This repository contains the codes I used to learn Kubernetes. The aim is to learn Kubernetes from the basic setup using Deployments, ReplicaSets and Service to full production quality deployment integrated with Amazon EKS. 

# Prerequisites

* Docker Installed

# Basics

Learn basics of Kubernetes deployment and kubectl commands using a local Kubernetes cluster called Minikube.

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

1. Define deployment YAML [deployment.yml](basics/deployment.yml)
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
2. Define service YAML [service.yml](basics/service.yml)
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
kubectl apply -f basics/deployment.yml 
kubectl apply -f basics/service.yml   
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
kubectl delete -f basics/service.yml
kubectl delete -f basics/deployment.yml  
```

* Reference
  * [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
  * [LoadBalancer Service](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)
  
# Basics in the Cloud

Learn the basics of Kubernetes deployment using Amazon EKS. EKS is a managed Kubernetes cluster provided by AWS.

## Create an Amazon EKS Cluster

1. Install AWS CLI https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
2. Configure AWS CLI credentials https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds
3. Install EKS CLI command called eksctl https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
4. Create an EKS Cluster
    ```
    eksctl create cluster -f basics-cloud/kubee-cluster.yml
    ``` 
    WARNING: Do not forget to destroy this cluster. This will cost you money.
    The output of the command should show a message similar to below:
    ```
    [✔]  EKS cluster "kubee-cluster" in "ap-southeast-2" region is ready
    ```
5. Check cluster and node status 
   ```
   eksctl get cluster
   kubectl get nodes -o wide
   ```

## Deploy nginx to Amazon EKS

1. Deploy the nginx resources to your Kubernetes cluster
    ```
    kubectl apply -f basics/deployment.yml 
    kubectl apply -f basics/service.yml   
    ```
1. Check status of nodes. Wait for a few minutes and run command below sevral times until there are 2 ready pods.
    ```
    kubectl get all
    ```
    Output:
    ```
    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/nginx-declarative-6d4cf56db6-8vxrl   1/1     Running   0          3m9s
    pod/nginx-declarative-6d4cf56db6-nhf5r   1/1     Running   0          3m9s
    
    NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)        AGE
    service/kubernetes          ClusterIP      10.100.0.1       <none>                                                                         443/TCP        29m
    service/nginx-declarative   LoadBalancer   10.100.115.161   acd6cbaa6e7134c80a5162fe848b3e21-1422704247.ap-southeast-2.elb.amazonaws.com   80:32487/TCP   3m2s
    
    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-declarative   2/2     2            2           3m9s
    
    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-declarative-6d4cf56db6   2         2         2       3m9s
    ```
2. Get the load balancer DNS. Copy the value in the column "EXTERNAL-IP". 
    ```
    kubectl get service nginx-declarative
    ```
3. Open a browser and enter http://[EXTERNAL-IP] to open the nginx home page.
4. Clean-up.
    ```
    kubectl delete -f basics/service.yml
    kubectl delete -f basics/deployment.yml  
    ```

### Delete your EKS cluster

1. Delete your cluster
    ```
    eksctl delete cluster -f basics-cloud/kubee-cluster.yml
    ```
2. Check Cluster status. Wait for a few minutes before running this command.
    ```
    eksctl get cluster
    ```
   
* References
   * [AWS EKS pricing](https://aws.amazon.com/eks/pricing/)
   * [eksctl create cluster](https://eksctl.io/usage/creating-and-managing-clusters/)
   * [Estimated annual cost of this cluster](https://calculator.aws/#/estimate?id=03636d4199a0e931a35772d9c412fcbb823fa5f6)
