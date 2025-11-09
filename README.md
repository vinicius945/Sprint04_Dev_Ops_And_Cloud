# 🚀 Challenge 2025 - Sprint 4: DevOps & Cloud Computing

## 👥 Grupo LTAKN

* Enzo Prado Soddano — RM557937
* Lucas Resende Lima — RM556564
* Vinícius Prates Altafini — RM559183

---

## 🔗 Links de Entrega (Requisito 0)

* **Aplicação (Azure Web App):** `https://webapp-challenge-945-sprint4.azurewebsites.net`
* **Vídeo de Demonstração (YouTube):** 
* **Repositório (GitHub):** `https://github.com/vinicius945/Sprint04_Dev_Ops_And_Cloud`
* **Projeto (Azure DevOps):** `https://dev.azure.com/RM5591830661/Sprint%204%20%E2%80%93%20Azure%20DevOps`

---

## 📌 1. Descrição da Solução

Este projeto é a entrega da Sprint 4 da disciplina de DevOps Tools & Cloud Computing. O objetivo foi pegar a aplicação de Gestão de Frota (Mottu) desenvolvida em Java/Spring Boot, "containerizar" a aplicação com Docker, e construir uma pipeline de CI/CD completa no Azure DevOps.

A pipeline (`azure-pipelines.yml`) automatiza todo o processo:

1.  **CI (Integração Contínua):** Um `push` na branch `main` dispara o build, a execução de 18 testes (unitários e de integração), a criação da imagem Docker e a publicação desta imagem no Azure Container Registry (ACR).
2.  **CD (Entrega Contínua):** Após o sucesso do CI, a pipeline automaticamente faz o deploy da nova imagem do ACR para o Azure Web App, publicando o site sem intervenção manual.

### Stack de Tecnologias Utilizadas

* **Aplicação:** Java 17, Spring Boot 3.3.4, Spring MVC, Spring Security, Spring Data JPA
* **Banco de Dados:** Azure SQL Server (com migrações Flyway)
* **Build:** Maven
* **Containerização:** Docker
* **Cloud (PaaS):** Azure Web App (para Contêineres), Azure Container Registry (ACR)
* **CI/CD:** Azure DevOps Pipelines (YAML)

---

## 🗺️ 2. Diagrama da Arquitetura (Requisito 2)

Abaixo está o fluxo de CI/CD implementado, conforme solicitado:

![Arquitetura](https://github.com/user-attachments/assets/a96ab9e8-f1fe-45e9-b089-db72862ac11a)


---

## 📋 3. Detalhamento dos Componentes 

| Componente | Tipo | Tecnologia/Ferramenta | Descrição Funcional |
| :--- | :--- | :--- | :--- |
| `Sprint04_Dev_Ops_And_Cloud` | Repositório de Código | GitHub | Onde o código-fonte está versionado. |
| `main` (branch) | SCM | GitHub | Branch principal que dispara o CI (conforme Requisito 7-II, adaptado de `master` para `main`). |
| `azure-pipelines.yml` | Pipeline (CI) | Azure DevOps Pipelines | **(CI)** Compila o código Java, executa os 18 testes e publica o artefato (`.jar`). |
| `Dockerfile` | Containerização | Docker | Cria a imagem Docker da aplicação Java (Requisito 7-VII). |
| `acrchallenge945sprint4` | Registro de Imagem | Azure Container Registry | Armazena as imagens Docker prontas para o deploy. |
| `azure-pipelines.yml` | Pipeline (CD) | Azure DevOps Pipelines | **(CD)** Disparado após o CI, faz o deploy automático da nova imagem do ACR para o Web App. |
| `webapp-challenge-945-sprint4` | Hospedagem (PaaS) | Azure Web App (Containers) | Executa o contêiner Docker e expõe a aplicação na web. |
| `sqlLTAKN-sprint4` | Banco de Dados (PaaS) | Azure SQL | Armazena os dados de pátios, motos e usuários. |
| Variáveis de Ambiente | Segurança | Azure DevOps (Secrets) & Azure Web App (Configuration) | Protege as credenciais do banco de dados (Requisito 7-III). |

---

## 💻 Como Rodar Localmente (com Docker)

1.  Inicie o **Docker Desktop**.
2.  Certifique-se de que seu IP local está liberado no firewall do **Azure SQL Server** (`sqlserver-challenge-945-sprint4`).
3.  Navegue até a pasta do projeto Java (`Challenge_Java_2025_Sprint4-main`).
4.  **Construa a imagem:**
    ```bash
    docker build -t gestaofrota-local:latest .
    ```
5.  **Rode a imagem**, passando as variáveis de ambiente do banco (use `cmd` ou `PowerShell`):

    *Se for PowerShell:*
    ```powershell
    docker run -d `
      -p 8080:8080 `
      --name gestaofrota-container `
      -e SPRING_DATASOURCE_URL="jdbc:sqlserver://sqlserver-challenge-945-sprint4.database.windows.net:1433;database=sqlLTAKN-sprint4;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;" `
      -e SPRING_DATASOURCE_USERNAME="adminSprint4" `
      -e SPRING_DATASOURCE_PASSWORD="ChallengeS4@2025" `
      -e SPRING_DATASOURCE_DRIVER_CLASS_NAME="com.microsoft.sqlserver.jdbc.SQLServerDriver" `
      gestaofrota-local:latest
    ```

6.  Acesse: `http://localhost:8080`

---

7. Criação do grupo de recursos mias banco de dados, web app e acr

```bash
# =============================================================
# Scripts de Criação da Infraestrutura (Azure CLI)
# Sprint 4 - DevOps Tools & Cloud Computing
# =============================================================

# --- 1. Definição de Variáveis ---
# (Todos os nomes de recursos usados no projeto)

RESOURCE_GROUP="rg-challenge-sprint4"
LOCATION="eastus2"

# SQL Server e Database
SQL_SERVER_NAME="sqlserver-challenge-945-sprint4"
SQL_DATABASE_NAME="sqlLTAKN-sprint4"
ADMIN_USER="adminSprint4"
ADMIN_PASSWORD="ChallengeS4@2025"

# App Service (Plano e Web App)
APPSERVICE_PLAN_NAME="plan-challenge-sprint4"
WEBAPP_NAME="webapp-challenge-945-sprint4"

# Container Registry (ACR) e Imagem Docker
ACR_NAME="acrchallenge945sprint4"
ACR_LOGIN_SERVER="acrchallenge945sprint4.azurecr.io"
IMAGE_NAME="gestaofrota-challenge" # Nome da imagem que a pipeline vai criar


# --- 2. Criação dos Recursos Principais ---

# Criar o Grupo de Recursos (Resource Group)
echo "Criando Grupo de Recursos..."
az group create --name $RESOURCE_GROUP --location $LOCATION

# Criar o Servidor SQL
echo "Criando Servidor SQL..."
az sql server create \
--name $SQL_SERVER_NAME \
--resource-group $RESOURCE_GROUP \
--location $LOCATION \
--admin-user $ADMIN_USER \
--admin-password $ADMIN_PASSWORD

# Configurar a Regra de Firewall do SQL (Permite acesso de serviços do Azure)
echo "Configurando Firewall do SQL..."
az sql server firewall-rule create \
--resource-group $RESOURCE_GROUP \
--server $SQL_SERVER_NAME \
--name AllowAzureServices \
--start-ip-address 0.0.0.0 \
--end-ip-address 0.0.0.0

# Criar o Banco de Dados SQL
echo "Criando Banco de Dados..."
az sql db create \
--resource-group $RESOURCE_GROUP \
--server $SQL_SERVER_NAME \
--name $SQL_DATABASE_NAME \
--service-objective S0

# Criar o Plano de Serviço (SKU B1, Linux)
echo "Criando Plano de Serviço..."
az appservice plan create \
--name $APPSERVICE_PLAN_NAME \
--resource-group $RESOURCE_GROUP \
--sku B1 \
--is-linux

# Criar o Azure Container Registry (ACR)
echo "Criando Container Registry (ACR)..."
az acr create \
--name $ACR_NAME \
--resource-group $RESOURCE_GROUP \
--location $LOCATION \
--sku Basic \
--admin-enabled true

# Criar o Web App para CONTÊINERES (Requisito 7-VII)
# (Este comando já aponta para a imagem que a pipeline irá criar)
echo "Criando Web App para Contêineres..."
az webapp create \
--name $WEBAPP_NAME \
--resource-group $RESOURCE_GROUP \
--plan $APPSERVICE_PLAN_NAME \
--deployment-container-image-name "$ACR_LOGIN_SERVER/$IMAGE_NAME:latest"


# --- 3. Configuração das Variáveis de Ambiente (App Settings) ---
# (Isso protege as senhas, conforme Requisito 7-III)

echo "Configurando Variáveis de Ambiente do Web App..."
az webapp config appsettings set \
--resource-group $RESOURCE_GROUP \
--name $WEBAPP_NAME \
--settings \
SPRING_DATASOURCE_URL="jdbc:sqlserver://sqlserver-challenge-945-sprint4.database.windows.net:1433;database=sqlLTAKN-sprint4;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;" \
SPRING_DATASOURCE_USERNAME="adminSprint4" \
SPRING_DATASOURCE_PASSWORD="ChallengeS4@2025" \
SPRING_DATASOURCE_DRIVER_CLASS_NAME="com.microsoft.sqlserver.jdbc.SQLServerDriver"

echo "✅ Infraestrutura concluída!"

```

## 🔑 Usuários para Teste

O `DataInitializer` cria dois usuários no primeiro deploy:
* **Usuário:** `admin` / **Senha:** `adminpass` (Role: ADMIN)
