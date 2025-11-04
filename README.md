# Kyverno Overview

![Arquitetura do Kyverno](./images/kyverno-architecture.png)

## 📘 Introdução

**Kyverno** é uma engine nativa de Kubernetes para políticas como código (_Policy-as-Code_).  
Suas políticas são definidas em YAML, sem a necessidade de linguagens externas como Rego (OPA Gatekeeper).

Projetado para operadores de plataforma, equipes **DevSecOps** e administradores de clusters, o Kyverno facilita a implementação de **segurança**, **conformidade** e **governança** em ambientes Kubernetes.

Kyverno atua como um  
> [controlador de admissão dinâmica](https://kyverno.io/docs/introduction/admission-controllers/)

Recebendo via webhooks de validação e mutação do **kube-apiserver** para aplicar, mutar ou rejeitar recursos com base nas políticas configuradas.

---

## ⚙️ Componentes

Kyverno realiza três funções principais:

### ✅ Validation (Validação)
Valida objetos antes de serem criados ou alterados, rejeitando recursos que não atendem às regras definidas.

### 🔄 Mutation (Mutação)
Modifica objetos antes de sua criação.  
Ex.: adicionar labels, annotations, atributos, etc.

### ⚡ Generation (Geração)
Gera automaticamente novos recursos ao detectar eventos do cluster.  
Ex.: criar um `ConfigMap` quando um `Namespace` for criado.

---

## 🧩 Admission Controller

Admission controllers validam/modificam requisições antes do armazenamento no etcd.

Existem dois tipos principais usados pelo Kyverno:

| Tipo | Função |
|-----|-------|
| **MutatingAdmissionWebhook** | Altera o objeto antes de ser persistido |
| **ValidatingAdmissionWebhook** | Permite ou rejeita a requisição |

Fluxo Kubernetes (alto nível):

![Kubernetes Admission](./images/kubernetes-admission-controllers.png)

---

# 🚀 Instalação

## ✅ Requisitos
- Kubernetes cluster
- Helm 3+

---

## 📦 Instalação via Helm (HA + RBAC Extended)

### 1) Adicione o repositório Helm

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
```

### 2) Instale com valores personalizados

```bash
helm upgrade --install kyverno kyverno/kyverno \
  --namespace kyverno --create-namespace \
  -f kyverno-values.yaml
```

---

## 📄 Exemplo de `kyverno-values.yaml`

```yaml
features:
  policyExceptions:
    enabled: true
    namespace: kyverno

admissionController:
  replicas: 3
  rbac:
    coreClusterRole:
      extraResources:
        - apiGroups: [""]
          resources: ["namespaces"]
          verbs: ["update", "patch", "get", "list", "watch"]
        - apiGroups: [""]
          resources: ["secrets"]
          verbs: ["get", "list", "watch", "create", "update", "delete"]

backgroundController:
  replicas: 3
  rbac:
    coreClusterRole:
      extraResources:
        - apiGroups: ["networking.k8s.io"]
          resources: ["ingresses", "ingressclasses", "networkpolicies"]
          verbs: ["create", "update", "patch", "delete"]
        - apiGroups: ["rbac.authorization.k8s.io"]
          resources: ["rolebindings", "roles"]
          verbs: ["create", "update", "patch", "delete"]
        - apiGroups: [""]
          resources: ["configmaps", "resourcequotas", "limitranges"]
          verbs: ["create", "update", "patch", "delete"]
        - apiGroups: [""]
          resources: ["namespaces"]
          verbs: ["update", "patch", "get", "list", "watch"]
        - apiGroups: [""]
          resources: ["secrets"]
          verbs: ["get", "list", "watch", "create", "update", "delete"]

cleanupController:
  replicas: 3

reportsController:
  replicas: 3
```

---

# 🛠️ Kyverno CLI

A CLI permite validar, testar e aplicar políticas localmente.

## ✅ Instalação

> https://kyverno.io/docs/kyverno-cli/install/

Exemplo (Linux):

```bash
curl -LO https://github.com/kyverno/kyverno/releases/latest/download/kyverno-cli_linux_x86_64.tar.gz
tar -zxvf kyverno-cli_linux_x86_64.tar.gz
sudo mv kyverno /usr/local/bin
```

Verifique:

```bash
kyverno version
```

---

## ▶️ Uso básico da CLI

### Testar políticas contra recursos

```bash
kyverno apply -t --resources app_k8s.yaml -f policies.yaml
```

### Validar políticas aplicadas em um cluster
```bash
kyverno apply policies.yaml --cluster
```

### Testar políticas a manifestos via Kustomize
```bash
kustomize build policies/ | kyverno apply -t --resources app_k8s -
```

---

# 📚 Referências

- https://kyverno.io/docs/exceptions/
- https://github.com/kyverno/policies/tree/main
- https://kyverno.io/docs/kyverno-cli/install/
- https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/
- https://www.sysdig.com/blog/kubernetes-admission-controllers