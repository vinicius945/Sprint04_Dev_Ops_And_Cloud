# 🚀 Challenge 2025 - Sprint 4: DevOps & Cloud Computing

## 👥 Grupo LTAKN

* Enzo Prado Soddano — RM557937
* Lucas Resende Lima — RM556564
* Vinícius Prates Altafini — RM559183

---

## 🔗 Links de Entrega (Requisito 0)

* **Aplicação (Azure Web App):** `https://webapp-challenge-945-sprint4.azurewebsites.net`
* **Vídeo de Demonstração (YouTube):** `[COLE AQUI O LINK DO SEU VÍDEO NO YOUTUBE]`
* **Repositório (GitHub):** `https://github.com/vinicius945/Sprint04_Dev_Ops_And_Cloud`
* **Projeto (Azure DevOps):** `https://dev.azure.com/RM5591830661/Sprint%204%20%E2%80%93%20Azure%20DevOps`

---

## 📌 1. Descrição da Solução (Requisito 1)

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

```mermaid
graph TD
    A[👨‍💻 Desenvolvedor] -- 1. git push --> B[<img src='[https://i.imgur.com/v0n2yS7.png](https://i.imgur.com/v0n2yS7.png)' width='20'/> GitHub (main)];
    B -- 2. Trigger --> C[<img src='[https://i.imgur.com/vzoR3GZ.png](https://i.imgur.com/vzoR3GZ.png)' width='20'/> Azure DevOps Pipeline];
    
    subgraph Estágio 1: CI (Build & Test)
        C -- 3. Build & Test --> D[<img src='[https://i.imgur.com/kS5xP3I.png](https://i.imgur.com/kS5xP3I.png)' width='20'/> mvn clean package];
        D -- 4. Publica --> E[<img src='[https://i.imgur.com/vzoR3GZ.png](https://i.imgur.com/vzoR3GZ.png)' width='20'/> Publica Testes (18) e Artefato (.jar)];
        D -- 5. Build Image & Push --> F[<img src='[https://i.imgur.com/f290gqj.png](https://i.imgur.com/f290gqj.png)' width='20'/> Azure Container Registry (ACR)];
    end

    subgraph Estágio 2: CD (Deploy)
        F -- 6. Trigger --> G[<img src='[https://i.imgur.com/vzoR3GZ.png](https://i.imgur.com/vzoR3GZ.png)' width='20'/> Azure DevOps Pipeline (Deploy)];
        G -- 7. Deploy Image --> H[<img src='[https://i.imgur.com/pYIqTfH.png](https://i.imgur.com/pYIqTfH.png)' width='20'/> Azure Web App (Contêiner)];
    end

    H -- 8. Conexão JDBC --> I[<img src='[https://i.imgur.com/o2PqBqH.png](https://i.imgur.com/o2PqBqH.png)' width='20'/> Azure SQL DB];
    J[👩‍💼 Usuário Final] -- 9. Acessa o site --> H;

    style A fill:#fff,stroke:#333,stroke-width:2px
    style J fill:#fff,stroke:#333,stroke-width:2px
```

---

## 📋 3. Detalhamento dos Componentes (Requisito 3)

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

## 🔑 Usuários para Teste

O `DataInitializer` cria dois usuários no primeiro deploy:
* **Usuário:** `admin` / **Senha:** `adminpass` (Role: ADMIN)