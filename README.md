# Projeto GitOps Kubernetes

- [Objetivos](#Objetivos)
- [Pré-requisitos](#Pré-requisitos)

---

## Objetivos

## Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.

## Pré-requisitos

- Rancher Desktop instalado (com Kubernetes habilitado);
- Kubectl configurado (kubectl get nodes funcionando);
- ArgoCD instalado no cluster;
- Conta no GitHub com repositório público;
- Git instalado;
- Docker funcionando localmente.

## 1. Fork e repositório GitHub.

Para fazer um fork do repositório do Online Boutique, deve-se abrir o repositório e clicar no botão abaixo:

Após, deve-se baixar o arquivo "kubernetes-manifests.yaml" do repositório criado no Fork.

Agora deve-se criar a estrutura solicitada previamente no repositório. Para isso, com o repositório aberto em um terminal digite os seguintes comandos:

```cmd
mkdir gitops-microservices
cd gitops-microservices

mkdir k8s
cd k8s
```

Na pasta k8s, será adicionado o arquivo baixado anteriormente.

## 2. Instalar o ArgoCD no cluster local.

Para instalar o ArgoCD primeiro deve-se criar um namespace exclusivo para ele. Em um terminal, execute:

```cmd
kubectl create namespace argocd
```

Em seguida, é necessário aplicar os manifests de instalação do ArgoCD. Em um terminal, execute:

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 3. Acessar e criar um app localmente no ArgoCD

Para acessar o ArgoCD é necessário primeiro fazer um port-forward, para tal execute em um terminal:

```cmd
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Com esse comando será possível acessar o ArgoCD no localhost:8080.

Agora, será necessário encontrar as credenciais para logar. O usuário é admin e a senha devemos executar os seguintes comandos para ter acesso a ela:

```cmd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"

[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("Output_do_comando_anterior"))
```

Com isso, é possível logar no ArgoCD.

## 4.
