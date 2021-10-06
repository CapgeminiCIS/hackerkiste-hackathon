```
name: 'Stage 2 Ingress'
on: [workflow_dispatch]

jobs:
  kubernetes:
    name: 'Kubernetes'
    needs: terraform
    runs-on: ubuntu-latest
    environment: production


    defaults:
      run:
        shell: bash
        working-directory: ./stage_3_Ingress
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