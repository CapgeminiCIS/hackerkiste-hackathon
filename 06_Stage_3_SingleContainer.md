# Stage 3 Hosting a Single Container

## Introduction

Finally you will deploy your first Container/Pod onto your Kubernetes Cluster.
Which you can then access as a Website.

- For that you will first visit your the [Docker Build Process](https://docs.docker.com/engine/reference/commandline/build/).
- Also you will check out how to Authenticate in a Pipeline against Azure.
- In the End the Container will be Deployed and you provide the Ingress Controller with its needed Configuration.

## Docker File 


We prepared a simple Dockerfile for you.

`stage_3_SingleContainer/Dockerfile`

```
FROM nginx:1.14.2

COPY index.html /usr/share/nginx/html/
```

As you can see it only uses a basic nginx image and copies over the "index.html" in the same folder into the final Image.

[Learn more about Docker Images](https://www.computerweekly.com/de/definition/Docker-Image)
## Authentication

In the Pipeline you will see that we first authenticate against the AKS Cluster and the ACR.

`az aks get-credentials`

`az acr login`
## Docker Build

Next we the image is created via Docker Build and is first stored locally under the name `2021hackathon.azurecr.io/nginx:latest`. 

It is important to have the Url of the Image Repository in the Name as this will signal the Docker Daemon where to push the Image to. 

This is done with the push command, which pushes the local Image to our central Container Registry.

Images with the same Name and Tag will override each other once pushed to the Registry. If you want to create Unique Images and maybe alter the Website we provided to your liking you should rename the Image to your liking everywhere its referenced.

`.github/workflows/frontend_backend.yml`
```
         
      # Build Old NGINX Frontend
      - name: Build NGINX Base Frontend
        run: docker build -t ${{ env.registryLoginServer }}/nginx:latest ./stage_3_SingleContainer

      - name: Push NGINX Base Frontend
        run: docker push ${{ env.registryLoginServer }}/nginx:latest

```

## Pushing the Service Definitions to Kubernetes

Finally we apply the Image as a Service on our Kubernetes Cluster.
The following code will apply all `*.yml` files in the `stage_3_SingleContainer/` folder

`.github/workflows/frontend_backend.yml`
```
      - name: Deploy Manifests
        run: |
          kubectl apply -f ./stage_3_SingleContainer/
```

### Definition 1 deployment.yaml

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
### Definition 2 ingress.yaml

In the ingress.yml we define that our Service named "nginx" and is available over the Ingress Controller without a suffix.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-single-container
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

After running the Pipeline Stage 3 try connecting to your Ingress Controller again did anything change?
## Whats Next?

Continue with [Stage 4 Frontend Backend](07_Stage_4_Frontend_Backend.md)