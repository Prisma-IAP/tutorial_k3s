Copie a configuração abaixo e salve no arquivo **pandaweb.yaml**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front-app
  name: front-app
spec:
  revisionHistoryLimit: 2
  replicas: 3
  selector:
    matchLabels:
      app: front-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: front-app
    spec:
      containers:
      - image: ronaldo87/panda-frontend:latest
        name: front-app
        ports:
        - containerPort: 80
          name: server
---
apiVersion: v1
kind: Service
metadata:
  name: front-app-service
  labels:
    app: front-app
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 31125
  selector:
    app: front-app
  type: NodePort
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: front-app-ingress
  annotations:
spec:
  ingressClassName: nginx
  rules:
  - host: panda-web.io
    http:
      paths:
      - path: "/"
        pathType: Exact
        backend:
          service:
            name: front-app-service
            port:
              number: 80
```