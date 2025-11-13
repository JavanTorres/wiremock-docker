# WireMock Docker - Mock API Completo

Servidor WireMock rodando via Docker Compose com suporte a **REST** e **GraphQL**, incluindo CRUD completo de usuários, cenários de erro, delays e dados randomizados.

## 📋 Estrutura do Projeto

```
wiremock-docker/
├── docker-compose.yml              # Configuração Docker Compose
├── README.md                        # Esta documentação
├── rest-tests.http                  # Testes REST (REST Client)
├── graphql-tests.http               # Testes GraphQL (REST Client)
├── __files/                         # Arquivos estáticos para responses
├── extensions/                      # Extensões customizadas
│   └── wiremock-faker-extension-0.2.0.jar  # Faker extension (não ativada)
└── mappings/                        # Mapeamentos de stubs
    ├── health-check.json            # Health check simples
    └── examples/
        ├── error-400.json           # Erro 400 - Bad Request com validação
        ├── error-404.json           # Erro 404 - Not Found
        ├── error-500.json           # Erro 500 - Internal Server Error
        ├── slow-response.json       # Resposta com delay 1s
        ├── timeout-error.json       # Resposta com delay 10s (timeout)
        ├── user-crud/               # CRUD REST de Usuários
        │   ├── user-by-id.json      # GET /api/users/{id}
        │   ├── user-list.json       # GET /api/users
        │   ├── user-create.json     # POST /api/users (201)
        │   ├── user-update.json     # PUT /api/users/{id}
        │   ├── user-patch.json      # PATCH /api/users/{id}
        │   ├── user-delete.json     # DELETE /api/users/{id} (204)
        │   └── user-not-found.json  # GET /api/users/999 (404)
        └── graphql/                 # GraphQL User CRUD
            ├── queries.json         # Query user by ID
            ├── query-users-list.json # Query all users
            ├── mutations.json       # Mutation createUser
            ├── update-mutation.json # Mutation updateUser
            └── delete-mutation.json # Mutation deleteUser
```

## 🚀 Quick Start

### Pré-requisitos
- Docker e Docker Compose instalados

### Subir o WireMock
```bash
docker compose up -d
```

### Verificar Status
```bash
docker compose ps
docker compose logs wiremock --tail 20
```

### Admin Dashboard
- WireMock Admin: http://localhost:8090/__admin
- Mappings Carregados: http://localhost:8090/__admin/mappings

### Parar e Limpar
```bash
docker compose down
docker compose restart wiremock
```

## 🔧 Configuração Docker Compose

```yaml
entrypoint: ["/docker-entrypoint.sh", "--global-response-templating", "--disable-gzip", "--verbose"]
```

**Flags importantes:**
- `--global-response-templating`: Habilita Handlebars templates em TODAS as respostas (essencial para dados dinâmicos)
- `--disable-gzip`: Desabilita compressão para facilitar debug
- `--verbose`: Logging detalhado em stdout

## 📡 APIs - REST

### Base URL
```
http://localhost:8090
```

### Variável no REST Client
```http
@restlUrl = http://localhost:8090
```

### Health Check
```bash
GET http://localhost:8090/api/health-check
```
Retorna: 200 OK

### CRUD Usuários

#### READ - Listar Todos
```bash
GET http://localhost:8090/api/users
```

Resposta (200 OK):
```json
[
  {
    "id": "user_550e8400",
    "name": "User_A1B2C3D4",
    "email": "user123456@example.com",
    "cep": "12345678",
    "createdAt": "2024-11-12 10:30:45"
  },
  ...
]
```

#### READ - Buscar por ID
```bash
GET http://localhost:8090/api/users/123
```

Resposta (200 OK):
```json
{
  "id": "123",
  "name": "User_XyZaBcD9",
  "email": "user999888@example.com",
  "cep": "87654321",
  "createdAt": "2024-11-12 10:30:45"
}
```

#### CREATE - Criar Usuário
```bash
POST http://localhost:8090/api/users
Content-Type: application/json

{
  "name": "João Silva",
  "email": "joao@example.com",
  "cep": "12345678"
}
```

Resposta (201 Created):
```json
{
  "id": "user_550e8400",
  "name": "João Silva",
  "email": "joao@example.com",
  "cep": "12345678",
  "createdAt": "2024-11-12 11:45:30"
}
```

Headers: `Location: /api/users/user_550e8400`

#### UPDATE - Atualizar Usuário (PUT - completo)
```bash
PUT http://localhost:8090/api/users/123
Content-Type: application/json

{
  "name": "João Atualizado",
  "email": "joao.updated@example.com",
  "cep": "99999999"
}
```

Resposta (200 OK):
```json
{
  "id": "123",
  "name": "João Atualizado",
  "email": "joao.updated@example.com",
  "cep": "99999999",
  "updatedAt": "2024-11-12 12:00:00"
}
```

#### UPDATE - Atualizar Parcial (PATCH)
```bash
PATCH http://localhost:8090/api/users/456
Content-Type: application/json

{
  "email": "novo@example.com"
}
```

Resposta (200 OK): Retorna usuário com novo email

#### DELETE - Remover Usuário
```bash
DELETE http://localhost:8090/api/users/789
```

Resposta: **204 No Content** (sem body)

### Cenários de Erro

#### 400 - Bad Request (Validação)
```bash
POST http://localhost:8090/api/bad-request
Content-Type: application/json

{
  "name": "JAVAN",
  "email": "javan@example.com"
}
```

Resposta (400):
```json
{
  "error": "Bad Request",
  "message": "Campo 'cep' é obrigatório"
}
```

**O mapping valida**: campos `name`, `email` e `cep` são obrigatórios. Enviar sem `cep` retorna 400.

#### 404 - Not Found
```bash
GET http://localhost:8090/api/not-found
```

Resposta (404):
```json
{
  "error": "Not Found",
  "message": "Recurso não encontrado"
}
```

#### 500 - Internal Server Error
```bash
GET http://localhost:8090/api/server-error
```

Resposta (500):
```json
{
  "error": "Internal Server Error",
  "timestamp": "2024-11-12 12:30:15"
}
```

#### Delays
- `GET /api/slow-endpoint` → **1 segundo de delay** (resposta normal após 1s)
- `GET /api/timeout` → **10 segundos de delay** (pode dar timeout em clientes com timeout < 10s)

## 🔬 GraphQL

### Base URL
```
http://localhost:8090/graphql
```

### Variáveis no REST Client
```http
@graphqlUrl = http://localhost:8090/graphql
@authToken = Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### CRUD Usuários (GraphQL)

#### Query - Listar Todos
```graphql
query GetUsers {
  users {
    id
    name
    email
    cep
    createdAt
  }
}
```

Resposta (200 OK):
```json
{
  "data": {
    "users": [
      {
        "id": "1",
        "name": "User_A1B2C3D4",
        "email": "user123@example.com",
        "cep": "12345678",
        "createdAt": "2024-11-12T10:30:45Z"
      },
      ...
    ]
  }
}
```

#### Query - Buscar por ID
```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    cep
    createdAt
  }
}
```

**Variables:**
```json
{
  "id": "123"
}
```

Resposta (200 OK):
```json
{
  "data": {
    "user": {
      "id": "123",
      "name": "User_X9Y8Z7A6",
      "email": "user999@example.com",
      "cep": "87654321",
      "createdAt": "2024-11-12T10:30:45Z"
    }
  }
}
```

#### Mutation - Criar Usuário
```graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
    cep
    createdAt
  }
}
```

**Variables:**
```json
{
  "input": {
    "name": "João Silva",
    "email": "joao@example.com",
    "cep": "12345678"
  }
}
```

Resposta (200 OK):
```json
{
  "data": {
    "createUser": {
      "id": "user_550e8400",
      "name": "João Silva",
      "email": "joao@example.com",
      "cep": "12345678",
      "createdAt": "2024-11-12T11:45:30Z"
    }
  }
}
```

#### Mutation - Atualizar Usuário
```graphql
mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
  updateUser(id: $id, input: $input) {
    id
    name
    email
    cep
    updatedAt
  }
}
```

**Variables:**
```json
{
  "id": "123",
  "input": {
    "name": "João Atualizado",
    "email": "joao.updated@example.com",
    "cep": "11111111"
  }
}
```

#### Mutation - Deletar Usuário
```graphql
mutation DeleteUser($id: ID!) {
  deleteUser(id: $id) {
    success
    message
  }
}
```

**Variables:**
```json
{
  "id": "789"
}
```

Resposta (200 OK):
```json
{
  "data": {
    "deleteUser": {
      "success": true,
      "message": "User deleted successfully"
    }
  }
}
```

## 📊 Dados Dinâmicos e Templating

### Como Funcionam os Dados Randomizados

O WireMock suporta **Handlebars templating** via `response-template` transformer. Cada resposta pode incluir templates que são processados no runtime.

**Exemplo em um mapping JSON:**
```json
{
  "response": {
    "status": 200,
    "jsonBody": {
      "id": "{{request.pathSegments.[2]}}",
      "name": "User_{{randomValue length=8 type='ALPHANUMERIC'}}",
      "email": "user{{randomValue length=6 type='NUMERIC'}}@example.com",
      "timestamp": "{{now format='yyyy-MM-dd HH:mm:ss'}}"
    },
    "transformers": ["response-template"]
  }
}
```

**Processamento:**
1. `{{request.pathSegments.[2]}}` → extrai o ID do path (ex: `/api/users/123` → `123`)
2. `{{randomValue length=8 type='ALPHANUMERIC'}}` → gera string aleatória (ex: `A1B2C3D4`)
3. `{{now format='...}}` → insere timestamp atual

**Resultado final:**
```json
{
  "id": "123",
  "name": "User_A1B2C3D4",
  "email": "user987654@example.com",
  "timestamp": "2024-11-12 12:30:45"
}
```

### Helpers Disponíveis (Nativos - sem extensões)

| Helper | Sintaxe | Exemplo | Resultado |
|--------|---------|---------|-----------|
| randomValue | `{{randomValue length=N type='TYPE'}}` | `{{randomValue length=8 type='ALPHANUMERIC'}}` | `a1B2x9Y8` |
| randomValue (UUID) | `{{randomValue type='UUID'}}` | `{{randomValue type='UUID'}}` | `550e8400-e29b-41d4-a716-446655440000` |
| now | `{{now format='format'}}` | `{{now format='yyyy-MM-dd'}}` | `2024-11-12` |
| request.pathSegments | `{{request.pathSegments.[N]}}` | `{{request.pathSegments.[2]}}` | `123` (para `/api/users/123`) |
| jsonPath | `{{jsonPath request.body '$.field'}}` | `{{jsonPath request.body '$.name'}}` | Valor de `name` do JSON |
| if/else | `{{#if condition}}...{{/if}}` | Condicional Handlebars | - |

### Validação com bodyPatterns

Matchear requests com regras específicas:

```json
{
  "request": {
    "method": "POST",
    "url": "/api/users",
    "bodyPatterns": [
      {
        "matchesJsonPath": "$.name"
      },
      {
        "matchesJsonPath": "$.email"
      },
      {
        "matchesJsonPath": {
          "expression": "$.cep",
          "absent": true
        }
      }
    ]
  },
  "response": {
    "status": 400,
    "jsonBody": { "error": "CEP é obrigatório" }
  }
}
```

## 🧩 Faker Extension - Análise Técnica

### O que é?

`wiremock-faker-extension` é uma extensão oficial do WireMock que integra a biblioteca **DataFaker** para gerar dados **realistas** em templates Handlebars.

**Site oficial:** https://wiremock.org/docs/faker-extension/

### Exemplo com Faker (se ativado)
```json
{
  "jsonBody": {
    "id": "{{uuid}}",
    "name": "{{random 'Name.full_name'}}",
    "email": "{{random 'Internet.email_address'}}",
    "phone": "{{random 'PhoneNumber.phone_number'}}",
    "zip": "{{random 'Address.zip_code'}}"
  },
  "transformers": ["response-template"]
}
```

**Geraria dados realistas:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "João Silva da Costa",
  "email": "joao.silva.costa@hotmail.com",
  "phone": "(11) 98765-4321",
  "zip": "01234-567"
}
```

### Por que não está ativado aqui?

Tentou-se implementar mas encontrou-se limitações críticas na imagem Docker:

| Etapa | Status | Motivo |
|-------|--------|--------|
| Download JAR | ✅ | `wiremock-faker-extension-0.2.0.jar` baixado |
| Cópia para extensions/ | ✅ | Arquivo copiado em `extensions/` |
| Carregamento via `--extensions` | ❌ | **ClassNotFoundException** |
| **Raiz do problema** | - | **Dependências transitivas faltando (DataFaker)** |

**Stack trace do erro:**
```
java.lang.ClassNotFoundException: org.wiremock.extensions.RandomExtension
  at java.lang.Class.forName(Unknown Source)
  at com.github.tomakehurst.wiremock.extension.Extensions.loadClass(...)
```

**Problemas técnicos específicos:**
1. **JAR da extensão é apenas interface** - `wiremock-faker-extension-0.2.0.jar` tem ~2KB (só as classes da extensão)
2. **Dependência oculta** - Necessita `net.datafaker:datafaker` no classpath (essa lib não vem na imagem Docker)
3. **Imagem standalone não inclui** - `wiremock/wiremock:latest` vem com classpath limitado
4. **Sem build customizado** - WireMock não consegue encontrar as classes DataFaker em runtime

### Como ativar o Faker (3 Soluções)

#### ✅ Opção 1: Build Customizado com Dockerfile (RECOMENDADO)

Criar arquivo `Dockerfile` na raiz do projeto:
```dockerfile
FROM wiremock/wiremock:latest

# Instalar curl e ferramentas
RUN apt-get update && apt-get install -y curl

# Criar pasta de libs se não existir
RUN mkdir -p /home/wiremock/lib

# Baixar DataFaker (versão compatível com a extensão)
RUN curl -L -o /home/wiremock/lib/datafaker-1.8.0.jar \
  https://repo1.maven.org/maven2/net/datafaker/datafaker/1.8.0/datafaker-1.8.0.jar

# Copiar extensão (já existe em extensions/)
COPY extensions/wiremock-faker-extension-0.2.0.jar /home/wiremock/extensions/

# Entrypoint padrão com templating
ENTRYPOINT ["/docker-entrypoint.sh", "--global-response-templating", "--extensions", "org.wiremock.extensions.RandomExtension"]
```

**Build e uso:**
```bash
docker build -t wiremock-faker:latest .

# Atualizar docker-compose.yml
```

```yaml
services:
  wiremock:
    image: wiremock-faker:latest
    ports:
      - "8090:8080"
    volumes:
      - ./mappings:/home/wiremock/mappings
      - ./extensions:/home/wiremock/extensions
```

```bash
docker compose up -d
```

#### ✅ Opção 2: Java Setup Local (Se controlar a execução)

Se rodar WireMock via Java em vez de Docker:

**pom.xml:**
```xml
<dependency>
  <groupId>org.wiremock.extensions</groupId>
  <artifactId>wiremock-faker-extension</artifactId>
  <version>0.2.0</version>
</dependency>
<dependency>
  <groupId>net.datafaker</groupId>
  <artifactId>datafaker</artifactId>
  <version>1.8.0</version>
</dependency>
```

**Java code:**
```java
import org.wiremock.extensions.RandomExtension;
import com.github.tomakehurst.wiremock.WireMockServer;
import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.*;

public class MockServer {
  public static void main(String[] args) {
    WireMockServer wms = new WireMockServer(
      wireMockConfig()
        .port(8090)
        .globalTemplating(true)
        .extensions(RandomExtension.class)
    );
    wms.start();
    System.out.println("WireMock running on http://localhost:8090");
  }
}
```

#### ⚠️ Opção 3: Solução Atual (Workaround - Funciona 100%)

**Status:** ✅ Implementado e testado

Usar helpers nativos do WireMock que funcionam **sem extensões**:

```json
{
  "name": "User_{{randomValue length=8 type='ALPHANUMERIC'}}",
  "email": "user{{randomValue length=6 type='NUMERIC'}}@example.com",
  "id": "{{randomValue type='UUID'}}",
  "timestamp": "{{now format='yyyy-MM-dd HH:mm:ss'}}"
}
```

**Vantagens:**
- ✅ Funciona imediatamente (sem configuração)
- ✅ Zero dependências adicionais
- ✅ Suficiente para 95% dos casos de teste
- ✅ Produz dados aleatórios válidos

**Desvantagens:**
- ❌ Dados menos realistas (`user999888@example.com` vs `joao.silva@hotmail.com`)
- ❌ Não valida formato (email pode ser inválido)
- ❌ Sem dados localizados (pt_BR, en_US, etc)

### Recomendação Geral

| Cenário | Solução | Por quê |
|---------|---------|--------|
| **Desenvolvimento local** | Opção 3 (nativa) | Rápido, sem setup |
| **Testing simples** | Opção 3 (nativa) | Dados aleatórios são suficientes |
| **Staging/Demo** | Opção 1 (Dockerfile) | Dados realistas impressionam stakeholders |
| **Integração CI/CD** | Opção 1 (Dockerfile) | Confiável, reproduzível, sem permissões |

**Para este projeto:** Recomenda-se implementar **Opção 1 (Dockerfile)** em produção. Por enquanto, a Opção 3 (nativa) está totalmente funcional.

## 🧪 Testando com REST Client (VS Code)

### Instalação
1. Instale a extensão `REST Client` (Huachao Mao) no VS Code: https://marketplace.visualstudio.com/items?itemName=humao.rest-client
2. Abra arquivo `.http` (ex: `rest-tests.http`)
3. Clique em `Send Request` acima de cada teste
4. Resposta aparece em painel lateral

### Arquivo rest-tests.http

Define variáveis reutilizáveis:
```http
@restlUrl = http://localhost:8090
@graphqlUrl = http://localhost:8090/graphql

### Health Check
GET {{restlUrl}}/api/health-check HTTP/1.1

### Lista Usuários
GET {{restlUrl}}/api/users HTTP/1.1

### Buscar por ID
GET {{restlUrl}}/api/users/123 HTTP/1.1

### Criar Usuário
POST {{restlUrl}}/api/users HTTP/1.1
Content-Type: application/json

{
  "name": "João Silva",
  "email": "joao@example.com",
  "cep": "12345678"
}
```

**Vantagens do sistema de variáveis:**
- ✅ Centralizadas (fácil mudar base URL)
- ✅ Sem repetição (DRY principle)
- ✅ Suporta múltiplos ambientes (`@restlUrl_dev`, `@restlUrl_prod`)
- ✅ Tokens e headers reutilizáveis

### Arquivo graphql-tests.http

```http
@graphqlUrl = http://localhost:8090/graphql
@authToken = Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

### Query GetUsers
POST {{graphqlUrl}}
Authorization: {{authToken}}
Content-Type: application/json

{
  "query": "query GetUsers { users { id name email } }"
}

### Mutation CreateUser
POST {{graphqlUrl}}
Authorization: {{authToken}}
Content-Type: application/json

{
  "query": "mutation CreateUser(...) { ... }",
  "variables": {
    "input": {
      "name": "João",
      "email": "joao@example.com",
      "cep": "12345678"
    }
  }
}
```

## 🛠️ Troubleshooting

### Problema: "Request foi matched mas retornou resposta errada"

**Causa:** Múltiplos mappings fazendo match, o WireMock usa o primeiro encontrado

**Debug:**
```bash
curl -v http://localhost:8090/__admin/mappings
```

**Solução:** Adicione `priority` aos mappings (maior = testado primeiro):
```json
{
  "priority": 1,
  "request": { ... },
  "response": { ... }
}
```

### Problema: "Dados randomizados não aparecem na resposta"

**Causa 1:** Falta `"transformers": ["response-template"]` no mapping

**Verificar:**
```bash
curl http://localhost:8090/__admin/mappings | jq '.[] | select(.id=="seu-mapping-id")'
```

**Adicionar ao mapping:**
```json
{
  "response": {
    "jsonBody": { "name": "{{randomValue}}" },
    "transformers": ["response-template"]
  }
}
```

**Causa 2:** WireMock não carregou a flag `--global-response-templating`

**Verificar no docker-compose.yml:**
```yaml
entrypoint: ["/docker-entrypoint.sh", "--global-response-templating"]
```

**Causa 3:** Syntax Handlebars incorreta

**Correto:** `{{randomValue length=8 type='ALPHANUMERIC'}}`
**Incorreto:** `{randomValue}` ou `{{ randomValue }}` (espaços extra)

### Problema: "400 erro não é retornado, retorna 404"

**Causa:** `bodyPatterns` não faz match com o JSON enviado

**Debug:**
```bash
curl -X POST http://localhost:8090/api/bad-request \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","email":"test@test.com"}' \
  -v
```

**Verificar mapping:**
Abra `error-400.json` e veja os `bodyPatterns`. Compare com JSON enviado.

**Dica:** WireMock retorna 404 se NENHUM mapping fizer match. Se o mapping fizer match mas a validação falhar, revise as regras.

### Problema: "GraphQL mutations retornam dados de query"

**Causa:** Prioridade dos mappings (query-matcher muito genérico)

**Solução:** Verificar prioridades em `graphql/` folder:
- `delete-mutation.json`: `"priority": 1`
- `update-mutation.json`: `"priority": 2`
- `mutations.json`: `"priority": 3`
- `queries.json`: `"priority": 5`

Maior priority = testado primeiro = mais específico.

### Problema: "Container não sobe, Docker daemon não responde"

**Solução:**
```bash
# Windows: Reiniciar Docker Desktop
# Linux: Verificar daemon
sudo systemctl restart docker

# Ou use WSL2 se estiver no Windows com Docker Desktop
wsl --list -v
```

### Problema: "Mapping não é carregado mesmo após arquivo criado"

**Solução 1:** Restart do container
```bash
docker compose restart wiremock
```

**Solução 2:** Verificar volume mapping em docker-compose.yml
```yaml
volumes:
  - ./mappings:/home/wiremock/mappings
```

**Solução 3:** Recriar container
```bash
docker compose up -d --force-recreate
```

## 📚 Recursos Úteis

| Recurso | Link |
|---------|------|
| **WireMock Official** | https://wiremock.org |
| **WireMock Docs - Faker** | https://wiremock.org/docs/faker-extension/ |
| **DataFaker Docs** | https://www.datafaker.net |
| **Handlebars Docs** | https://handlebarsjs.com |
| **REST Client Extension** | https://marketplace.visualstudio.com/items?itemName=humao.rest-client |
| **Docker Compose Docs** | https://docs.docker.com/compose/ |
| **JSON Path** | https://github.com/json-path/JsonPath |

## ✅ Status do Projeto

| Recurso | Status | Notas |
|---------|--------|-------|
| Docker Compose | ✅ | Configurado, flags otimizadas |
| REST CRUD | ✅ | Completo (GET list, GET id, POST, PUT, PATCH, DELETE) |
| GraphQL CRUD | ✅ | Completo (2 queries, 3 mutations com prioridade) |
| Cenários de Erro | ✅ | 400 (validação), 404, 500 implementados |
| Validação (bodyPatterns) | ✅ | POST requer name, email, cep |
| Delays | ✅ | slow (1s), timeout (10s) |
| Dados Randomizados | ✅ | Via `randomValue` + `now` (nativo, sem extensões) |
| Faker Extension | ⚠️ | JAR disponível em `extensions/`, requer build Dockerfile |
| Autenticação GraphQL | ✅ | Bearer token de exemplo em variáveis |
| Variáveis REST Client | ✅ | @restlUrl, @graphqlUrl padronizadas |
| Testes (.http files) | ✅ | rest-tests.http + graphql-tests.http completos |

## 📝 Melhorias Futuras

- [ ] Implementar Faker Extension com Dockerfile customizado
- [ ] Adicionar scenarios stateful (workflow multi-step com estado)
- [ ] Suporte a WebSocket mocking
- [ ] Integração com CI/CD (GitHub Actions, GitLab CI)
- [ ] Exportar dados de request journal para análise
- [ ] Rate limiting e throttling
- [ ] Mock de serviços de terceiros (Stripe, PayPal, etc.)
- [ ] Suporte a SOAP/XML além de JSON/GraphQL
- [ ] Custom matchers para validações complexas
- [ ] Gravação e playback de requisições reais
- [ ] Dashboard web customizado para gerenciar stubs
- [ ] Suporte a mTLS/certificados

## 📄 Referências de Implementação

### Exemplo Mapping Completo (REST POST)
Ver: `mappings/examples/user-crud/user-create.json`
```json
{
  "request": {
    "method": "POST",
    "urlPath": "/api/users",
    "bodyPatterns": [
      { "matchesJsonPath": "$.name" },
      { "matchesJsonPath": "$.email" },
      { "matchesJsonPath": "$.cep" }
    ]
  },
  "response": {
    "status": 201,
    "jsonBody": {
      "id": "user_{{randomValue type='UUID'}}",
      "name": "{{jsonPath request.body '$.name'}}",
      "email": "{{jsonPath request.body '$.email'}}",
      "cep": "{{jsonPath request.body '$.cep'}}",
      "createdAt": "{{now format='yyyy-MM-dd HH:mm:ss'}}"
    },
    "headers": {
      "Location": "/api/users/user_{{randomValue type='UUID'}}"
    },
    "transformers": ["response-template"]
  }
}
```

### Exemplo GraphQL Query (com prioridade)
Ver: `mappings/examples/graphql/query-users-list.json`
```json
{
  "priority": 5,
  "request": {
    "method": "POST",
    "urlPathPattern": "/graphql.*",
    "bodyPatterns": [
      { "matchesJsonPath": "$[?(@.query =~ /.*users.*query.*/)]" }
    ]
  },
  "response": {
    "status": 200,
    "jsonBody": {
      "data": {
        "users": [
          {
            "id": "{{randomValue type='UUID'}}",
            "name": "User_{{randomValue length=8}}",
            "email": "user{{randomValue length=6 type='NUMERIC'}}@example.com"
          }
        ]
      }
    },
    "transformers": ["response-template"]
  }
}
```

---

**Versão:** 2.0 (11/2024)
**WireMock:** Latest Docker image
**Última atualização:** 12 de novembro de 2024
**Autor:** Desenvolvido e testado com WireMock 3.x
