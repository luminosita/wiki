```
#config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
```

```
#values.yaml
proxmox:
  endpoint: "192.168.1.6"
  insecureSkipTLSVerify: true
  tokenID: "root@pam!ias"
  secret: "xxxxxx-xxxxxxx-xxxxxxx-xxxxxx"
  username: ""
  password: ""
```

```bash
$ kind create cluster --config config.yaml
$ helm repo add kubemox https://alperencelik.github.io/helm-charts/
$ helm repo update
$ helm install kubemox kubemox/kubemox -f values.yaml
```

```bash
$ helm repo add crossplane-stable https://charts.crossplane.io/stable
$ helm repo update
$ helm install crossplane \
    --namespace crossplane-system \
    --create-namespace crossplane-stable/crossplane
```

```
#kubernetes-provider.yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-kubernetes
spec:
  package: xpkg.upbound.io/upbound/provider-kubernetes:v0.18.0
```

```
#config-in-cluster.yaml
apiVersion: kubernetes.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: kubernetes-provider
spec:
  credentials:
    source: InjectedIdentity
```

```bash
$ kubectl apply -f kubernetes-provider.yaml
$ SA=$(kubectl -n crossplane-system get sa -o name | grep provider-kubernetes | sed -e 's|serviceaccount\/|crossplane-system:|g')
$ kubectl create clusterrolebinding provider-kubernetes-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"
$ kubectl apply -f config-in-cluster.yaml
```

```bash
```