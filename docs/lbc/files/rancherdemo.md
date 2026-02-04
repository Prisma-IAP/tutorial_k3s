Copie a configuração abaixo e salve no arquivo **rancherdemo.yaml**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rancher-app
spec:
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      name: rancher-app
  template:
    metadata:
      labels:
        name: rancher-app
    spec:
      containers:
        - name: rancher-demo
          image: ronaldo87/rancher-demo:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
              name: web
              protocol: TCP
          resources:
            limits:
              cpu: 200m
              memory: 100m # fix here
            requests:
              cpu: 100m
              memory: 100m # fix here
          env:
          - name: CONTAINER_COLOR
            value: orange
          - name: PETS
            value: chameleons
          - name: TITLE
            value: TCC Load Balancer Demo
          readinessProbe:
            httpGet:
              port: web
              path: /
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: rancher-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: rancher-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 90
---
apiVersion: v1
kind: Service
metadata:
  name: rancher-service
spec:
  type: NodePort
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
      nodePort: 31120
  selector:
    name: rancher-app
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rancher-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: rancher-demo-app.io
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: rancher-service
              port:
                number: 8080
```