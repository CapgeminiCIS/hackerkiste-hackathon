# Stage 2 Ingress Controller

## Hosting a Single Container
In this stage you will learn about ingress controllers and why we need them.

An ingress controller is a fancy name for a proxy or LoadBalancer which routes the traffic to its designated services inside your Kubernetes cluster. Without it we would need to supply each pod with its own public IP address.

Learn more about:
- [Ingress Controller](https://www.nginx.com/resources/glossary/kubernetes-ingress-controller/)

### Stage 2 Ingress Controller
In our example code the pipeline is already ready to go and will deploy an nginx based ingress controller, which we will use later to create our routes.

To make it easier we used a publicly available Helm chart. But we will not go deeper into it here, so we ask you kindly just to accept this as a service in your cluster to manage your ingoing routing. If you would like to know more about it: [What is Helm?](https://helm.sh/)

### Helm Chart We Used
Helm Chart Name: *ingress-nginx/ingress-nginx*

### Run Your Pipeline
Simply run the stage 2 workflow

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
Check your services again and select the public IP address to connect to the ingress controller. You might experience an expected error as there are currently no routes configured this will be done in the next stage.

Can you find the associated Resource in Azure for the public IP address which Kubernetes provisioned for you?

# Whats Next?
Continue with [Stage 3 Single Container](06_Stage_3_SingleContainer.md).