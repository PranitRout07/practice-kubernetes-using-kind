# Command to apply load on the cluster
kubectl run -i --tty load-generator --image=busybox -- sh -c "while true; do wget -O - http://my-service:8501; done"
