# Multi-Node Kubernetes Cluster with Multiple Applications and Replicas



## Project Overview



This project demonstrates how to deploy two Python Flask applications on a three-node Kubernetes cluster using Minikube and Docker.



The applications are deployed with multiple replicas to demonstrate pod distribution, service discovery, and high availability across nodes.



## Applications



### 1. Product Catalog (AppA)



Provides a list of available products through the `/products` endpoint.



Sample products:

\- Laptop â€” 1200

\- Phone â€” 800

\- Headphones â€” 150



\*\*Replicas:\*\* 2



### 2. Shopping Cart (AppB)



Provides endpoints to view and add items to a shopping cart.



\- `GET /cart` â€” retrieves cart contents.

\- `POST /cart` â€” adds an item to the cart.



\*\*Replicas:\*\* 3



Note: The cart is stored in application memory. Each replica has its own separate cart, so data is not shared between replicas.



## Technologies Used



\- Python

\- Flask

\- Docker

\- Kubernetes

\- Minikube

\- kubectl

\- YAML



## Project Structure



```text

kubernetes-multinode-project/

â”œâ”€â”€ product\_catalog.py

â”œâ”€â”€ shopping\_cart.py

â”œâ”€â”€ Dockerfile.product

â”œâ”€â”€ Dockerfile.shopping

â”œâ”€â”€ product\_catalog\_deployment.yaml

â”œâ”€â”€ shopping\_cart\_deployment.yaml

â”œâ”€â”€ product\_catalog\_service.yaml

â”œâ”€â”€ shopping\_cart\_service.yaml

â””â”€â”€ README.md

```



## Architecture



The project uses a three-node Minikube cluster:



\- One control-plane node

\- Two worker nodes



The Product Catalog Deployment maintains two replicas on separate nodes. The Shopping Cart Deployment maintains three replicas, distributed one per node using pod anti-affinity.



Kubernetes Services expose both applications through NodePort.



## Setup and Deployment



### 1. Start the Minikube cluster



```powershell

minikube start --nodes 3 -p devops-multinode --driver=docker

```



If the profile already exists, check its status before starting it again.



### 2. Build Docker images



```powershell

docker build -t product-catalog:latest -f Dockerfile.product .

docker build -t shopping-cart:latest -f Dockerfile.shopping .

```



### 3. Load images into Minikube



```powershell

minikube -p devops-multinode image load product-catalog:latest

minikube -p devops-multinode image load shopping-cart:latest

```



### 4. Deploy applications



```powershell

kubectl --context=devops-multinode apply -f product\_catalog\_deployment.yaml

kubectl --context=devops-multinode apply -f shopping\_cart\_deployment.yaml

```



### 5. Create Services



```powershell

kubectl --context=devops-multinode apply -f product\_catalog\_service.yaml

kubectl --context=devops-multinode apply -f shopping\_cart\_service.yaml

```



### 6. Verify nodes and pods



```powershell

kubectl --context=devops-multinode get nodes

kubectl --context=devops-multinode get pods -o wide

kubectl --context=devops-multinode get services

```



Expected deployment:

\- 3 Ready nodes

\- 2 Product Catalog pods

\- 3 Shopping Cart pods

\- 2 NodePort Services



## Testing



### Product Catalog



Start a service tunnel:



```powershell

minikube -p devops-multinode service product-catalog-service

```



Open the displayed URL with `/products` appended. The endpoint should return the three products as JSON.



### Shopping Cart



Start port forwarding:



```powershell

kubectl --context=devops-multinode port-forward service/shopping-cart-service 8085:80

```



Keep the forwarding terminal open.



In another PowerShell window, retrieve the cart:



```powershell

Invoke-RestMethod -Uri "http://127.0.0.1:8085/cart"

```



Add an item:



```powershell

$body = @{

&#x20;   id = 1

&#x20;   name = "Laptop"

&#x20;   quantity = 1

} | ConvertTo-Json



Invoke-RestMethod -Uri "http://127.0.0.1:8085/cart" -Method Post -ContentType "application/json" -Body $body

```



The POST request should return the added item with its ID, name, and quantity.



## Results



The deployment was verified successfully:



\- All three Kubernetes nodes reached `Ready` status.

\- Both Product Catalog replicas ran on separate worker nodes.

\- All three Shopping Cart replicas ran across the three nodes.

\- The Product Catalog endpoint returned the expected product list.

\- The Shopping Cart accepted a POST request and returned the added laptop item.

\- Both Kubernetes Services were created successfully.



## Learning Outcomes



\- Creating and managing a multi-node Kubernetes cluster.

\- Building and loading Docker images.

\- Deploying applications using Kubernetes Deployments.

\- Configuring pod anti-affinity for node distribution.

\- Exposing applications using Kubernetes Services.

\- Testing HTTP GET and POST endpoints.

\- Verifying pod placement and application availability.




