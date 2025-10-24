
# Online Boutique - Deployment de cluster Kubernetes com ArgoCD

<img width="784" height="348" alt="thumbnail" src="https://github.com/user-attachments/assets/5caaf57d-d634-4ab3-bf3e-b1e40ce6287f" />

Olá! Este projeto tem o intuito de demonstrar o deployment de uma aplicação web através de um cluster Kubernetes local com práticas de GitOps, com a ferramenta ArgoCD.

Este projeto é realizado no contexto do Programa de Estágio em DevSecOps promovido pela Compass UOL | Ai/R Company.

## 🧩 1 - Instalando o Kubernetes local com minikube

### 1.1 – Requisitos mínimos

Antes de tudo, é preciso se atentar aos requisitos mínimos para a instalação e utilização do minikube:

- CPU: pelo menos 2 núcleos de CPU;
- Memória: mínimo 2 GB RAM livres; recomendado 8GB;
- Armazenamento: 20 GB ou mais;
- Conexão com a internet;
- Gerenciador de containers ou máquinas virtuais, tais como Docker, QEMu ou VirtualBox.

Neste teste, utilizaremos o Docker em sistema operacional Ubuntu 25.10.

### 1.2 – Instalando o Docker

A instalação do Docker varia entre sistemas operacionais, sejam Windows, Linux ou MacOS. Detalhes sobre instalação em diferentes SOs e distribuições Linux estão disponíveis na documentação oficial do Docker [aqui](https://docs.docker.com/engine/install/).

Para Instalar na distribuição Ubuntu, utilizando o método de pacotes apt:

>
    # Adicionando a chave GPG oficial do Docker:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
>

>
    # Adicionando o repositório às fontes apt:
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
>

>
    #Instalando a versão mais recente do Docker
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
>

Para checar se a instalação foi bem sucedida:

>
    sudo systemctl status docker
    sudo docker run hello-world
>

### 1.3 – Instalando o minikube

Para instalar o minikube na distribuição Linux (considerando arquitetura x86-64):

>
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
    sudo dpkg -i minikube_latest_amd64.deb
>

Para iniciar o cluster Kubernetes no minikube, basta rodar o comando:

>
    minikube start
>

<img width="1440" height="900" alt="screenshot_0" src="https://github.com/user-attachments/assets/243c366d-d9a1-40ce-8d27-ed7b7088d14d" />

Assim, o minikube iniciará um cluster, disponibilizando o kubectl para interagir com o mesmo ao final:

>
    minikube kubectl — get po -A
>

Em alguns sistemas, o kubectl pode não funcionar sozinho, devendo ser precedido por “minikube”. Nesse caso, pode ser atribuído um alias no bash para agilizar os comandos de interação com o cluster Kubernetes:

>
    alias kube=”minikube kubectl —”
>

Assim, o comando para verificar todos os objetos criados no cluster, por exemplo, ficará assim:

>
    kube get all
>

## 🐙 2 – Instalando o ArgoCD

O ArgoCD é uma ferramenta declarativa de entrega contínua GitOps para o Kubernetes. Ele será instalado no cluster com um namespace próprio.

>
    kube create namespace argocd
    kube apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
>

Para acessar o servidor API do ArgoCD, pode ser utilizado um port-forward:

>
    kube port-forward svc/argocd-server -n argocd 8080:443
>

<img width="1440" height="900" alt="screenshot_1" src="https://github.com/user-attachments/assets/ad433268-5eee-4e7f-9356-03b42a98c510" />

Para o primeiro acesso, é preciso informar a senha inicial auto gerada pelo ArgoCD, que pode ser visualizada com o seguinte comando:

>
    argocd admin initial-password -n argocd
>

Para usuário, utilize admin.

## ⚙️ 3 – Configurando os repositórios para deployment

Em uma prática GitOps, o código fonte da aplicação será separado do repositório de onde partirá as instruções (pelo manifesto YAML) para implantação e manutenção da aplicação.

Para este projeto, realizamos um fork do repositório microservices-demo, que contém a aplicação Online Boutique, disponibilizado pelo Google Cloud Platform para testes, podendo ser acessado [aqui](https://github.com/GoogleCloudPlatform/microservices-demo).

Em seguida, criamos o repositório gitops-microservices, com a seguinte estrutura de diretório:

>
    gitops-microservices
    |___k8s
        |___online_boutique.yaml
>

O manifest YAML “online_boutique” regerá o deployment da aplicação Online Boutique em nosso cluster Kubernetes.

No escopo deste projeto, utilizaremos o exemplo contido no manifest disponível no repositório oficial microservices-demo, que permitirá o deploy de todos os componentes necessários para rodar a aplicação, seja em frontend ou backend.

No online-boutique.yaml, codifique as instruções para o Deployment. Para título de exemplo, consulte o arquivo manifest YAML neste repositório: [`/k8s/online-boutique.yaml`](./k8s/online-boutique.yaml)

Os parâmetros podem ser alterados pelo usuário conforme a necessidade ou para outros testes (como número de réplicas, por exemplo). A estrutura pode ser localizada também no arquivo kubernetes-manifest.yaml, disponível no repositório microservices-demo/release.

## ▶️ 4 – Criando a aplicação no ArgoCD

Para fazer o deploy da aplicação:

- Acesse o ArgoCD e clique em “New App”;

<img width="1440" height="900" alt="screenshot_2" src="https://github.com/user-attachments/assets/183d1498-4c4f-408c-98aa-5d2d2672f41e" />

Na etapa *"GENERAL"*:

- Em Application name, digite online-boutique;

- Em Project, digite default;

- Em Sync Policy, é possível escolher “manual” para sincronizar manualmente, ou “auto-sync” para sincronização automática;

<img width="1440" height="900" alt="screenshot_3" src="https://github.com/user-attachments/assets/2fb234c7-f863-4e61-9605-392aec82dcbb" />

Na etapa *"SOURCE"*:

- Em Repository URL, informe o repositório da aplicação: https://github.com/seu-usuario/gitops-microservices;

- Em Revision, escolha HEAD;

- Em PATH, defina k8s;

<img width="1440" height="900" alt="screenshot_4" src="https://github.com/user-attachments/assets/91ebbdb7-98a7-4f2c-bca4-3d3dba152ee3" />

Em *"DESTINATION"*:

- Em cluster URL, informe https://kubernetes.default.svc;

- Em namespace, defina online-boutique.

Antes de iniciar o deployment, crie um namespace para a aplicação (pode ser necessário abrir outro bash para os comandos, uma vez que o port-forward no ArgoCD mantém o primeiro ocupado):

>
    minikube kubectl create namespace argocd
>

Após isso, volte ao ArgoCD e clique no botão “Create” para iniciar o deployment.

É possível acompanhar em tempo real a implantação de cada objeto no ArgoCD, como também consultar pelo bash:

>
    minikube kubectl — get all -n online-boutique
>

<img width="1440" height="900" alt="screenshot_5" src="https://github.com/user-attachments/assets/278fad3d-7826-4ec0-a55f-40bf920386fc" />

Por fim, a aplicação sincronizada deixará pendente somente o objeto do tipo LoadBalancer, por estar rodando em um cluster local. Para acessar no navegador:

>
    minikube service frontend-external –n online-boutique
>

## 👗👜 Considerações finais

<img width="1440" height="900" alt="screenshot_6" src="https://github.com/user-attachments/assets/2a6eb8d0-fcde-45e0-aada-487ae07a0cb4" />

Caso apareça o site de compras Online Boutique, parabéns! A aplicação está sincronizada com o repositório GitOps e funcionando perfeitamente em um cluster local Kubernetes via minikube.

Para atualizar parâmetros nesse cluster, tais como número de réplicas, capacidade de memória para a aplicação, etc., é necessário que isso esteja definido única e exclusivamente no arquivo manifest YAML do repositório GitOps.

Logo, caso o usuário tente alterar qualquer elemento do cluster localmente, seja com *“kubectl edit”* ou seja com *“kubectl scale —replicas=2”*, por exemplo, o ArgoCD irá sincronizar com os parâmetros definidos no repositório Git, impedindo qualquer alteração que não seja pelo manifest YAML.
