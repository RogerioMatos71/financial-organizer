# Financial Organizer

Plataforma de Organização Financeira — nome interno do projeto. O nome comercial ainda não foi definido.

## Objetivo

Organizar compras e despesas a partir de fontes de dados, começando pela importação de XML de NFC-e (modelo 65). O sistema preserva o documento fiscal original e registra seus dados estruturados, permitindo evoluir futuramente para outras fontes, como QR Code, transações de cartão e integrações financeiras.

## Escopo do primeiro incremento

- Importar XML de NFC-e, modelo 65.
- Validar e processar o XML.
- Preservar o arquivo original fora do MySQL e calcular seu hash SHA-256.
- Registrar o documento fiscal e, quando aplicável, criar a compra e seus itens.
- Registrar cada tentativa de importação, inclusive duplicidades e falhas.
- Isolar os dados de cada usuário.
- Manter o cadastro de estabelecimentos (`Merchant`) compartilhado, sem compartilhar compras ou documentos entre usuários.

Não fazem parte do primeiro incremento: NF-e modelo 55, CF-e-SAT, ECF, classificação automática de produtos, integração com bancos/cartões, recuperação automática de documentos por CPF, armazenamento em nuvem e interface completa de usuário.

## Tecnologias previstas

- Java 21
- Spring Boot
- Maven
- Spring Data JPA
- MySQL
- Docker Compose para executar o MySQL localmente
- JUnit 5; Mockito, Spring Boot Test e Testcontainers conforme a necessidade
- Git/GitHub

A aplicação será executada diretamente pela IDE no início. Um Dockerfile da aplicação poderá ser criado posteriormente, quando houver motivo para executar a aplicação em contêiner.

## Documentação

- [Requisitos](docs/requirements.md)
- [Arquitetura](docs/architecture.md)
- [Modelo de domínio](docs/domain-model.md)
- [Modelo de banco de dados](docs/database-model.md)
- [Estratégia de testes](docs/testing-strategy.md)
- [Segurança](docs/security.md)
- [Registro de decisões](docs/decisions/decision-log.md)

## Situação

Projeto em fase de análise e desenho. O modelo físico do banco e o registro de decisões foram revisados e consolidados em novembro de 2026. Ainda existem validações fiscais e técnicas pendentes antes da elaboração do DDL definitivo e do início da implementação. Consulte `docs/database-model.md`, seção “Pendências antes do DDL/migrations”.
