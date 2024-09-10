# Cloudflare

To use Cloudflare, you may use one of two types of tokens.  **API Tokens**  allow application-scoped keys bound to specific zones and permissions, while  **API Keys**  are globally-scoped keys that carry the same permissions as your account.

**API Tokens**  are recommended for higher security, since they have more restrictive permissions and are more easily revocable.

## API Tokens[](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/#api-tokens)

Tokens can be created at  **User Profile > API Tokens > API Tokens**. The following settings are recommended:

-   Permissions:
    -   `Zone - DNS - Edit`
    -   `Zone - Zone - Read`
-   Zone Resources:
    -   `Include - All Zones`

To create a new  `Issuer`, first make a Kubernetes secret containing your new API token:

```yaml
apiVersion: v1

kind: Secret

metadata:

  name: cloudflare-api-token-secret

type: Opaque

stringData:

  api-token: <API Token>
```
Then in your  `Issuer`  manifest:

```yaml
apiVersion: cert-manager.io/v1

kind: Issuer

metadata:

  name: example-issuer

spec:

  acme:

  ...

  solvers:

  -  dns01:

  cloudflare:

  apiTokenSecretRef:

  name: cloudflare-api-token-secret

  key: api-token
  ```

# Certificate resource

> **apiVersion:**  cert-manager.io/v1  
> **kind:**  Certificate

In cert-manager, the  `Certificate`  resource represents a human readable definition of a certificate request. cert-manager uses this input to generate a private key and  [`CertificateRequest`](https://cert-manager.io/docs/usage/certificaterequest/)  resource in order to obtain a signed certificate from an  [`Issuer`](https://cert-manager.io/docs/configuration/)  or  [`ClusterIssuer`](https://cert-manager.io/docs/configuration/). The signed certificate and private key are then stored in the specified  `Secret`  resource. cert-manager will ensure that the certificate is  [auto-renewed before it expires](https://cert-manager.io/docs/usage/certificate/#renewal-reissuance)  and  [re-issued if requested](https://cert-manager.io/docs/usage/certificate/#non-renewal-reissuance).

In order to issue any certificates, you'll need to configure an  [`Issuer`](https://cert-manager.io/docs/configuration/)  or  [`ClusterIssuer`](https://cert-manager.io/docs/configuration/)  resource first.

## Creating Certificate Resources[](https://cert-manager.io/docs/usage/certificate/#creating-certificate-resources)

A  `Certificate`  resource specifies fields that are used to generate certificate signing requests which are then fulfilled by the issuer type you have referenced.  `Certificates`  specify which issuer they want to obtain the certificate from by specifying the  `certificate.spec.issuerRef`  field.

A  `Certificate`  resource, for the  `example.com`  and  `www.example.com`  DNS names,  `spiffe://cluster.local/ns/sandbox/sa/example`  URI Subject Alternative Name, that is valid for 90 days and renews 15 days before expiry is below. It contains an exhaustive list of all options a  `Certificate`  resource may have however only a subset of fields are required as labelled.

```yaml
apiVersion: cert-manager.io/v1

kind: Certificate

metadata:

  name: example-com

  namespace: sandbox

spec:

  # Secret names are always required.

  secretName: example-com-tls

  # secretTemplate is optional. If set, these annotations and labels will be

  # copied to the Secret named example-com-tls. These labels and annotations will

  # be re-reconciled if the Certificate's secretTemplate changes. secretTemplate

  # is also enforced, so relevant label and annotation changes on the Secret by a

  # third party will be overwriten by cert-manager to match the secretTemplate.

  secretTemplate:

  annotations:

  my-secret-annotation-1:  "foo"

  my-secret-annotation-2:  "bar"

  labels:

  my-secret-label: foo

  privateKey:

  algorithm: RSA

  encoding: PKCS1

  size:  2048

  # keystores allows adding additional output formats. This is an example for reference only.

  keystores:

  pkcs12:

  create:  true

  passwordSecretRef:

  name: example-com-tls-keystore

  key: password

  profile: Modern2023

  duration: 2160h # 90d

  renewBefore: 360h # 15d

  isCA:  false

  usages:

  - server auth

  - client auth

  subject:

  organizations:

  - cert-manager

  # Avoid using commonName for DNS names in end-entity (leaf) certificates. Unless you have a specific

  # need for it in your environment, use dnsNames exclusively to avoid issues with commonName.

  # Usually, commonName is used to give human-readable names to CA certificates and can be avoided for

  # other certificates.

  commonName: example.com

  # The literalSubject field is exclusive with subject and commonName. It allows

  # specifying the subject directly as a string. This is useful for when the order

  # of the subject fields is important or when the subject contains special types

  # which can be specified by their OID.

  #

  # literalSubject: "O=jetstack, CN=example.com, 2.5.4.42=John, 2.5.4.4=Doe"

  # At least one of commonName (possibly through literalSubject), dnsNames, uris, emailAddresses, ipAddresses or otherNames is required.

  dnsNames:

  - example.com

  - www.example.com

  uris:

  - spiffe://cluster.local/ns/sandbox/sa/example

  emailAddresses:

  - john.doe@cert-manager.io

  ipAddresses:

  - 192.168.0.5

  # Needs cert-manager 1.14+ and "OtherNames" feature flag

  otherNames:

  # Should only supply oid of ut8 valued types

  -  oid: 1.3.6.1.4.1.311.20.2.3 # User Principal Name "OID"

  utf8Value: upn@example.local

  # Issuer references are always required.

  issuerRef:

  name: ca-issuer

  # We can reference ClusterIssuers by changing the kind here.

  # The default value is Issuer (i.e. a locally namespaced Issuer)

  kind: Issuer

  # This is optional since cert-manager will default to this value however

  # if you are using an external issuer, change this to that issuer group.

  group: cert-manager.io
  ```

## Example wildcard certificate:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: cert-emisia-wild
  namespace: gateway
spec:
  issuerRef:
    name: cloudflare-cluster-issuer
    kind: ClusterIssuer
    group: cert-manager.io
  dnsNames:
    - emisia.net
    - "*.emisia.net"
  secretName: cert-emisia-wild

  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048

  duration: 2160h # 90d
  renewBefore: 360h # 15d

  isCA: false
  usages:
    - server auth
    - client auth

  subject:
    organizations:
      - cert-manager
```

## Example regular certificate:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: cert-emisia
  namespace: gateway
spec:
  issuerRef:
    name: cloudflare-cluster-issuer
    kind: ClusterIssuer
    group: cert-manager.io
  dnsNames:
    - emisia.net
    - argocd.emisia.net
  secretName: cert-emisia

  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048

  duration: 2160h # 90d
  renewBefore: 360h # 15d

  isCA: false
  usages:
    - server auth
    - client auth

  subject:
    organizations:
      - cert-manager
```