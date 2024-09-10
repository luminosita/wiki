
# Bitnami Sealed Secrets — Kubernetes Secret Management


![](https://miro.medium.com/v2/resize:fit:1392/0*GTublBxyY__jaU4a)

Nowadays we are using  [GitOps](https://foxutech.com/category/kubernetes/argocd/) for application deployment and for that we tend to put all the application’s information and configuration on Git, but same time can we do the same with Kubernetes Secrets YAML file on Git? The answer is definitely NO, as the Secret’s file contains just the base64 encoded value of our sensitive information; which can be passwords, keys, certificates, tokens etc.

Following is a sample YAML file for a Secret: –

```yaml
apiVersion: v1  
kind: Secret  
metadata:  
  name: mysecret  
type: Opaque  
data:  
  username: d2VsY29tZSB0bw==  
  password: Zm94dXRlY2g=
```

And we can easily decode the sensitive information with base64 command like following:

```bash
# echo -n Zm94dXRlY2g= | base64 -d
```
So, definitely we should not store Kubernetes Secrets on Git/any visible systems. There are many different ways to externalize k8s secrets like Hashicorp’s Vault, Azure key vault, external secret operator, Helm Secrets, Secrets Store CSI Driver, Bitnami’s SealedSecret etc. In this blog we are going to explore about Bitnami’s Sealed Secret.

# Sealed Secrets

[Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)  is an open-source Kubernetes controller and a client-side CLI tool from Bitnami that aims to solve the “storing secrets in Git” part of the problem, using asymmetric crypto encryption. Sealed Secrets with an RBAC configuration preventing non-admins from reading secrets is an excellent solution for the entire problem.

# How it works

![](https://miro.medium.com/v2/resize:fit:1280/0*fBTPKxQmuabvZjsG)

It works as below;

-   Encrypt the secret on the developer machine using a public key and the kubeseal CLI. This encodes the encrypted secret into a Kubernetes Custom Resource Definition (CRD)
-   Deploy the CRD to the target cluster
-   The Sealed Secret controller decrypts the secret using a private key on the target cluster to produce a standard Kubernetes secret.

The private key is only available to the Sealed Secrets controller on the cluster, and the public key is available to the developers. This way, only the cluster can decrypt the secrets, and the developers can only encrypt them.

# Install

# Client

```bash
# wget https://github.com/bitnami-labs/sealed-secrets/releases/download/<release-tag>/kubeseal-<version>-linux-amd64.tar.gz  
```

Example:   
```bash
# wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.19.4/kubeseal-0.19.4-linux-amd64.tar.gz  
# tar -xvzf kubeseal-<version>-linux-amd64.tar.gz kubeseal  
# install -m 755 kubeseal /usr/local/bin/kubeseal
```

# Server

# Sealed Secrets Controller

Current deployment process can be done manual helm install command or kubectl on targeted cluster:

# Installing via helm chart

```bash
# helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets  
# helm dependency update sealed-secrets  
# helm install sealed-secrets sealed-secrets/sealed-secrets --namespace kube-system --version 2.7.4
```

# Installing via Kubectl

```bash
# wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.1/controller.yaml  
# kubectl apply -f controller.yaml
```

You can ensure that the relevant Pod is running as expected by executing the following command:

```bash
# kubectl get pods -n kube-system | grep sealed-secrets-controller
```

# Sealing the Secrets

# Create Secret

The code for Kubernetes Secret is given below. Save the code in a file name secrets.yaml.

```yaml
apiVersion: v1  
kind: Secret  
metadata:  
  creationTimestamp: null  
  name: my-secret  
  namespace: test-ns  
data:  
  password: dXNlcm5hbWU=        # base64 encoded username  
  username: cGFzc3dvcmQ=        # base64 encoded password
```

Here the username and passwords are base64 encoded. Change with appropriate secret for your use. You can retrieve the generated public key certificate using kubeseal and store it on your local disk:

```bash
# kubeseal --fetch-cert > public-key-cert.pem
```

kubeseal encrypts the Secret using the public key that it fetches at runtime from the controller running in the Kubernetes cluster. If a user does not have direct access to the cluster, then a cluster administrator may retrieve the public key from the controller logs and make it accessible to the user.

A SealedSecret CRD is then created using kubeseal as follows using the public key file:

```bash
# kubeseal --format=yaml --cert=public-key-cert.pem < secret.yaml > sealed-secret.yaml
```

Note that the keys in the original Secret — namely, username and password — are not encrypted in the SealedSecret, only their values are. You may change the names of these keys, if necessary, in the SealedSecret YAML file and still be able to deploy it successfully to the cluster. However, you cannot change the name and namespace of the SealedSecret. Doing so will invalidate the SealedSecret because the name and namespace of the original Secret are used as input parameters in the encryption process.

The YAML manifest that pertains to the Secret is no longer needed and may be deleted. The SealedSecret is the only resource that will be deployed to the cluster as follows:

```bash
# kubectl apply -f sealed-secret.yaml
```

Once the SealedSecret CRD is created in the cluster, the controller knows and unseals the underlying Secret using the private key and deploys it to the same namespace. This is seen by looking at the logs from the controller.