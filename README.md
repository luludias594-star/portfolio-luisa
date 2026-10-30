# Portfólio — Luísa Dias | Azure + Containers + CI/CD

Site estático responsivo servido por Nginx em container Docker, com infraestrutura provisionada em Bicep e pipeline no Azure DevOps.

## Arquitetura

1. GitHub guarda o código-fonte.
2. Azure DevOps Pipeline executa o deploy quando há push na branch `main`.
3. Bicep provisiona Azure Container Registry (ACR), Log Analytics, Container Apps Environment, identidade gerenciada e Azure Container App.
4. A pipeline constrói a imagem Docker, publica no ACR e atualiza o Container App.
5. O Azure Container Apps disponibiliza o portfólio por HTTPS.

## Estrutura

```text
portfolio-luisa/
├── index.html
├── style.css
├── script.js
├── Dockerfile
├── nginx.conf
├── .dockerignore
├── azure-pipelines.yml
└── infra/
    └── main.bicep
```

## Testar localmente

Pré-requisito: Docker instalado e em execução.

```bash
docker build -t luisa-portfolio:local .
docker run --rm -p 8080:8080 luisa-portfolio:local
```

Acesse `http://localhost:8080`. Health check: `http://localhost:8080/health`.

## Preparar o Azure

1. Crie uma Azure Resource Manager service connection no projeto Azure DevOps e dê a ela o nome `AZURE_SERVICE_CONNECTION` (ou altere a variável no YAML).
2. Garanta que a identidade da service connection tenha permissões para criar recursos no resource group e atribuir a função `AcrPull` à identidade gerenciada. Em ambientes corporativos, prefira uma permissão mínima revisada pelo administrador.
3. Escolha um nome globalmente único para o ACR, apenas letras e números, e substitua `SUBSTITUA_POR_NOME_UNICO` na variável `acrName` do `azure-pipelines.yml`.
4. Crie um pipeline no Azure DevOps apontando para este repositório e selecione `azure-pipelines.yml`.
5. Faça push na branch `main` para iniciar o pipeline.

A região padrão está como `brazilsouth`; confirme disponibilidade e custos dos serviços na sua assinatura antes de provisionar.

## Observações importantes

- O pipeline pressupõe que o Azure CLI, Docker e a extensão `containerapp` estejam disponíveis na imagem hospedada do agente. Se necessário, instale/atualize a extensão no início do script.
- A primeira execução cria o Container App com uma imagem pública temporária; em seguida, a pipeline publica a imagem do projeto e atualiza o app.
- O ACR está com usuário administrador desativado. A identidade da service connection precisa de `AcrPush` no ACR para executar `az acr login` e enviar a imagem. A identidade gerenciada do Container App recebe `AcrPull` para baixar a imagem.
- `minReplicas: 0` reduz consumo quando não há tráfego, mas pode causar alguns segundos de inicialização na primeira visita.
- Antes de publicar, confira o link do LinkedIn e todos os dados pessoais exibidos no `index.html`.
