### Steps To Deploy Movie-recommender application on kubernetes cluster 

1)
Cluster created with three nodes one for control plane and two for worker nodes . And in this config file it forwards the traffic from nodeport 32215 to host port 80.
<br>
config.yaml
</br>
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 32215
    hostPort: 80
- role: worker
- role: worker
```
Create the kubernetes cluster using the below command . 
```
kind create cluster --config config.yaml
```

2)

After creating the cluster follow the below yaml files to deploy and autoscale an application on the Kubernetes cluster.

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: movie-recommender
  labels:
    app: movie-recommender
spec:
  replicas: 1
  selector:
    matchLabels:
      app: movie-recommender
  template:
    metadata:
      labels:
        app: movie-recommender
    spec:
      containers:
      - name: movie-recommender
        image: movie-recommender:latest
        ports:
        - containerPort: 8501
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
```
```
kubectl apply -f Deployment.yaml
```

3)

Download the metrics server yaml file

https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

Then after downloading the components.yaml , under the section container add this command part (line number 140 in components.yaml): 

        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP

Create the metric server using the command below
```
kubectl apply -f components.yaml
```
4)

service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: movie-recommender
  ports:
    - port: 8501
      targetPort: 8501
      nodePort: 32215
```
Create a service to connect the deployment from the localhost.
```
kubectl apply -f service.yaml
```
5)

hpa.yaml
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
 name: movie-recommender
spec:
 scaleTargetRef:
   apiVersion: apps/v1
   kind: Deployment
   name: movie-recommender
 minReplicas: 1
 maxReplicas: 10
 targetCPUUtilizationPercentage: 50
```
Create a horizontal pod autoscaler (this will scale up or scale down the number of pods on the user traffic)
```
kubectl apply -f HPA.yaml
```

### Command to apply load on the cluster
```
kubectl run -i --tty load-generator --image=busybox -- sh -c "while true; do wget -O - http://my-service:8501; done"
```
 
