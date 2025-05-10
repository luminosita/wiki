# A Simple CA Setup with Kubernetes Cert Manager

## Install Cert Manager

First thing we need to do is install cert-manager. We will assume you will be using  [minikube](https://minikube.sigs.k8s.io/docs/)  to get started.

Let’s use Helm to install cert-manager. Start by adding the appropriate repo:

```bash
$ helm repo add jetstack https://charts.jetstack.io  
$ helm repo update
```

After the repo has been added, install cert-manager:

```bash
$ helm install \  
    cert-manager jetstack/cert-manager \  
    --namespace test \  
    --create-namespace \  
    --version v1.11.0 \  
    --set installCRDs=true
```

Note that we will be installing cert-manager in the  `test`  namespace, and we will be installing version 1.11.0.

## Create Your CA

With cert-manager installed we can start building our CA. We will start with creating a self-signed certificate that our CA will use. To do so we will first need to create a self-signed certificate issuer. Create a yaml file called  `cert-manager-ss-issuer.yaml`  with the following content:

```yaml
apiVersion:  cert-manager.io/v1  
kind:  Issuer  
metadata:  
  name:  selfsigned-issuer  
  namespace:  test  
spec:  
  selfSigned: {}
```
Create this issuer:

```bash
$ kubectl create -f cert-manager-ss-issuer.yaml
```

With our issuer created, let’s create a self-signed certificate. Create a yaml file called  `cert-manager-ca-cert.yaml`  with the following content:

```yaml
apiVersion: cert-manager.io/v1  
kind: Certificate  
metadata:  
  name: test-ca  
  namespace: test  
spec:  
  isCA: true  
  commonName: test-ca  
  subject:  
    organizations:  
      - ACME Inc.  
    organizationalUnits:  
      - Widgets  
  secretName: test-ca-secret  
  privateKey:  
    algorithm: ECDSA  
    size: 256  
  issuerRef:  
    name: selfsigned-issuer  
    kind: Issuer  
    group: cert-manager.io
```
Create the certificate:

```bash
$ kubectl create -f cert-manager-ca-cert.yaml
```

With the certificate created, we can go and inspect it:

```bash
$ kubectl -n test get certificate  
NAME      READY   SECRET           AGE  
test-ca   True    test-ca-secret   16s
```

Next inspect the secret:

```bash
$ kubectl -n test get secret test-ca-secret  
NAME             TYPE                DATA   AGE  
test-ca-secret   kubernetes.io/tls   3      71s
```

Excellent! This secret contains the  `ca.crt`,  `tls.crt`, and  `tls.key`  that belong to the CA itself.

Now it’s time to create our CA issuer. Create a file called  `cert-manager-ca-issuer.yaml` with the following:

```yaml
apiVersion: cert-manager.io/v1  
kind: Issuer  
metadata:  
  name: test-ca-issuer  
  namespace: test  
spec:  
  ca:  
    secretName: test-ca-secret
```

Important to note here is that we will be using an  `Issuer`  and not a  `ClusterIssuer`. The main difference between the two is that an  `Issuer`  can only issue certificates within the same namespace! If you want your CA to issue certificates in other namespaces as well, you will have to use the  `ClusterIssuer`. See the  [cert-manager documentation](https://cert-manager.io/docs/configuration/ca/)  for more info about this.

## Issue CA Signed Certificate

With our issuer ready we can go ahead and issue a certificate from this CA. Create a file called  `test-server-cert.yaml`:

```bash
apiVersion: cert-manager.io/v1  
kind: Certificate  
metadata:  
  name: test-server  
  namespace: test  
spec:  
  secretName: test-server-tls  
  isCA: false  
  usages:  
    - server auth  
    - client auth  
  dnsNames:  
  - "test-server.test.svc.cluster.local"  
  - "test-server"  
  issuerRef:  
    name: test-ca-issuer  
---  
apiVersion: cert-manager.io/v1  
kind: Certificate  
metadata:  
  name: test-client  
  namespace: test  
spec:  
  secretName: test-client-tls  
  isCA: false  
  usages:  
    - server auth  
    - client auth  
  dnsNames:  
  - "test-client.test.svc.cluster.local"  
  - "test-client"  
  issuerRef:  
    name: test-ca-issuer
```

What we are doing above is actually creating two certificates: a server certificate, and a client certificate. The server certificate will be used by the server to certify its identity. The client certificate will be used to authenticate the client with the server.

Go ahead and create these two certificates:

```bash
$ kubectl create -f test-server-cert.yaml
```

Let’s see if things work! First validate if our server certificate against our CA:

```bash
$ openssl verify -CAfile \  
<(kubectl -n test get secret test-ca-secret -o jsonpath='{.data.ca\.crt}' | base64 -d) \  
<(kubectl -n test get secret test-server-tls -o jsonpath='{.data.tls\.crt}' | base64 -d)  
/dev/fd/16: OK
```

Nice! Our server certificates validates correctly against our CA. Let’s now create a test server so we can try out our client certificate. Using  `openssl s_server`  utility, launch a server:

```bash
$ echo Hello World! > test.txt  
$ openssl s_server \  
  -cert <(kubectl -n test get secret test-server-tls -o jsonpath='{.data.tls\.crt}' | base64 -d) \  
  -key <(kubectl -n test get secret test-server-tls -o jsonpath='{.data.tls\.key}' | base64 -d) \  
  -CAfile <(kubectl -n test get secret test-server-tls -o jsonpath='{.data.ca\.crt}' | base64 -d) \  
  -WWW -port 12345  \  
  -verify_return_error -Verify 1
```

Our little test server running on port  `12345`  will serve  `Hello World!`  if all goes ok. Test it out as follows:

```bash
$ echo -e 'GET /test.txt HTTP/1.1\r\n\r\n' | \  
  openssl s_client \  
  -cert <(kubectl -n test get secret test-client-tls -o jsonpath='{.data.tls\.crt}' | base64 -d) \  
  -key <(kubectl -n test get secret test-client-tls -o jsonpath='{.data.tls\.key}' | base64 -d) \  
  -CAfile <(kubectl -n test get secret test-client-tls -o jsonpath='{.data.ca\.crt}' | base64 -d) \  
  -connect localhost:12345 -quiet
```

If all went well you should get the following output from the command above:

```html
Can't use SSL_get_servername  
depth=1 O = ACME Inc., OU = Widgets, CN = test-ca  
verify return:1  
depth=0  
verify return:1  
HTTP/1.0 200 ok  
Content-type: text/plain  
  
Hello World!  
read:errno=0
```

Great! Our certificates seems to work well. Let’s create an ingress with a simple echo service using our new shiny CA issuer.

## Echo Server Setup with CA Signed Certificate

Let’s try our setup with a simple echo server using Ingress. When using minikube be sure to enable ingress:

```bash
$ minikube addons enable ingress
```

Create a file called  `echo-server.yaml`  with the following:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    app: echo  
  name: echo  
  namespace: test  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: echo  
  template:  
    metadata:  
      labels:  
        app: echo  
    spec:  
      containers:  
      - name: echo  
        image: fdeantoni/echo-server  
        imagePullPolicy: Always  
        ports:  
        - containerPort: 9000  
        readinessProbe:  
          httpGet:  
            path: /  
            port: 9000  
          initialDelaySeconds: 5  
          periodSeconds: 5  
          successThreshold: 1  
---  
apiVersion: v1  
kind: Service  
metadata:  
    name: echo-service  
    namespace: test  
spec:  
    selector:  
      app: echo  
    ports:  
    - name: http  
      protocol: TCP  
      port: 9000  
      targetPort: 9000  
---  
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
    name: echo-ingress  
    namespace: test  
    annotations:  
      cert-manager.io/issuer: test-ca-issuer  
spec:  
    rules:  
    - http:  
        paths:  
        - path: /test  
          pathType: Prefix  
          backend:  
            service:  
              name: echo-service  
              port:  
                number: 9000  
  tls:   
  - hosts:  
    - echo.info  
    secretName: echo-cert
```

Create it:

```bash
$ kubectl create -f echo-server.yaml
```

Now your echo server should be up and running with its own SSL certificate signed by our CA. When using minikube be sure to add the  `echo.info`  to your hosts file first as follows:

```bash
127.0.0.1    echo.info
```

After doing so, enable the minkube tunnel so the ingress services can be reached:

```bash
$ minikube tunnel
```
Now test it:

```bash
$ curl --cacert <(kubectl -n test get secret echo-server-cert -o jsonpath='{.data.ca\.crt}' | base64 -d) https://echo.info/test  
{"source":"172.17.0.7:42246","method":"GET","headers":[["host","echo.info"],["x-request-id","6e0035387cfa6be8c53a3e03e73e9f23"],["x-real-ip","172.17.0.1"],["x-forwarded-for","172.17.0.1"],["x-forwarded-host","echo.info"],["x-forwarded-port","443"],["x-forwarded-proto","https"],["x-forwarded-scheme","https"],["x-scheme","https"],["user-agent","curl/7.79.1"],["accept","*/*"]],"path":"/test","server":"echo-6885c7cfdc-8phts"}
```

Nice! Our echo service works with our CA signed certificate.

