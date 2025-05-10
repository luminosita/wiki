# Run `kubectl` thru SOCKS5 Proxy

### Create SOCKS5 proxy thru SSH

ssh -D 8002 -q -N username@your-bastion-server.example.com

### Option 1: Run `kubectl` thru SOCKS5 proxy on a command line
HTTPS_PROXY=socks5://localhost:8002 kubectl get pods

### Option 2: Export SOCKS5 proxy as a environment variable
export HTTPS_PROXY=socks5://localhost:8002
kubectl get pods

### Option 3: Add SOCKS5 proxy into `kubectl` config 

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LRMEMMW2 # shortened for readability 
    server: https://<API_KUBE_CLUSTER_IP_ADRESS>:443  # the "Kubernetes API" server, in other words the IP address of kubernetes cluster api ip and port
    proxy-url: socks5://localhost:8002   # the "SSH SOCKS5 proxy" in the diagram above
```
