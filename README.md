# API de Pagamentos para Casamento

API Spring Boot para gerenciamento de lista de presentes de casamento com integração de pagamentos via ASAAS. Inclui persistência em PostgreSQL (Supabase), autenticação via Spring Security e suporte a múltiplos métodos de pagamento (PIX, Boleto, Cartão de Crédito).

## Visão Geral

- **Gestão de Presentes**: Listagem e detalhamento de presentes disponíveis
- **Sistema de Doações**: Processamento de pagamentos via PIX, Boleto ou Cartão de Crédito
- **Gerenciamento de Doadores**: Cadastro e controle de informações dos doadores
- **Integração ASAAS**: Gateway de pagamentos para processar transações
- **Segurança**: Configuração de CORS e endpoints protegidos

## Tecnologias

- Java 21
- Spring Boot 4.0.1
- Spring Data JPA
- Spring Security
- Spring Validation
- PostgreSQL (Supabase)
- ASAAS SDK (Gateway de Pagamentos)
- Docker
- Lombok

## Deploy

A aplicação está publicada e acessível em ambiente de produção através do **Render**:

🔗 **URL Base da API**

https://wedding-project-00so.onrender.com

## Estrutura do Projeto

```
src/main/java/com/capisite/backend/
├── config/                  # Configurações (ASAAS)
├── exceptions/              # Tratamento de exceções
├── infra/security/          # Configurações de segurança
└── modules/
    ├── donors/              # Módulo de doadores
    │   ├── Donor.java
    │   ├── DonorRepository.java
    │   ├── DonorService.java
    │   └── dto/
    ├── gifts/               # Módulo de presentes
    │   ├── Gift.java
    │   ├── GiftController.java
    │   ├── GiftRepository.java
    │   ├── GiftService.java
    │   └── dto/
    └── payments/            # Módulo de pagamentos
        ├── Payment.java
        ├── PaymentController.java
        ├── PaymentRepository.java
        ├── PaymentService.java
        ├── dto/
        └── enums/
```

## Pré-requisitos

- Java 21 instalado
- Maven instalado (ou utilize o Maven Wrapper incluso)
- Docker (opcional, para execução containerizada)
- Conta no Supabase (ou PostgreSQL local)
- Conta sandbox/produção no ASAAS

## Configuração de Ambiente

Configurações em `src/main/resources/application.properties`:

```properties
spring.application.name=backend

# --- Supabase Configuration ---
spring.datasource.url=jdbc:postgresql://aws-1-sa-east-1.pooler.supabase.com:6543/postgres?sslmode=require
spring.datasource.username=postgres.xxxxx
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false

# --- ASAAS Configuration ---
asaas.url=https://api-sandbox.asaas.com/v3
asaas.api.key=${ASAAS_API_KEY}
```

### Variáveis de Ambiente

Configure as seguintes variáveis de ambiente:

| Variável        | Descrição                          |
| --------------- | ---------------------------------- |
| `DB_PASSWORD`   | Senha do banco de dados PostgreSQL |
| `ASAAS_API_KEY` | Chave de API do ASAAS              |

## Executar a Aplicação

### Via Maven Wrapper (Windows)

```bash
./mvnw.cmd spring-boot:run
```

### Via Maven Wrapper (Linux/Mac)

```bash
./mvnw spring-boot:run
```

### Via Docker

Build da imagem:

```bash
docker build -t wedding-hugo-backend .
```

Executar container:

```bash
docker run -p 8080:8080 \
  -e DB_PASSWORD=sua_senha \
  -e ASAAS_API_KEY=sua_chave \
  wedding-hugo-backend
```

## Endpoints da API

### Presentes (Gifts)

| Método | Endpoint      | Descrição                |
| ------ | ------------- | ------------------------ |
| `GET`  | `/gifts`      | Lista todos os presentes |
| `GET`  | `/gifts/{id}` | Obtém um presente por ID |

### Pagamentos (Payments)

| Método | Endpoint    | Descrição            |
| ------ | ----------- | -------------------- |
| `POST` | `/payments` | Cria uma nova doação |

### Payload de Criação de Pagamento

```json
{
  "name": "Nome do Doador",
  "document": "12345678900",
  "email": "doador@email.com",
  "amount": 100.0,
  "billingType": "PIX",
  "message": "Felicidades ao casal!",
  "creditCardDetails": {
    "holderName": "NOME NO CARTAO",
    "number": "4444 4444 4444 4444",
    "expiryMonth": "12",
    "expiryYear": "2025",
    "ccv": "123"
  }
}
```

**Tipos de Pagamento (billingType):**

- `PIX` - Pagamento via PIX
- `BOLETO` - Pagamento via Boleto Bancário
- `CREDIT_CARD` - Pagamento via Cartão de Crédito
- `UNDEFINED` - Não definido

> **Nota:** O campo `creditCardDetails` é obrigatório apenas quando `billingType` for `CREDIT_CARD`.

## Modelo de Dados

### Gift (Presente)

| Campo         | Tipo   | Descrição             |
| ------------- | ------ | --------------------- |
| `id`          | Long   | Identificador único   |
| `name`        | String | Nome do presente      |
| `description` | String | Descrição do presente |
| `price`       | Double | Valor sugerido        |
| `image`       | String | URL da imagem         |

### Donor (Doador)

| Campo        | Tipo          | Descrição           |
| ------------ | ------------- | ------------------- |
| `id`         | UUID          | Identificador único |
| `externalId` | String        | ID externo (ASAAS)  |
| `name`       | String        | Nome do doador      |
| `email`      | String        | E-mail do doador    |
| `document`   | String        | CPF/CNPJ            |
| `createdAt`  | LocalDateTime | Data de criação     |

### Payment (Pagamento)

| Campo               | Tipo          | Descrição                  |
| ------------------- | ------------- | -------------------------- |
| `id`                | UUID          | Identificador único        |
| `donor`             | Donor         | Referência ao doador       |
| `amount`            | BigDecimal    | Valor do pagamento         |
| `billingType`       | String        | Tipo de pagamento          |
| `status`            | PaymentStatus | Status do pagamento        |
| `externalReference` | String        | Referência externa (ASAAS) |
| `paymentUrl`        | String        | URL para pagamento         |
| `message`           | String        | Mensagem do doador         |
| `createdAt`         | LocalDateTime | Data de criação            |

## Tratamento de Erros

A API retorna erros no formato padrão:

```json
{
  "timestamp": "2026-01-23T11:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Recurso não encontrado",
  "path": "/gifts/999"
}
```

## Versionamento

- **1.0.0-SNAPSHOT** - Versão inicial

## Autor

- [https://www.linkedin.com/in/luizfernando-react-java-fullstack/](https://www.linkedin.com/in/luizfernando-react-java-fullstack/)

---

Obrigado por visitar e bons códigos!
