# Node.js - PostgreSQL App with ArgoCD

#### Prerequisites:
- docker or docker desktop
- helm
- minikube

#### Setting the PostgreSQL Database

Before creating the Kubernetes cluster, our application needs to connect a postgresql db. We can simply create a db with the docker-compose and simple credentials.
```
docker-compose -f docker/postgres/docker-compose.yaml up -d
```
Now we need to create our database and some data in it. Sample queries for our application specific are provided in "./docker/postgres/food.sql". I used pgAdmin to connect and query to my database. "www.pgadmin.org"

#### Cluster & Environment

After database gets ready for our node api, we need a Kubernetes cluster, run the command below to create a single node cluster with minikube.
```
minikube start
```
We will use secrets that aws secrets manager provided for production environment, so we need to handle these secrets in our kubernetes cluster with external secrets. 
```
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace 
```
ArgoCD will play the main role on this project, so simply install ArgoCD with the commands below.
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
To login ArgoCD GUI we need to forward the web service to the outside of the cluster.
(you can run this command on another terminal as it will run on attached mode)
```
kubectl port-forward svc/argocd-server -n argocd 8070:443
```
The username will be admin, and you should decode the initial admin secret.
```
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```
We should take the value in "data.password" key and decode it on base64, decoding method depends on your operating system, so it is also safe to use "www.base64decode.org" website. 

#### Deploying & Testing the Application with ArgoCD

Now, all required aspects have been installed to run our helm application with argocd. We can now apply our argocd manifests to build our helm chart on remote git repository.
```
kubectl apply -f .\argocd\application-dev.yaml
kubectl apply -f .\argocd\application-prod.yaml
```
After waiting a while applications to be ready. We can forward these applications ports also.
(commands below will require to be attached on two separate terminals more)
```
kubectl port-forward svc/sample-node-app -n dev 8080:8080
kubectl port-forward svc/sample-node-app -n prod 8090:8090
```
For checking the api's if they are healthy;
dev: http://localhost:8080/food
prod: http://localhost:8090/food