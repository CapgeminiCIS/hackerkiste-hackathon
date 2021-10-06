# Stage 2 Ingress Controller

In this stage you will learn all about Ingress Controllers and why we need them.

A Ingress Controller is a fancy name for a Proxy or LoadBalancer which routes the Traffic to its designated Services inside your Kubernetes Cluster

https://www.nginx.com/resources/glossary/kubernetes-ingress-controller/

In our Example Code the Pipeline is already ready to go and will Deploy an nginx based Ingress Controller with predefined.

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

`#File: stage_2_Ingress/ingress.yaml`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
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
          - path: /backend/java
            pathType: Prefix
            backend:
              service:
                name: java
                port:
                  number: 80
          - path: /backend/go
            pathType: Prefix
            backend:
              service:
                name: go
                port:
                  number: 80                
```