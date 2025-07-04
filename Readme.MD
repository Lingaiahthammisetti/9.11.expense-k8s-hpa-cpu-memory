# How to create and delete all pods? 
```
kubectl apply -f mysql/manifest.yaml
```
```
kubectl apply -f backend/manifest.yaml
```
```
kubectl apply -f frontend/manifest.yaml
```
```
kubectl apply -f frontend/hpa.yaml
```
```
kubectl apply -f debug/manifest.yaml
```
```
kubectl apply -f namespace.yaml
```

```
kubectl delete pods --all -n your-namespace
```

# 1. MySQL Dockerfile: In docker best practices, We run images as non-root user by adding USER expense at the end of the dockerfile. 
FROM mysql:8.0
# not a human, only for system
RUN adduser -r expense
ENV MYSQL_ROOT_PASSWORD=ExpenseApp@1 \
    MYSQL_USER=expense \
    MYSQL_PASSWORD=ExpenseApp@1 \
    MYSQL_DATABASE=transactions
RUN chown -R expense:expense /var/lib/mysql /var/run/mysqld
COPY scripts/*.sql /docker-entrypoint-initdb.d/

# 2. MySQL Dockerfile: Remaining all, We run MySQL dockerfile as root user by not including USER expense.
FROM mysql:8.0
ENV MYSQL_ROOT_PASSWORD=ExpenseApp@1
RUN groupadd expense && \
    useradd -g expense expense && \
    chown -R expense:expense /var/lib/mysql /var/run/mysqld /docker-entrypoint-initdb.d
ADD scripts/*.sql /docker-entrypoint-initdb.d
USER expense
    # MYSQL_DATABASE=transactions \
    # MYSQL_USER=expense \
    # MYSQL_PASSWORD=ExpenseApp@1

# Note: We have added resources limits to backend and frontend servers.
resources: # added for HPA autoscalling in Kubernetes
  requests:
    cpu: "100m"
    memory: "256Mi"
  limits:
    cpu: "150m"
    memory: "256Mi"

and Horizontal Pod Autoscaling is also created.

hpa.yaml
=============
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend
  namespace: expense
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 20

$yum install httpd-tools -y
ab -n 10000 -c 100 "http://a5cc80af4a20545da8e9840001bcd083-446768232.us-east-1.elb.amazonaws.com/"
