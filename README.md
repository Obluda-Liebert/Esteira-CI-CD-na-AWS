# Esteira de CI/CD para API Python na AWS

Este repositório contém a implementação de uma API em Flask (SQLite) com testes automatizados, conteinerização via Docker e esteira automatizada de CI/CD utilizando **GitHub Actions**, **GitHub Container Registry (GHCR)** e **AWS EC2**.

## Arquitetura e Fluxo de CI/CD

```
  [ Dev (Feature Branch) ] 
            │
            ▼ Git Push & Open PR
   ┌─────────────────┐
   │ GitHub Actions  │ ──► Runs CI: pytest + Docker Build
   └────────┬────────┘
            │ Merge na branch main
            ▼
   ┌─────────────────┐
   │ GitHub Actions  │ ──► Runs CD: Build & Push Image
   └────────┬────────┘          │
            │                   ▼
            │        ┌──────────────────────┐
            │        │ GHCR Image Registry  │
            │        └──────────────────────┘
            ▼
 ┌───────────────────────┐
 │ AWS EC2 (Deploy SSH)  │ ──► Docker Pull & Run
 └──────────┬────────────┘
            │
            ▼
 [ HTTP Health Check /health (Status 200 OK) ]

```

## Infraestrutura e Portas

* **Provedor:** Amazon Web Services (AWS)

* **Instância:** EC2 `t3.micro`

* **Regras do Security Group:**

  * `Porta 80 (HTTP)`: Liberada para `0.0.0.0/0` para acesso público à API na web.

  * `Porta 22 (SSH)`: Restrita ao IP do desenvolvedor/runners para gerenciamento seguro.

  * `Porta Interna do Contêiner`: `8000` mapeada para a porta `80` do host EC2.

## Tecnologias e Dependências

* **Linguagem & Framework:** Python 3.12, Flask, Gunicorn

* **Banco de Dados:** SQLite

* **Testes & Cobertura:** Pytest

* **Contêineres:** Docker & Docker Compose

* **CI/CD & Registro:** GitHub Actions & GitHub Container Registry (GHCR)

## Como Executar o Projeto

### Localmente (Desenvolvimento e Testes)

1. Ativar o ambiente virtual Python:

   ```
   python -m venv venv
   source venv/bin/activate  # No Windows: .\venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   
   ```

2. Executar os testes unitários e de integração:

   ```
   python -m pytest -v
   
   ```

3. Testar via Docker localmente:

   ```
   docker build -t api-local:v1 .
   docker run -d -p 8000:8000 -v "${PWD}/data:/data" --name api-test api-local:v1
   curl http://localhost:8000/health
   
   ```

## Evidências da Persistência de Dados

A persistência do banco de dados SQLite é garantida através de um **Docker Volume** mapeando o diretório `/data` do contêiner para a pasta `/home/ubuntu/app_data` na máquina host EC2.

### Passos de Validação:

1. **Criação de um registro:**

   ```
   curl -X POST http://<IP_EC2>/tasks -H "Content-Type: application/json" -d '{"title": "Tarefa Persistente"}'
   # Retorno: {"id": 1, "title": "Tarefa Persistente", "done": false}
   
   ```

2. **Remoção e reinicialização forçada do contêiner na EC2:**

   ```
   ssh -i "chave.pem" ubuntu@<IP_EC2>
   docker stop api-service && docker rm api-service
   
   ```

3. **Subida de novo contêiner reaproveitando o mesmo volume:**

   ```
   docker run -d -p 80:8000 -v /home/ubuntu/app_data:/data --name api-service ghcr.io/<usuario>/<repo>:latest
   
   ```

4. **Verificação de integridade dos dados:**

   ```
   curl http://<IP_EC2>/tasks
   # Retorno: [{"id": 1, "title": "Tarefa Persistente", "done": false}] -> DADOS MANTIDOS!
   
   ```

## Links Úteis de Evidência

* **Pull Request Aprovado (CI):** `https://github.com/Obluda-Liebert/Esteira-de-CI-CD-para-API-na-AWS/pull/1`

* **Run do Workflow de CI (Sucesso):** `https://github.com/Obluda-Liebert/Esteira-de-CI-CD-para-API-na-AWS/actions/runs/000000001`

* **Run do Workflow de CD (Sucesso):** `https://github.com/Obluda-Liebert/Esteira-de-CI-CD-para-API-na-AWS/actions/runs/000000002`

* **URL / HealthCheck da Aplicação:** `http://98.81.142.35/health`
