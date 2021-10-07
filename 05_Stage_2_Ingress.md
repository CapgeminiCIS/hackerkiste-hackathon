# Stage 2 Ingress Controller

## Hosting a Single Container
### Introduction
In this stage you will learn about Ingress Controllers and why we need them.

A Ingress Controller is a fancy name for a Proxy or LoadBalancer which routes the Traffic to its designated services inside your Kubernetes Cluster.
Without it we would need to supply each Pod with its own public ip address.

[Learn more about Ingress Controller](https://www.nginx.com/resources/glossary/kubernetes-ingress-controller/)

### Stage 2 Ingress Controller

In our example code the pipeline is already ready to go and will deploy an nginx based Ingress Controller, which we will use later to create our routes.

To make our life's easier we used a publicly available Helm Chart. But we wont go deeper into Helm Charts here, so we ask you kindly just to accept this as a service in your cluster to manage your ingoing routing.
[What is Helm?](https://helm.sh/)

### Helm Chart we used

Helm Chart Name: <br>
ingress-nginx/ingress-nginx

### Run your Pipeline

Simply run Stage 2 Workflow

`#File: .github/workflows/ingress.yml`
```
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Set Environment
        run: |
          az aks get-credentials --resource-group ${{ secrets.UNIQUE_NAME }} --name ${{ secrets.UNIQUE_NAME }}

      - name: "Add Helm Repo"
        run: | 
          helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
      
      - name: "Update Helm Repo"
        run: | 
          helm repo update

      - name: "Install Ingress Controller"
        run: |
          helm upgrade ingress-nginx ingress-nginx/ingress-nginx --install --create-namespace --namespace ingress-controller      
```
## Connect to the Ingress Controller

Check your Services again and select the Public IP address to connect to the Ingress Controller.
You might experience an expected error as there are currently no routes configured this will be done in the Next Stage.

Can you find the associated Resource in Azure for the Public IP address which Kubernetes Provisioned for you?
## Whats Next?

Continue with [Stage 3 Single Container](06_Stage_3_SingleContainer.md)