# Stage 3 Hosting a Single Container

## Docker File 

Finally you can deploy your first Container/Pod onto your Kubernetes Cluster.

We prepared a simple Dockerfile for you.

`stage_3_SingleContainer/Dockerfile`

```
FROM nginx:1.14.2

COPY index.html /usr/share/nginx/html/
```

As you can see it only uses a basic nginx image and copies over the "index.html" in the same folder into the Container.


## Authentication

In the Pipeline you will see that we first authenticate against the AKS Cluster and the ACR.

`az aks get-credentials ...`

`az acr login`
## Docker Build

The image is created via Docker Build and first stored locally under the name `2021hackathon.azurecr.io/nginx:latest`. 
Then we push the Image to our central Container Registry.


`.github/workflows/frontend_backend.yml`
```
name: 'Stage 3 Single Container'
on: [workflow_dispatch]


jobs:
  container:
    name: 'Container Build'
    runs-on: ubuntu-latest
    environment: production
    
    env:
      registryName: "2021hackathon"
      registryLoginServer: "2021hackathon.azurecr.io"

    defaults:
      run:
        shell: bash
        #working-directory: ./container
    steps:
      
      - name: Checkout
        uses: actions/checkout@v2

      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Set Environment
        run: |
          az aks get-credentials --resource-group ${{ secrets.UNIQUE_NAME }} --name ${{ secrets.UNIQUE_NAME }}

      - name: Docker Login to Registry
        run: |
          az acr login --name ${{ env.registryName }}
          
      # Build Old NGINX Frontend
      - name: Build NGINX Base Frontend
        run: docker build -t ${{ env.registryLoginServer }}/nginx:latest ./stage_3_SingleContainer

      - name: Push NGINX Base Frontend
        run: docker push ${{ env.registryLoginServer }}/nginx:latest

```

Finally we apply the Image as a Service on our Kubernetes Cluster.
The following code will apply all `*.yml` files in the `stage_3_SingleContainer/` folder

`.github/workflows/frontend_backend.yml`
```
      - name: Deploy Manifests
        run: |
          kubectl apply -f ./stage_3_SingleContainer/
```

## deployment.yaml

In the deployment.yaml we define our nginx server service.
The Service name will be nginx and it will be available via Port 80.
Also we use the freshly generated `2021hackathon.azurecr.io/nginx:latest`

<br>

`stage_3_SingleContainer/deployment.yaml`
```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hello-world
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: 2021hackathon.azurecr.io/nginx:latest
        ports:
        - containerPort: 80
        resources: {}

```
## ingress.yaml

In the ingress.yml we define that our Service named "nginx"

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-single-container
  annotations:
    #nginx.org/rewrites: "serviceName!=nginx rewrite=/$3"
    #nginx.ingress.kubernetes.io/app-root: 
    #nginx.ingress.kubernetes.io/rewrite-target: /$3
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80

```

## Introduction

# 1. Setting up the Infrastructure Pipeline

