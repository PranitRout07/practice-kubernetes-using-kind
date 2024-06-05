# Steps To Create Users in K8S

1. Generate a key for the user .
```
openssl genrsa -out pranit.key 2048
openssl req -new -key pranit.key -out pranit.csr -subj "/CN=pranit/O=group1"
```
2. Generate the base64 encoded certificate.
```
cat pranit.csr | base64 | tr -d '\n'
```
3. Create csr.yaml

```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: pranit
spec:
  request: <mention the base64 encoded certificate>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

4. Apply the yaml file .

```
kubectl apply -f csr.yaml 
kubectl certificate approve pranit
```

5. Setup the kube config file
```
kubectl config set-credentials pranit --client-certificate=pranit.crt --client-key=pranit.key
kubectl config get-contexts 
kubectl config set-context pranit-context --cluster=kubernetes --namespace=default --user=pranit
kubectl config use-context pranit-context
```