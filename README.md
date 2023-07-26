# Login no Kubernetes

## APOIO

- [Kubernetes Doc - Certificates and Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
- [📺 Fabricio Veronez - Como criar usuários no cluster Kubernetes](https://youtu.be/WQx_aFVFXh8)
- [📺 vNugget - How To Create A Normal User In Kubernetes](https://youtu.be/r_fSTn_Ixuk)
- [📺 InfraHQ - Creating Users in Kubernetes](https://youtu.be/dnKVZR4eK7Q)

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
code k8s-csr.yaml;
```

### Configuração do *k8s-csr.yaml*

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: saulo
spec:
  request: [CERTIFICATE REQUEST em base64]
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
kubectl certificate approve saulo;
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
kubectl get csr saulo -o jsonpath='{.status.certificate}'| base64 --decode > saulo.crt;
```

## config

Copiar o *config* local para o diretorio atual para usa-lo como base

```sh
cp ~/.kube/config ./kubeconfig-saulo.yaml;
code kubeconfig-saulo.yaml;
```

Tentar listar os nodes usando o kubeconfig criado.

```sh
kubectl get nodes --kubeconfig kubeconfig-saulo.yaml;
```

```yaml
...
contexts:
- context:
    cluster: docker-desktop
    user: saulo
  name: saulo@docker-desktop
current-context: saulo@docker-desktop
kind: Config
preferences: {}
users:
- name: saulo
  user:
    client-certificate-data: ./saulo.crt
    client-key-data: ./saulo.key
```

![-](/images/image.png)

> Criou o usuario mas ele ainda não tem nenhuma permisão

## Add permisões para o usuario usando o 👉[*RBAC*](https://youtu.be/1cv94XguLyg)

### criar o *rbac.yaml*

```sh
code rbac.yaml;
```

### modelo

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ecomerce-user
  namespace: ecomercer
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch", "create", "update", "delete", "patch"]
  - apiGroups: ["apps"]
    resources: ["replicasets", "deployments"]
    verbs: ["get", "list", "watch", "create", "update", "delete", "patch"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ecomerce-user-bind
  namespace: ecomercer
subjects:
- kind: User
  name: saulo
roleRef:
  kind: Role
  name: ecomerce-user
  apiGroup: bac.authorization.k8s.io
```

Aplica o *rbac*

```sh
kubectl apply -f rbac.yaml;
```

Testar como o comando

```sh
kubectl get pods -n ecomerce --kubeconfig kubeconfig-saulo.yaml;
```

> No video o Fabrício faz um teste realizadno um deploy usando as config criada expecificando o namespace (-n)

```sh
kubectl apply -f deployment.yaml -n ecomerce --kubeconfig kubeconfig-saulo.yaml;
```
