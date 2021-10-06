# Stage 2 Ingress Controller

## Hosting a Single Container

In this stage you will learn all about Ingress Controllers and why we need them.

A Ingress Controller is a fancy name for a Proxy or LoadBalancer which routes the Traffic to its designated Services inside your Kubernetes Cluster

https://www.nginx.com/resources/glossary/kubernetes-ingress-controller/

In our Example Code the Pipeline is already ready to go and will Deploy an nginx based Ingress Controller, which we will use later to create our Routes.

To make our Lives easier we used a Publicly available Helm Chart. But we wont go deeper into Helm Charts here so you kindly just need to Accept this as a Service in your Cluster to Manage your Input Signals.
https://helm.sh/
ingress-nginx/ingress-nginx


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
