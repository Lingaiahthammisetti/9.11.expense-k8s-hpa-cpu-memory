
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