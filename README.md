# clouddog-heartbeat-uptime-setup

Este Helm Chart implanta um monitor de "heartbeat" no Kubernetes. Ele foi projetado para ser usado com o ArgoCD, permitindo o monitoramento de múltiplos serviços internos a partir de um único Pod, sem a necessidade de expô-los publicamente.

## Estrutura do Repositório

O repositório está organizado nas seguintes pastas:

### `monitoring/`
Esta pasta contém o Helm Chart completo para o monitor de heartbeat.

**Chart.yaml:** O arquivo de definição do chart.

**values.yaml:** Contém os valores padrão (com a lista apps: [] vazia). Este arquivo é sobrescrito pelos valores injetados pelo ArgoCD.


### `templates/`
Esta pasta armazena os templates dos Helm Charts, que são os arquivos YAML parametrizados para a implantação de recursos Kubernetes.
- **deployment.yaml:** Cria o Deployment principal com um loop (range) para gerar múltiplos contêineres dentro de um único Pod.
- **secrets.yaml:** Cria automaticamente um Secret do Kubernetes para cada item na lista apps (para injetar HEARTBEAT_URL e SERVICE_URL).

## Como Usar

1.  **Adicionar ao repositorio do cliente** Clonar o repositorio padrão do heartbeat-uptime-template e adicionar a pasta `monitoring/` ao repositório do cliente que utiliza ArgoCD para deploys.

    ```
    . (repositório do cliente)
    ├── monitoring/
    │      └── Chart.yaml
    │      └── templates/
    │          └── deployment.yaml
    │          └── secrets.yaml
    │      └── values.yaml
    |      └── README.md
    ```

2.  **Atualizar o values.yaml** Atualize o apps no values.yaml para refletir ao monitoramento do cliente. Por exemplo:

    ```yaml

    replicas: 1
    apps:
      - name: nome-da-sua-aplicacao
        secretName: heartbeat-secret-nome-da-sua-aplicacao
        secretData:
          SERVICE_URL: "https://minha-aplicacao-interna.svc.cluster.local/health"
          HEARTBEAT_URL: "https://uptimerobot.com/api/heartbeat/SEU_TOKEN_AQUI"
        resources:
          requests:
            cpu: 50m
            memory: 50Mi
          limits:
            cpu: 100m
            memory: 100Mi

      - name: nome-da-sua-aplicacao2
        secretName: heartbeat-secret-nome-da-sua-aplicacao2
        secretData:
          SERVICE_URL: "https://minha-aplicacao-interna2.svc.cluster.local/health"
          HEARTBEAT_URL: "https://uptimerobot.com/api/heartbeat/SEU_TOKEN_AQUI2"
        resources:
          requests:
            cpu: 50m
            memory: 50Mi
          limits:
            cpu: 100m
            memory: 100Mi

    ```

  Repita o bloco para cada aplicação que deseja monitorar.

  3.  **Crie o aplication no Argo** Crie um novo Application no ArgoCD apontando para o repositório do cliente e o caminho `monitoring/`.
      ```yaml
      apiVersion: argoproj.io/v1alpha1
      kind: Application
      metadata:
        name: heartbeat-uptime
        namespace: argocd
      spec:
        project: monitoring
        source:
          repoURL: 'https://seu-repositorio-git.com/seu-projeto.git'
          targetRevision: HEAD
          helm:
          values: |
            replicas: 1
        destination:
          server: https://kubernetes.default.svc
          namespace: heartbeat
        syncPolicy:
          automated:
            prune: true
            selfHeal: true
          syncOptions:
            - CreateNamespace=true
      ```
