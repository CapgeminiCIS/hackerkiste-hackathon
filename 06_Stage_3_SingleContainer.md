# Stage 3 Hosting a Single Container

## Introduction
Finally you will deploy your first container inside a pod onto your Kubernetes cluster. You can then access it as a Website using your browser.

For that you will first visit the [Docker Build Process](https://docs.docker.com/engine/reference/commandline/build/). Also you will check out how to authenticate in a pipeline against Azure. In the end the container will be deployed and you provide the ingress controller with its needed configuration.

## Docker File 
We prepared a simple Dockerfile for you: `stage_3_SingleContainer/Dockerfile`

```
FROM nginx:1.14.2

COPY index.html /usr/share/nginx/html/
```

As you can see it only uses a basic nginx image and copies over the `index.html` in the same folder into the final Image.

Learn more about:
- [Docker Images](https://www.computerweekly.com/de/definition/Docker-Image)

## Authentication
In the pipeline you will see that we first authenticate against the AKS cluster and the ACR.

- `az aks get-credentials`
- `az acr login`

## Docker Build
Next the image is created via Docker Build and is first stored locally under the name `hackerkisteregistry.azurecr.io/nginx:latest`. 

It is important to have the URL of the image repository in the name as this will signal the Docker daemon where to push the image to. 

This is done with the push command, which pushes the local image to our central container registry.

Images with the same name and tag will override each other once pushed to the registry. If you want to create unique images and maybe alter the website we provided to your liking you should rename the image to your liking everywhere its referenced.

`.github/workflows/frontend_backend.yml`
```     
      # Build old NGINX frontend
      - name: Build NGINX Base Frontend
        run: docker build -t ${{ env.registryLoginServer }}/nginx:latest ./stage_3_SingleContainer

      - name: Push NGINX Base Frontend
        run: docker push ${{ env.registryLoginServer }}/nginx:latest
```

## Pushing the Service Definitions to Kubernetes
Finally we apply the image as a service on our Kubernetes cluster. The following code will apply all `*.yml` files in the `stage_3_SingleContainer/` folder

`.github/workflows/frontend_backend.yml`
```
      - name: Deploy Manifests
        run: |
          kubectl apply -f ./stage_3_SingleContainer/
```

### Definition 1 `deployment.yaml`
In the `deployment.yaml` we define our nginx server service.
The service name will be nginx and it will be available via Port 80. Also we use the freshly generated `hackerkisteregistry.azurecr.io/nginx:latest`

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
        image: hackerkisteregistry.azurecr.io/nginx:latest
        ports:
        - containerPort: 80
        resources: {}

```
### Definition 2 `ingress.yaml`
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

# Whats Next?
Continue with [Stage 4 Frontend Backend](07_Stage_4_Frontend_Backend.md).
