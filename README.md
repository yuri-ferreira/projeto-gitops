# Projeto GitOps Kubernetes

- [Objetivos](#Objetivos)
- [Pré-requisitos](#Pré-requisitos)

1. Fork e repositório GitHub.
2. Instalar o ArgoCD no cluster local.
3. Acessar e criar um app localmente no ArgoCD
4. Acessar o front-end
5. (**EXTRA**) Customizando o manifest.yaml
6. (**EXTRA**) Criar um app com um repositório privado

---

## Objetivos

### Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.

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

## 3. Acesso e criar um app localmente no ArgoCD

Para acessar o ArgoCD é necessário primeiro fazer um port-forward, para tal execute em um terminal:

```cmd
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Com esse comando será possível acessar o ArgoCD no localhost:8080.

![Tela login ArgoCD](/imgs/argocd-login.png)

Agora, será necessário encontrar as credenciais para logar. O usuário é admin e a senha devemos executar os seguintes comandos para ter acesso a ela:

```cmd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"

[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("Output_do_comando_anterior"))
```

Com isso, é possível logar no ArgoCD.

![Tela home ArgoCD](/imgs/argocd-home.png)

Agora deve-se criar o app no ArgoCD, para isso:

Clique em "applications", e em seguida "new app".

Na tela, é necessário preencher alguns campos:

![Tela app ArgoCD](/imgs/argocd-app.png)

**Application Name:**: _online-boutique_

**Project Name:** _nome-do-projeto-criado_

**Sync Policy:** _Automatic_

**Marcar os seguintes:** _Prune Resources, Self Heal, Set Deletion Finalizer, Auto-Create Namespace_.

![Tela app ArgoCD](/imgs/argocd-app2.png)

**Repository URL:** _url do repositório_

**Revision:** _main_

**Path:** _./gitops-microservices/k8s_

**Cluster URL:** **Importante, troque para "NAME"**, e digite/selecione _in-cluster_

**Namespace:** _online-boutique_

Preenchendo essas informações, clique em "Create".

Assim, o aplicativo começará a sincronizar com o repositório.

![Tela app-status ArgoCD](/imgs/argocd-appstatus.png)

## 4. Acessar o front-end

Para isso precisamos fazer o port-forward do app,em um terminal execute:

```cmd
kubectl port-forward svc/frontend-external 80:80 -n online-boutique
```

![Tela Terminal](/imgs/terminal.png)

Com isso será possível acessar o front-end com o ip 127.0.0.1:80

![Tela app online-boutique](/imgs/site.png)

## 5. (**EXTRA**) Customizando o manifest.yaml

Há várias mudanças que podem ser feitas no manifesto, podem ser alteradas a quantidade de réplicas, alterar a versão da imagem dos containers, limite de recursos como CPU e Memória, criar um novo volume.

Para alterar alguns desses, deve-se alterar os seguintes:

Réplicas:

```kubernetes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loadgenerator
  labels:
    app: loadgenerator
spec:
  selector:
    matchLabels:
      app: loadgenerator
  replicas: 5
  template:
    metadata:
```

Foi alterado o número de réplicas atráves da mudança da linha "_replicas_" no **spec**. Neste exemplo foi alterado o número padrão de réplicas (1) para 5.

Limite CPU, Memória:

```kubernetes
 containers:
        - name: redis
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            privileged: false
            readOnlyRootFilesystem: true
          image: redis:alpine
          ports:
            - containerPort: 6379
          readinessProbe:
            periodSeconds: 5
            tcpSocket:
              port: 6379
          livenessProbe:
            periodSeconds: 5
            tcpSocket:
              port: 6379
          volumeMounts:
            - mountPath: /data
              name: redis-data
          resources:
            limits:
              memory: 512Mi
              cpu: 500m
            requests:
              cpu: 100m
              memory: 300Mi
```

Aqui em **resources**, foi alterado os valores de _requests_ e _limits_, que no padrão estavam definidos: cpuRequest(**70m**), cpuLimit(**125m**), memoryRequest(**200Mi**), memoryLimit(**256Mi**).

Essas configurações definem o quanto de _memória_ e _cpu_ um _container_ pode solicitar e utilizar no máximo.

## 6. (**EXTRA**) Criar um app com um repositório privado

Há certos momentos em que um repositório deve ser privado, nisso é feito um processo diferente para criação de um app no ArgoCD.

Para tal, deve-se:

- ### Criar um repositório privado no github.

Para isso, na página de criação do repositório selecione "Private"

![Tela github criação repositório](/imgs/github-privado.png)

- ### Agora deve-se criar uma chave SSH, para isso execute em um terminal:

```cmd
ssh-keygen -t rsa -b 4096 -C "argocd-repo-access" -f E:\chave
```

Com esse comando será criado duas chaves, uma pública e uma privada. Neste caso, não foi utilizado passphrase.

- ### Adicionar a chave pública ao repositório privado.

Clique em "_Settings_" do repositório, e siga as etapas nas imagens abaixo:

![Tela github adição chave pública no repositório](/imgs/github-addkey.png)

![Tela github adição chave pública no repositório](/imgs/github-addkey2.png)

**Importante:** Deve-se preencher a "key" com o conteúdo do arquivo da chave pública, para acessa-lo, pode-se abrir no vs code ou utilizar o comando **cat**. **Deve-se também marcar "Allow write acess"**.

- ### Adicionar a chave privada ao ArgoCD

Deve-se ter o ArgoCLI instalado em seu dispositivo. Em um terminal execute:

```cmd
argocd repo add git@github.com:SEU_USUARIO/SEU_REPOSITORIO.git --ssh-private-key-path E:\chave
```

### Pronto, agora é possível criar um app com o repositório privado.

**Importante:** Como foi utilizado via SSH, o repoURL na criação do app deve seguir o formado SSH. Por exemplo: **git@github.com:SEU_USUARIO/SEU_REPOSITORIO.git**
