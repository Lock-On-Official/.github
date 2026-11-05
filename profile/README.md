<div align="center">

<!-- Espaço para o logo/banner do projeto: troque o caminho quando a imagem existir -->
<img src="https://github.com/user-attachments/assets/385b7a5b-c715-4fc8-b2df-14d4defdfb1c" alt="Lockon CRM" width="500" />

# Lockon CRM

**API de CRM modular para gestão de empresas, contatos, funil de vendas e atividades.**

![Java](https://img.shields.io/badge/Java-17-orange?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.1-6DB33F?logo=springboot&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-4169E1?logo=postgresql&logoColor=white)
![Flyway](https://img.shields.io/badge/Flyway-migrations-CC0200?logo=flyway&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)

</div>

---

## 📖 Sobre o Projeto

O Lockon CRM é o backend de um sistema de relacionamento com clientes (CRM) desenvolvido pela equipe Lock-On. Ele organiza o dia a dia comercial: quem são os clientes, em que etapa do funil cada negociação está e quais ações precisam ser feitas, e com que prazo.

**Problema que resolve:** equipes de vendas que controlam clientes e negócios em planilhas perdem o histórico, não enxergam os prazos e não têm uma visão confiável do funil. O Lockon centraliza essas informações em uma API única, com acesso por login e regras de negócio validadas no servidor.

**Arquitetura:** monólito modular. Cada funcionalidade fica num módulo isolado (`identity`, `contacts`, `pipeline` e `activities`), dividido nas camadas `api`, `application` e `domain`. O código compartilhado (configuração, segurança, tratamento de erros e a entidade base) fica em `shared`, que não depende de nenhum módulo.

**Status:** em desenvolvimento. Cadastros, funil, oportunidades e atividades já estão disponíveis. A linha do tempo do cliente e a gestão de usuários estão planejadas no backlog.

---

## ✨ Funcionalidades

- 🔐 **Autenticação com JWT**: login com e-mail e senha, token no padrão OAuth2 (`Bearer`), validade configurável e senhas protegidas com BCrypt.
- 🛡️ **Controle de acesso por papel**: papéis `ADMIN`, `VENDEDOR` e `GERENTE`, com suporte a `@PreAuthorize`.
- 👤 **Administrador inicial**: criado na primeira subida a partir de variáveis de ambiente.
- 🏢 **Empresas**: cadastro, consulta, edição e exclusão.
- 👥 **Contatos**: pessoas físicas (PF) e jurídicas (PJ), com endereço e vínculo com empresa. A exclusão é lógica (soft delete).
- 📊 **Pipeline de vendas**: funis com etapas configuráveis, reordenação das etapas e tipos de etapa (aberta, ganha ou perdida).
- 💼 **Oportunidades**: negócios com valor, probabilidade e data prevista de fechamento. O valor esperado é calculado automaticamente, e mudar de etapa publica um evento de domínio.
- 📅 **Atividades**: ligações, e-mails, reuniões e follow-ups, com prazo, lembrete e conclusão. Cada atividade pertence a um contato e pode estar ligada a uma oportunidade.
- ✅ **Erros padronizados**: respostas no formato `ProblemDetail` (RFC 9457), com o status HTTP correto e sem vazar detalhes internos.
- 📚 **Documentação interativa**: Swagger UI gerado a partir do código.
- 🩺 **Health check**: `/actuator/health`, com grupos de liveness e readiness.

### Principais endpoints

| Recurso | Rotas |
|---|---|
| Autenticação | `POST /auth/login` |
| Empresas | `/companies`: CRUD |
| Contatos | `/contacts`: CRUD |
| Pipelines e etapas | `/pipelines`: CRUD, `/pipelines/{id}/stages`: CRUD e `PUT /reorder` |
| Oportunidades | `/opportunities`: CRUD e `PATCH /{id}/stage` |
| Atividades | `/activities`: CRUD e `PATCH /{id}/complete` |

Todas as rotas exigem token, exceto `/auth/login`, a documentação e o health check.

---

## 🧰 Tecnologias Utilizadas

**Backend**
- Java 17
- Spring Boot 4.1.1 (Spring Web MVC, Validation e Actuator)
- Spring Security com OAuth2 Resource Server (JWT, HS256)
- Spring Data JPA e Hibernate (`ddl-auto: validate`)
- Lombok
- SpringDoc OpenAPI (Swagger UI)

**Banco de dados**
- PostgreSQL 18
- Flyway (migrations versionadas)

**Testes**
- JUnit 5, Mockito e AssertJ
- Testcontainers (PostgreSQL real nos testes de integração)

**Infraestrutura e entrega**
- Docker e Docker Compose
- GitHub Actions (testes, publicação da imagem, análise de segredos e análise estática de segurança)
- GitHub Container Registry (GHCR)
- Maven Wrapper (`mvnw`)

---

## 🚀 Pré-requisitos e Como Executar

### Pré-requisitos

- [JDK 17](https://adoptium.net/)
- [Docker](https://www.docker.com/) e Docker Compose (também usados pelos testes)
- Git

### 1. Clonar o repositório

```bash
git clone https://github.com/Lock-On-Official/crm-lockon.git
cd crm-lockon
```

### 2. Subir o banco de dados

```bash
docker compose up -d postgres-db
```

O banco fica disponível em `localhost:5440` (usuário `postgres`, senha `postgres`, banco `lockon`). As credenciais podem ser trocadas pelas variáveis `LOCKON_DB_NAME`, `LOCKON_DB_USER` e `LOCKON_DB_PASSWORD`.

### 3. Configurar as variáveis de ambiente

Todas as variáveis do projeto começam com `LOCKON_`.

| Variável | Obrigatória | Descrição |
|---|---|---|
| `LOCKON_JWT_SECRET` | Sim, fora do perfil `dev` | Segredo de assinatura dos tokens (mínimo de 32 caracteres) |
| `LOCKON_JWT_EXPIRATION` | Não | Validade do token, como `8h` ou `30m` (padrão: `8h`) |
| `LOCKON_ADMIN_EMAIL` | Não | E-mail do administrador inicial |
| `LOCKON_ADMIN_PASSWORD` | Não | Senha do administrador inicial |

Crie um arquivo `.env` na raiz do projeto (ele não deve ir para o Git) ou exporte as variáveis no terminal. Para gerar um segredo forte:

```bash
openssl rand -base64 48
```

### 4. Rodar a aplicação

Com o perfil `dev`, que já traz um segredo de desenvolvimento e carrega empresas e contatos de exemplo:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

Na IntelliJ, preencha **Active profiles** com `dev` na Run Configuration.

A API fica em `http://localhost:8080`.

### 5. Rodar tudo pelo Docker

```bash
docker compose --profile app up -d --build
```

### 6. Rodar os testes

Os testes de integração sobem um PostgreSQL temporário com Testcontainers, então o Docker precisa estar em execução.

```bash
./mvnw test
```

### 7. Gerar o pacote

```bash
./mvnw clean package
```

### Primeiro acesso

1. Confirme que a aplicação subiu e que a linha `Admin inicial criado` aparece no log (ou que o administrador já existia).
2. Faça login:
   ```bash
   curl -X POST http://localhost:8080/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email":"seu-admin@exemplo.com","password":"sua-senha"}'
   ```
3. Use o `access_token` retornado no cabeçalho `Authorization: Bearer <token>`.
4. Explore a documentação em `http://localhost:8080/swagger-ui.html`.

---

## 📁 Estrutura de Pastas

```text
crm-lockon/
├── .github/                    # Workflows do GitHub Actions e configuração do labeler
├── .claude/                    # Skills e subagentes usados nas ferramentas de IA
├── docs/                       # Manual por branch, backlog de cards e relatórios de segurança
├── src/
│   ├── main/
│   │   ├── java/br/crm/lockon/
│   │   │   ├── shared/         # Transversal: config, entidade base e exceções (sem regra de negócio)
│   │   │   └── modules/
│   │   │       ├── identity/   # Usuários, login e JWT
│   │   │       ├── contacts/   # Empresas e contatos
│   │   │       ├── pipeline/   # Pipelines, etapas e oportunidades
│   │   │       └── activities/ # Atividades
│   │   │           # cada módulo: api/ · application/ · domain/
│   │   └── resources/
│   │       ├── db/migration/   # Migrations do Flyway (V001 em diante)
│   │       └── application*.yaml
│   └── test/                   # Testes unitários e de integração
├── docker-compose.yml          # Banco de dados e, opcionalmente, a aplicação
├── Dockerfile                  # Imagem da aplicação
├── pom.xml
├── mvnw / mvnw.cmd             # Maven Wrapper
├── CLAUDE.md                   # Regras de trabalho para o time e para ferramentas de IA
└── README.md
```

---

## 🤝 Como Contribuir

Obrigado pelo interesse em contribuir! Para manter o projeto organizado, siga estas diretrizes.

1. **Crie uma branch a partir da `main`** com um nome descritivo, por exemplo `feature/contatos-importacao` ou `fix/validacao-cnpj`.
2. **Mantenha o escopo pequeno.** Um PR deve tratar uma funcionalidade ou correção.
3. **Siga a arquitetura de módulos.** Use as camadas `api`, `application` e `domain`, e não acesse repositórios de outro módulo.
4. **Escreva testes.** Todo endpoint novo precisa de testes de sucesso e de erro.
5. **Mudanças no banco entram como migration nova**, com o próximo número livre. Nunca edite uma migration já aplicada.
6. **Commits no padrão Conventional Commits**, com o assunto em português:
   ```text
   feat(contacts): adiciona importação de contatos por CSV
   ```
7. **Rode `./mvnw test` antes de abrir o PR.** A CI roda os mesmos testes em cada PR.
8. **Documente decisões não óbvias** no código (com comentários curtos) ou em `docs/branches/`.
9. **Não envie segredos.** O workflow `secret-guard` bloqueia PRs com tokens, senhas ou arquivos `.env`.

Antes de começar, leia o [`CLAUDE.md`](CLAUDE.md), que descreve as convenções de código e de trabalho do projeto.

---

## 👥 Autores e Contato

**Lock-On Official**, equipe de desenvolvimento do Lockon CRM

- 🐙 GitHub da organização: [Lock-On-Official](https://github.com/Lock-On-Official)
- 👨‍💻 Autor principal: [Guilherme Lima](https://github.com/Gu1Ml)

Dúvidas e sugestões: abra uma [issue](https://github.com/Lock-On-Official/crm-lockon/issues) no repositório.
