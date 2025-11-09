![Rancher](https://img.shields.io/badge/rancher-%230075A8.svg?style=for-the-badge&logo=rancher&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326CE5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-%23EF7422.svg?style=for-the-badge&logo=argo&logoColor=white)
![GitHub](https://img.shields.io/badge/github-%23181717.svg?style=for-the-badge&logo=github&logoColor=white)

# Repositório de Manifestos Kubernetes (GitOps)

Este repositório armazena os manifestos Kubernetes para a aplicação **"yourdisc-api"**. Ele funciona como a "fonte da verdade" (Source of Truth) para o deploy contínuo gerenciado pelo ArgoCD.

Este repositório é a ponta de "Entrega Contínua" (CD) de um pipeline de GitOps. As atualizações na tag da imagem do `deployment.yaml` são feitas automaticamente por um pipeline de CI/CD (como o GitHub Actions) que monitora o repositório da aplicação.

## 🔗 Repositórios Relacionados

* **Repositório da Aplicação (CI):** https://github.com/igoor1/projetoCI-CD.git
* **Imagem Docker (Build):** https://hub.docker.com/repository/docker/igoor1/your-disc/general

## Estrutura de Arquivos

```
SeuProjeto/
└── k8s
  └── deployment.yaml
  └── service.yaml
```

## Manifestos

Abaixo estão os detalhes dos recursos Kubernetes gerenciados por este repositório.

### 1. Deployment (`deployment.yaml`)

Este arquivo define o `Deployment` que gerencia os Pods da aplicação.

- **Nome:** `yourdisc-deployment`

- **Réplicas:** 2

- **Imagem:** `igoor1/your-disc:latest` (Esta tag é o alvo que será atualizado pelo pipeline de CI).

- **Porta do Container:** `8000`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yourdisc-deployment
  labels:
    app: yourdisc-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: yourdisc-api
  template:
    metadata:
      labels:
        app: yourdisc-api
    spec:
      containers:
      - name: yourdisc-api
        image: igoor1/your-disc:latest # Esta linha é atualizada pelo CI
        ports:
        - containerPort: 8000
```

### 2. Service (`service.yaml`)

Este arquivo define o `Service` que expõe os Pods internamente no cluster.

- **Nome:** `yourdisc-service`

- **Tipo:** `ClusterIP` (acessível apenas de dentro do cluster).

- **Porta do Serviço:** `8080`

- **Porta Alvo (Container):** `8000` (direciona o tráfego da porta 8080 do serviço para a porta 8000 dos pods).

```bash
apiVersion: v1
kind: Service
metadata:
  name: yourdisc-service
  labels:
    app: yourdisc-api
spec:
  type: ClusterIP
  selector:
    app: yourdisc-api
  ports:
    - name: http
      port: 8080
      targetPort: 8000
```

## Uso com ArgoCD

Para implantar esta aplicação usando o ArgoCD, crie uma nova "Application" com as seguintes especificações:

- **Application Name:** yourdisc-api (ou de sua preferência)

- **Project:** default

- **Sync Policy:** Automatic (com Prune Resources e Self Heal habilitados)

- **Repository URL:** (A URL deste repositório)

- **Revision:** HEAD

- **Path:**  k8s 

- **Cluster URL:** https://kubernetes.default.svc

- **Namespace:** default (ou o namespace de sua escolha)

![create app](https://github.com/user-attachments/assets/a5564799-37fe-4193-b183-9c1f64b1d4f4)
![create app2](https://github.com/user-attachments/assets/d4160929-4854-42ec-a373-d66bdaf0b76e)


Após o ArgoCD sincronizar a aplicação, todos os pods e services estarão rodando no seu cluster. Para acessar a api, precisamos expor a porta do serviço para a sua máquina local.

![argocd app](https://github.com/user-attachments/assets/fc9e3173-a9fa-42e1-87e5-c13ad4ec25b0)

1. Abra um novo terminal (deixe os outros rodando) e execute o comando port-forward:

```bash
kubectl port-forward svc/yourdisc-service 8080:8080
```

2. Abra seu navegador e acesse a documentação do FastAPI: `http://localhost:8080/docs`.

- Acessando Api pelo navegador:

![api/docs](https://github.com/user-attachments/assets/d9939e0e-627c-4afd-a42b-b59128745d9e)