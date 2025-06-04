+++ 
draft = true
date = 2025-06-04T14:34:34-03:00
title = "Como configurar o ArgoCD localmente para desenvolvimento"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

Argo CD é uma ferramenta que ajuda os desenvolvedores a gerenciar e implantar facilmente seus aplicativos em um cluster Kubernetes. Funciona comparando constantemente o estado desejado do aplicativo, conforme definido em um repositório Git, com o estado atual do cluster. Ele faz automaticamente quaisquer atualizações necessárias e pode reverter as alterações, se necessário. Ele também tem uma interface web amigável e uma interface de linha de comando para gerenciar implantações.


Minikube é uma ferramenta que permite executar um cluster Kubernetes em uma única máquina, o que pode ser útil para fins de teste e desenvolvimento. (É importante notar que o Minikube é apenas uma ferramenta para desenvolvimento e teste local, não para o ambiente de produção)


Você pode usar o Minikube para executar o Argo CD. Para usar o Argo CD com o Minikube, primeiro você precisaria instalar e iniciar o Minikube e, em seguida, instalar o Argo CD no cluster Minikube. Uma vez que o Argo CD esteja em execução, você pode usá-lo para implantar e gerenciar aplicativos no cluster Minikube da mesma forma que faria com um cluster de produção.


Nesta história, vou demonstrar como instalar o argocd no minikube e implantar um aplicativo de amostra.

### Pré-requisito
Você precisa ter o minikube instalado.

### Instalando o ArgoCD no minikube
Começaremos com o lançamento do minikube. Existem várias opções de driver que você pode usar para iniciar um cluster minikube (virtualbox, docker, hyperv). Eu usei o docker como driver.

``` 
minikube start --driver docker
```

1. Assim como outras ferramentas Kubernetes, o ArgoCD requer um namespace com seu nome. Portanto, criaremos um namespace para argocd.

```
kubectl create ns argocd
```


2. Aplicaremos o arquivo de instalação do manifesto ArgoCD do repositório github do ArgoCD

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. Vamos verificar a instalação obtendo todos os objetos no namespace ArgoCD.

```
kubectl get all -n argocd
```


### Interface do Usuário ArgoCD

O ArgoCD tem uma boa interface de usuário para gerenciamento. Ele tem um painel que fornece uma representação visual do estado atual de seus aplicativos, sua integridade e os recursos em que são implantados. A interface do usuário foi projetada para ser intuitiva e fácil de usar, permitindo que você gerencie seus aplicativos e monitore seu progresso com facilidade.

1. Para acessar a GUI da web do ArgoCD, precisamos fazer um encaminhamento de porta. Para isso, usaremos o serviço argocd-server (Mas certifique-se de que os pods estejam em um estado de execução antes de executar este comando).

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

<br>

2. Agora podemos ir para um navegador e abrir localhost:8080

Você verá um aviso de privacidade. Ignore o aviso, clique em Avançado e, em seguida, em Prosseguir para o host local (inseguro) para prosseguir para a interface gráfica. (As configurações do seu navegador podem apresentar uma opção diferente para continuar).

<br>

3. Para usar a interface do ArgoCD, precisamos inserir nossas credenciais. O nome de usuário padrão é admin, então podemos inseri-lo imediatamente, mas precisaremos obter a senha inicial do ArgoCD através do terminal do Minikube.

A senha inicial é mantida como segredo no namespace argocd; portanto, usaremos a consulta jsonpath para recuperar o segredo do namespace argocd. Também precisamos decodificá-lo com base64. Para fazer as duas operações, basta abrir um novo terminal e inserir o seguinte código para fazer o truque para você. (Não feche a primeira janela do terminal, pois o encaminhamento da porta ainda está ativo)

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

<br>

4. Copie a senha, volte para o seu navegador e digite-a como senha (o nome de usuário é admin). Você deve estar na interface da GUI.