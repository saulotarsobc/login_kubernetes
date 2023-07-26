# Login no Kubernetes

## APOIO

- [Kubernetes Doc - Certificates and Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
- [Fabricio Veronez - Como criar usuários no cluster Kubernetes](https://youtu.be/WQx_aFVFXh8)

## Criar o namespace

```sh
kubectl create namespace ecomerce;
```

```sh
kubectl get namespace;
```

## Criar a **PRIVATE KEY**

```sh
openssl genrsa -out saulo.key 2048;
```

```sh
cat saulo.key;
```

## Criar o **CERTIFICATE REQUEST**

```sh
openssl req -new -key saulo.key -subj "/CN=saulo" -out saulo.csr;
```

```sh
cat saulo.csr;
```

## Converte o **CERTIFICATE REQUEST** para *base64*

```sh
cat saulo.csr | base64 | tr -d "\n";
```

## Criar o *manifesto*

```sh
code k9s-csr.yaml;
```

### Configuração do *k9s-csr.yaml*

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: saulo
spec:
  resquest: [CERTIFICATE REQUEST em base64]
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
```

### Apply

```sh
kubectl apply -f k8s-csr.yaml;
```

Vai dar msg de certificado **criado**

Verificar se foi criado. Vai estar como status(CONDITION) de *Peddind*

```sh
kubectl get csr;
```

### Aprovar o certificado

```sh
kubectl certificate appove saulo;
```

Vai dar msg de certificado **aprovado**

Verificar se status(CONDITION). Vai estar de *Approved,Ussued*

```sh
kubectl get csr;
```

## Pegar o *.CRT* em base64 e decodifcar

```sh
kubectl get csr saulo -o yaml;
```

```sh
echo "[CRT em base64]" | base64 --decode > saulo.crt;
```

Ou simplesmente rodar... (será que funciona msm? rsrs)

```sh
kubectl get csr saulo -o jsonpath='{.status.certificate}'| base64 -d > saulo.crt
```

