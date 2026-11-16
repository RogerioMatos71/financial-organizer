# Modelo de banco de dados — versão consolidada para revisão

Este documento consolida as decisões aprovadas para o modelo físico inicial. Ele é a referência para a elaboração do DDL, mas **ainda não é o script SQL definitivo**. As pendências fiscais e de integração listadas ao final precisam ser resolvidas ou explicitamente aceitas antes da criação das migrations.

## 1. Convenções gerais

- SGBD: MySQL **8.0.16 ou superior**. A versão mínima considera o uso efetivo de `CHECK`; a collation escolhida também exige MySQL 8.0.
- Engine: InnoDB.
- Charset e collation padrão: `utf8mb4` e `utf8mb4_0900_ai_ci`.
- Tabelas e colunas em `snake_case`; a tabela da entidade Java `User` chama-se `app_user` para evitar conflito com a palavra `USER`.
- IDs e FKs: `INT` com sinal (padrão MySQL), sem `UNSIGNED`; PKs `INT AUTO_INCREMENT`; Java `Integer`.
- Valores monetários: `DECIMAL(15,2)` / `BigDecimal`, exceto `purchase_item.unit_price`, que usa `DECIMAL(15,10)`.
- Quantidades: `DECIMAL(15,4)` / `BigDecimal`.
- Não utilizar `double` ou `float` para valores fiscais ou financeiros. Converter os textos numéricos do XML diretamente para `BigDecimal`.
- Campos monetários opcionais ausentes na origem permanecem `NULL`; zero informado permanece `0.00`.
- Instantes: Java `Instant`; MySQL `DATETIME(6)`; todos os valores são normalizados para UTC. Como `DATETIME` não armazena fuso, a conversão será explícita na aplicação e coberta por testes de integração.
- Não adicionar `updated_at` no MVP.
- E-mail: normalizar para minúsculas antes de persistir e consultar; unicidade global em `app_user.email`.
- CNPJ: armazenar em `VARCHAR(14)`, sem pontuação, em maiúsculas; aceitar caracteres alfanuméricos conforme o formato oficial vigente. Validar formato e dígitos verificadores na aplicação.
- Todas as FKs usam `ON DELETE RESTRICT` e `ON UPDATE RESTRICT`. Não usar cascata no banco.

## 2. Tabelas

### 2.1 `app_user` (entidade Java `User`)

| Coluna | Tipo MySQL | Nulo? | Regra |
|---|---|---:|---|
| `id` | `INT` | Não | PK, AUTO_INCREMENT |
| `name` | `VARCHAR(150)` | Não | |
| `email` | `VARCHAR(254)` | Não | UNIQUE; normalizado em minúsculas |
| `password_hash` | `VARCHAR(255)` | Não | Hash de senha; nunca armazenar senha em texto puro |
| `status` | `VARCHAR(20)` | Não | `ACTIVE`, `INACTIVE`, `BLOCKED` |
| `created_at` | `DATETIME(6)` | Não | Criado pela aplicação, UTC |

Restrições: `UNIQUE(email)` e `CHECK(status IN ('ACTIVE','INACTIVE','BLOCKED'))`.

### 2.2 `merchant`

| Coluna | Tipo MySQL | Nulo? | Regra |
|---|---|---:|---|
| `id` | `INT` | Não | PK, AUTO_INCREMENT |
| `tax_id` | `VARCHAR(14)` | Sim | CNPJ sem pontuação; maiúsculas; UNIQUE quando informado |
| `legal_name` | `VARCHAR(200)` | Sim | Comprimento provisório |
| `trade_name` | `VARCHAR(200)` | Sim | Comprimento provisório |
| `created_at` | `DATETIME(6)` | Não | UTC |

Restrições:
- Pelo menos um entre `tax_id`, `legal_name` e `trade_name` deve estar preenchido (não nulo e não vazio após `TRIM`).
- `UNIQUE(tax_id)`; o MySQL permite vários `NULL` em uma coluna UNIQUE.
- `tax_id`, quando informado, deve ter 14 caracteres, estar em maiúsculas e conter somente `A-Z` e `0-9`. A validação oficial do CNPJ, incluindo dígitos verificadores, será feita na aplicação.
- `Merchant` é compartilhado e não possui `user_id`.

Ao encontrar CNPJ já cadastrado, reutilizar o `Merchant`, preencher campos ausentes e não sobrescrever automaticamente campos preenchidos divergentes.

### 2.3 `purchase`

| Coluna | Tipo MySQL | Nulo? | Regra |
|---|---|---:|---|
| `id` | `INT` | Não | PK, AUTO_INCREMENT |
| `purchase_date` | `DATETIME(6)` | Não | Instante da compra, UTC |
| `total_amount` | `DECIMAL(15,2)` | Não | Total de origem; não substituir por soma recalculada |
| `subtotal_amount` | `DECIMAL(15,2)` | Sim | NULL quando ausente |
| `discount_amount` | `DECIMAL(15,2)` | Sim | NULL quando ausente |
| `freight_amount` | `DECIMAL(15,2)` | Sim | NULL quando ausente |
| `additional_amount` | `DECIMAL(15,2)` | Sim | NULL quando ausente |
| `status` | `VARCHAR(20)` | Não | `PENDING`, `VALID`, `CANCELED` |
| `created_at` | `DATETIME(6)` | Não | Cadastro no sistema, UTC |
| `user_id` | `INT` | Não | FK para `app_user(id)` |
| `merchant_id` | `INT` | Sim | FK para `merchant(id)` |

Restrições:
- `UNIQUE(id, user_id)`, necessária para a FK composta de `fiscal_document`.
- `CHECK(status IN ('PENDING','VALID','CANCELED'))`.
- Valores monetários obrigatórios e opcionais, quando não nulos, não podem ser negativos.
- `user_id` obrigatório; `merchant_id` opcional.

### 2.4 `purchase_item`

| Coluna | Tipo MySQL | Nulo? | Regra |
|---|---|---:|---|
| `id` | `INT` | Não | PK, AUTO_INCREMENT |
| `description` | `VARCHAR(120)` | Não | Correspondente a `xProd`; limite conforme leiaute consultado |
| `quantity` | `DECIMAL(15,4)` | Não | Correspondente a `qCom`; maior que zero |
| `unit_of_measure` | `VARCHAR(6)` | Não | Correspondente a `uCom`; preservar o código original |
| `unit_price` | `DECIMAL(15,10)` | Não | Correspondente a `vUnCom`; preservar precisão |
| `gross_amount` | `DECIMAL(15,2)` | Não | Correspondente a `vProd` |
| `discount_amount` | `DECIMAL(15,2)` | Sim | Correspondente a `vDesc`, quando informado |
| `freight_amount` | `DECIMAL(15,2)` | Sim | Correspondente a `vFrete`, quando informado |
| `insurance_amount` | `DECIMAL(15,2)` | Sim | Correspondente a `vSeg`, quando informado |
| `additional_amount` | `DECIMAL(15,2)` | Sim | Correspondente a `vOutro`, quando informado |
| `included_in_document_total` | `BOOLEAN` | Não | Correspondente a `indTot`; 0 ou 1 |
| `purchase_id` | `INT` | Não | FK para `purchase(id)` |

Restrições:
- `quantity > 0`.
- Preço e valores monetários não negativos; campos opcionais permanecem `NULL` quando ausentes.
- `included_in_document_total` obrigatório; armazenar explicitamente o valor do XML, sem presumir valor quando ausente ou inválido.
- Uma compra pode ter zero ou vários itens.

Os valores recebidos do XML não serão arredondados nem substituídos por cálculos. Cálculos financeiros próprios, se adicionados futuramente, serão operações separadas.

### 2.5 `fiscal_document`

| Coluna | Tipo MySQL | Nulo? | Regra |
|---|---|---:|---|
| `id` | `INT` | Não | PK, AUTO_INCREMENT |
| `access_key` | `VARCHAR(44)` | Não | Exatamente 44 dígitos, sem pontuação |
| `document_number` | `VARCHAR(9)` | Não | `nNF`; texto para preservar zeros à esquerda |
| `series` | `VARCHAR(3)` | Não | `serie`; texto para preservar zeros à esquerda |
| `model` | `VARCHAR(2)` | Não | Primeiro incremento: `65` |
| `issue_date` | `DATETIME(6)` | Não | Emissão, normalizada para UTC |
| `fiscal_status` | `VARCHAR(20)` | Não | `AUTHORIZED`, `CANCELED`, `DENIED`, `UNKNOWN` |
| `document_total_amount` | `DECIMAL(15,2)` | Não | Total informado no documento |
| `authorization_protocol` | `VARCHAR(17)` | Sim | `nProt`, quando disponível; comprimento requer conferência normativa final |
| `authorized_at` | `DATETIME(6)` | Sim | Instante de autorização, UTC |
| `xml_storage_reference` | `VARCHAR(500)` | Não | Referência relativa, gerada pela aplicação |
| `xml_hash` | `CHAR(64)` | Não | SHA-256 hexadecimal minúsculo |
| `imported_at` | `DATETIME(6)` | Não | Importação pelo sistema, UTC |
| `user_id` | `INT` | Não | FK para `app_user(id)` |
| `merchant_id` | `INT` | Sim | FK para `merchant(id)` |
| `purchase_id` | `INT` | Sim | FK composta para `purchase`; UNIQUE |

Restrições:
- `UNIQUE(user_id, access_key)`: a mesma NFC-e pode ser importada por usuários diferentes, mas não duplicada para o mesmo usuário.
- `UNIQUE(purchase_id)`: no máximo um documento por compra.
- `UNIQUE(id, user_id)`: chave candidata para a FK composta de `import_attempt`.
- FK composta `(purchase_id, user_id)` → `purchase(id, user_id)`, garantindo o mesmo proprietário.
- `CHECK(model = '65')`.
- `CHECK` da chave de acesso: exatamente 44 dígitos (`^[0-9]{44}$`).
- `CHECK` do hash: exatamente 64 caracteres hexadecimais minúsculos (`^[0-9a-f]{64}$`).
- `fiscal_status` limitado aos quatro estados listados.
- `document_total_amount` não negativo.

`AUTHORIZED` exige evidência fiscal adequada de autorização; XML bem formado ou chave de acesso, isoladamente, não bastam. Sem evidência suficiente, usar `UNKNOWN`. Documento cancelado é preservado; se houver compra associada, ela deve ficar com status `CANCELED` e não ser contabilizada como compra válida.

O XML original fica fora do MySQL, em armazenamento privado. O banco mantém somente referência e hash. A referência é relativa à raiz do armazenamento, gerada pela aplicação e interpretada exclusivamente por `FiscalDocumentStorage`; não pode conter caminhos que escapem da raiz autorizada. O arquivo não pode ser sobrescrito silenciosamente.

### 2.6 `import_attempt`

| Coluna | Tipo MySQL | Nulo? | Regra |
|---|---|---:|---|
| `id` | `INT` | Não | PK, AUTO_INCREMENT |
| `file_name` | `VARCHAR(255)` | Não | Nome original apenas como metadado; nunca como caminho físico |
| `status` | `VARCHAR(30)` | Não | Estados listados abaixo |
| `error_message` | `TEXT` | Sim | Não guardar segredos, dados sensíveis ou XML integral |
| `attempted_at` | `DATETIME(6)` | Não | Início da tentativa, UTC |
| `finished_at` | `DATETIME(6)` | Sim | Conclusão, UTC; NULL enquanto IN_PROGRESS |
| `user_id` | `INT` | Não | FK para `app_user(id)` |
| `fiscal_document_id` | `INT` | Sim | FK composta com `user_id` |

Estados permitidos: `IN_PROGRESS`, `SUCCESS`, `DUPLICATE`, `INVALID_XML`, `PROCESSING_ERROR`, `STORAGE_ERROR`, `UNSUPPORTED_DOCUMENT`.

Restrições:
- `CHECK` limitando `status` aos sete valores.
- `IN_PROGRESS` exige `finished_at IS NULL`; estados finais exigem `finished_at IS NOT NULL`.
- `finished_at` não pode ser anterior a `attempted_at`.
- FK composta `(fiscal_document_id, user_id)` → `fiscal_document(id, user_id)`, garantindo o mesmo proprietário.
- Uma tentativa pode não identificar documento; várias tentativas podem referenciar o mesmo documento.
- A duração é calculada por `finished_at - attempted_at`, sem coluna adicional.

## 3. Chaves, índices e relacionamentos

### Chaves primárias

Todas as tabelas possuem `id INT AUTO_INCREMENT` como PK.

### Chaves estrangeiras

| Tabela | Colunas locais | Referência | Obrigatória |
|---|---|---|---:|
| `purchase` | `user_id` | `app_user(id)` | Sim |
| `purchase` | `merchant_id` | `merchant(id)` | Não |
| `purchase_item` | `purchase_id` | `purchase(id)` | Sim |
| `fiscal_document` | `user_id` | `app_user(id)` | Sim |
| `fiscal_document` | `merchant_id` | `merchant(id)` | Não |
| `fiscal_document` | `(purchase_id, user_id)` | `purchase(id, user_id)` | Não |
| `import_attempt` | `user_id` | `app_user(id)` | Sim |
| `import_attempt` | `(fiscal_document_id, user_id)` | `fiscal_document(id, user_id)` | Não |

As FKs compostas dependem de `UNIQUE(id, user_id)` em `purchase` e `fiscal_document`. Colunas opcionais compostas podem ser integralmente nulas quando a associação não existir.

### Restrições UNIQUE

- `app_user(email)`.
- `merchant(tax_id)`.
- `purchase(id, user_id)`.
- `fiscal_document(user_id, access_key)`.
- `fiscal_document(id, user_id)`.
- `fiscal_document(purchase_id)`.

### Índices iniciais

- `app_user`: UNIQUE em `email`.
- `merchant`: UNIQUE em `tax_id`.
- `purchase`: `(user_id, purchase_date)`; `merchant_id`; UNIQUE `(id, user_id)`.
- `purchase_item`: `(purchase_id)`.
- `fiscal_document`: UNIQUE `(user_id, access_key)`; `(user_id, issue_date)`; `(merchant_id)`; `(purchase_id, user_id)`; UNIQUE `(purchase_id)`; UNIQUE `(id, user_id)`.
- `import_attempt`: `(user_id, attempted_at)`; `(fiscal_document_id, user_id)`.

Os índices compostos adicionais nas colunas locais das FKs garantem suporte aos relacionamentos compostos. O MySQL pode criar índices automaticamente para FKs, mas o DDL deverá declarar explicitamente os índices planejados e evitar duplicações desnecessárias.

### Exclusão e atualização

Todas as FKs usam `ON DELETE RESTRICT ON UPDATE RESTRICT`. Não usar `CASCADE`.

- `User` não pode ser fisicamente excluído enquanto houver dados dependentes.
- `FiscalDocument` e `ImportAttempt` não são excluídos fisicamente pelo fluxo normal.
- `Merchant` pode ser excluído administrativamente somente se não tiver vínculos.
- `Purchase` pode ser excluída por operação explícita somente se não tiver `FiscalDocument` associado; a aplicação remove seus `PurchaseItem` antes da compra, em transação.
- Exceções administrativas devem ser auditáveis.

## 4. Regras de processamento relacionadas ao modelo

1. Criar `ImportAttempt` como `IN_PROGRESS` antes do processamento e finalizar com estado técnico.
2. O resultado técnico da importação é separado da situação fiscal.
3. XML malformado ou estruturalmente inválido: `INVALID_XML`; modelo não suportado: `UNSUPPORTED_DOCUMENT`; falhas técnicas: `PROCESSING_ERROR` ou `STORAGE_ERROR`; duplicidade: `DUPLICATE`; processamento concluído: `SUCCESS`.
4. Uma NFC-e cancelada pode ter `ImportAttempt.status = SUCCESS` e `FiscalDocument.fiscal_status = CANCELED`; se houver `Purchase`, seu status será `CANCELED`.
5. O XML original é preservado byte a byte; `xml_hash` é SHA-256 calculado sobre os bytes originais.
6. Armazenamento local inicial por `FiscalDocumentStorage`; organização aprovada: ano/mês, prefixo do hash e arquivo com nome baseado no hash. A referência é opaca para o restante do domínio.
7. Consistência arquivo/banco: transação MySQL e compensação no armazenamento. Em falha de banco, remover somente arquivo criado pela operação; falha de remoção gera pendência para limpeza administrativa. Não remover arquivo que já esteja referenciado por outro registro.

## 5. Pendências antes do DDL/migrations

1. Confirmar em fonte normativa vigente os comprimentos, ocorrências e condições de `nProt` (protocolo de 15/17 posições), especialmente efeitos da Nota Técnica aplicável à NFC-e.
2. Confirmar formalmente os comprimentos/ocorrências de `nNF`, `serie`, `xProd`, `uCom`, `qCom`, `vUnCom`, `vProd`, `indTot` e componentes de valores no leiaute NFC-e aplicável; decisões atuais refletem a revisão realizada, mas devem ser verificadas contra o XSD/NT efetivamente adotado.
3. Confirmar semântica e obrigatoriedade de `indTot` em todos os cenários de XML que o MVP aceitará.
4. Validar regras de totalização e tratamento dos campos `vProd`, `vDesc`, `vFrete`, `vSeg`, `vOutro` e itens com `indTot = 0`, sem recalcular ou substituir os valores originais.
5. Definir exatamente a evidência fiscal necessária para `AUTHORIZED`, `CANCELED` e `DENIED`, incluindo contingência e eventos fiscais; protocolo isolado pode não ser suficiente em todos os cenários.
6. Confirmar regra oficial de validação do CNPJ alfanumérico e sua data de aplicação; armazenar em texto já foi aprovado.
7. Confirmar comprimentos de `legal_name` e `trade_name` e os limites de mensagens de erro/nome de arquivo conforme o produto.
8. Testar no MySQL escolhido as FKs compostas, constraints `CHECK`, collation `utf8mb4_0900_ai_ci`, regexes e índices; revisar o DDL gerado pelo Hibernate/JPA se aplicável.
9. Definir e testar a conversão explícita `Instant` ↔ `DATETIME(6)` no driver JDBC/Hibernate com UTC.
10. Fechar a estratégia de migrations (por exemplo, Flyway ou Liquibase) antes da implementação do esquema.

