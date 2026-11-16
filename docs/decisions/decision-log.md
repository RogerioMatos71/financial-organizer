# Registro consolidado de decisões

Este arquivo resume as decisões tomadas durante a análise do projeto. Os números identificam a sequência histórica das decisões; esta consolidação não substitui eventuais registros individuais que venham a ser criados futuramente.

| DEC | Tema | Decisão consolidada |
|---|---|---|
| 001 | Público-alvo | Núcleo comum para pessoas físicas e empresas. |
| 002 | Entrada de dados | Começar com XML de NFC-e e prever outras fontes futuramente. |
| 003 | Primeiro incremento | Importar NFC-e e persistir os dados; classificação e consultas completas ficam para depois. |
| 004 | Domínio | Separar `FiscalDocument` de `Purchase`; compra pode existir antes do documento. |
| 005 | Nome | “Plataforma de Organização Financeira” é nome interno; marca comercial permanece em aberto. |
| 006 | NFC-e cancelada | Preservar documento e XML cancelados, sem contabilizar como compra válida. |
| 007 | XML original | Preservar XML original e dados estruturados; não sobrescrever silenciosamente. |
| 008 | Duplicidade | Não duplicar nem atualizar automaticamente documento com mesma chave; registrar nova tentativa como duplicada. Atualização fiscal posterior é fluxo separado. |
| 009 | Ambiente | Docker Compose para MySQL; aplicação executada localmente na IDE inicialmente. |
| 010 | Cardinalidade | Purchase e FiscalDocument têm associação opcional um-para-um. |
| 011 | Itens | Usar uma única entidade `PurchaseItem`; XML original preserva os demais detalhes fiscais. |
| 012 | Valores da compra | Purchase mantém total e componentes disponíveis; total de origem não será substituído por soma recalculada. |
| 013 | Unidade | PurchaseItem preserva unidade original e permite quantidade decimal. |
| 014 | Classificação | Não classificar ou normalizar produtos no primeiro incremento. |
| 015 | Situação fiscal | Separar estado fiscal do resultado técnico de importação. |
| 016 | Identificador fiscal | Conceito inicialmente genérico, posteriormente simplificado para CNPJ no MVP (ver DEC-044). |
| 017 | Merchant | Cada CNPJ representa um Merchant independente; sem grupo empresarial/filiais no MVP. |
| 018 | Usuário e isolamento | User desde a primeira versão; compras e documentos pertencem a um usuário. |
| 019 | Conta/perfil financeiro | Sem múltiplas contas ou perfis financeiros no primeiro incremento. |
| 020 | Duplicidade por usuário | Unicidade de NFC-e por `(user_id, access_key)`; usuários diferentes podem importar a mesma chave. |
| 021 | ImportAttempt | Registrar cada tentativa, inclusive quando nenhum documento é identificado. |
| 022 | Purchase status | `PENDING`, `VALID`, `CANCELED`. |
| 023 | FiscalDocument status | `AUTHORIZED`, `CANCELED`, `DENIED`, `UNKNOWN`; autorização exige validação apropriada. |
| 024 | ImportAttempt status | `SUCCESS`, `DUPLICATE`, `INVALID_XML`, `PROCESSING_ERROR`, `STORAGE_ERROR`, `UNSUPPORTED_DOCUMENT`; posteriormente acrescentado `IN_PROGRESS`. |
| 025 | Consistência arquivo/banco | Transação MySQL e compensação no armazenamento; registrar falha de limpeza. |
| 026 | IN_PROGRESS | Criar tentativa antes do processamento e atualizar ao final. |
| 027 | xmlHash | SHA-256 do conteúdo original do XML. |
| 028 | Métricas de importação | Proposta inicial de `finishedAt` e duração; duração armazenada foi posteriormente retirada pela DEC-043. |
| 029 | ImportAttempt e documento | Tentativa pertence a User, pode referenciar zero ou um documento; documento pode ser referenciado por várias tentativas. Histórico preservado. |
| 030 | Propriedade | Purchase e FiscalDocument associados devem pertencer ao mesmo User; validar na aplicação e no banco. |
| 031 | Merchant compartilhado | Merchant não pertence a User; documentos e compras permanecem individualizados. |
| 032 | Identificação mínima de Merchant | Exigir pelo menos um entre taxId, legalName e tradeName; se taxId for informado, os metadados previstos deveriam acompanhar, posteriormente simplificados para CNPJ no MVP. |
| 033 | Merchant opcional | Purchase e FiscalDocument podem existir sem Merchant. |
| 034 | E-mail | E-mail de User único globalmente. |
| 035 | User status | `ACTIVE`, `INACTIVE`, `BLOCKED`. |
| 036 | Data da compra | Armazenar data e horário completos. |
| 037 | Valores monetários | DECIMAL(15,2) no MySQL e BigDecimal no Java. |
| 038 | Quantidades | DECIMAL(15,4) e BigDecimal no Java. |
| 039 | Valores de item | Manter preço unitário e valor total de origem; sem substituir por cálculo próprio. |
| 040 | Desconto de item | Incluir `discountAmount` opcional. |
| 041 | Acréscimos de item | Incluir `freightAmount` e `additionalAmount` opcionais. |
| 042 | Identificadores internos | Usar INT/Integer para IDs e FKs inicialmente; migrar para BIGINT somente se necessário. |
| 043 | Duração de tentativa | Não armazenar `processingDurationMs`; calcular por attemptedAt e finishedAt. |
| 044 | CNPJ no MVP | `taxId` armazena CNPJ; não implementar `taxIdType` nem `countryCode` agora. |
| 045 | Armazenamento | Interface simples `FiscalDocumentStorage` e implementação local; sem nuvem no MVP. |
| 046 | Processamento fiscal | Processador específico para NFC-e modelo 65; outros documentos ficam fora do MVP. |
| 047 | Precisão fiscal do item | `unitPrice` com até 10 casas; renomear `totalPrice` para `grossAmount`; incluir `insuranceAmount` e `includedInDocumentTotal`. |
| 048 | Fuso horário | Armazenar somente instante UTC; não preservar offset original no MVP. |
| 049 | Atualização Merchant | Reutilizar CNPJ; preencher campos vazios; não sobrescrever automaticamente divergências. |
| 050 | Arquivos órfãos | Compensação de arquivo e verificação administrativa simples de órfãos no MVP. |
| 051 | Testes | Testes automatizados acompanham desenvolvimento: unitários, integração, funcionais e regressão, conforme necessidade. |
| 052 | Segurança | Segurança e proteção de dados desde a concepção; isolamento, proteção de XML, credenciais, backups, LGPD e controles associados. |

## Decisões revisadas

- DEC-028 foi parcialmente revisada pela DEC-043: `finishedAt` permanece; `processingDurationMs` não será armazenado.
- A ideia genérica de identificador fiscal foi simplificada pela DEC-044: no primeiro incremento, somente CNPJ.
- A precisão de `unitPrice` e a semântica de valor bruto foram refinadas pela DEC-047.

## Observação

Quando uma decisão depender de validação fiscal, técnica ou de comportamento do MySQL, sua implementação deverá aguardar essa validação. A aprovação de um conceito não significa que todos os detalhes de DDL ou integração estejam fechados.


## Decisões complementares da revisão do modelo físico (14/11/2026)

As decisões abaixo complementam a consolidação DEC-001 a DEC-052 e registram as aprovações feitas durante a revisão detalhada do modelo físico. Quando uma regra depende de validação normativa ou de teste do SGBD, essa dependência permanece indicada como pendência.

| DEC | Tema | Decisão consolidada |
|---|---|---|
| 053 | Nome físico da tabela de usuário | A entidade Java `User` será persistida na tabela `app_user`. |
| 054 | Tipos de identificadores | PKs e FKs usam `INT` com sinal padrão MySQL, sem `UNSIGNED`; PKs `AUTO_INCREMENT`; Java `Integer`. |
| 055 | Plataforma e collation | MySQL mínimo 8.0.16; InnoDB; charset `utf8mb4`; collation `utf8mb4_0900_ai_ci`, sujeita a teste no ambiente escolhido. |
| 056 | Exclusão e atualização | Todas as FKs usam `ON DELETE RESTRICT` e `ON UPDATE RESTRICT`; sem cascatas. Compra sem documento pode ser excluída por operação explícita após remoção controlada dos itens; documentos e tentativas são históricos. |
| 057 | E-mail | Normalizar e-mail para minúsculas antes de persistir/consultar; `UNIQUE` global em `app_user.email`. |
| 058 | CNPJ | `Merchant.taxId` é `VARCHAR(14)`, sem pontuação e em maiúsculas; aceita formato alfanumérico vigente; validação oficial na aplicação; `UNIQUE` quando informado. |
| 059 | Tipos numéricos e nulos | Monetários `DECIMAL(15,2)`, quantidades `DECIMAL(15,4)`, `PurchaseItem.unitPrice DECIMAL(15,10)`; Java `BigDecimal`; nunca `double`/`float`; opcional ausente é `NULL`, zero informado é zero. |
| 060 | Tabela `app_user` | `id INT`, `name VARCHAR(150)`, `email VARCHAR(254)`, `password_hash VARCHAR(255)`, `status VARCHAR(20)`, `created_at DATETIME(6)`; campos obrigatórios; status limitado a `ACTIVE`, `INACTIVE`, `BLOCKED`. |
| 061 | Tabela `purchase` | Confirmados campos e tipos: `purchase_date DATETIME(6)`, `total_amount DECIMAL(15,2)`, componentes opcionais `subtotal_amount`, `discount_amount`, `freight_amount`, `additional_amount` DECIMAL(15,2), `status VARCHAR(20)`, `created_at DATETIME(6)`, `user_id INT` obrigatório e `merchant_id INT` opcional. |
| 062 | Tabela `purchase_item` | `description VARCHAR(120)`, `quantity DECIMAL(15,4)`, `unit_of_measure VARCHAR(6)`, `unit_price DECIMAL(15,10)`, `gross_amount` e componentes monetários DECIMAL(15,2), `included_in_document_total BOOLEAN`, `purchase_id INT`; todos obrigatórios exceto os componentes monetários opcionais. Comprimentos devem ser conferidos contra o leiaute adotado. |
| 063 | Tabela `merchant` | `tax_id VARCHAR(14)`, `legal_name VARCHAR(200)`, `trade_name VARCHAR(200)`, `created_at DATETIME(6)`; tax_id/legal_name/trade_name opcionais individualmente, mas ao menos um deve estar preenchido. Comprimentos de nomes são provisórios. |
| 064 | Tabela `fiscal_document` | `access_key VARCHAR(44)`, `document_number VARCHAR(9)`, `series VARCHAR(3)`, `model VARCHAR(2)`, `issue_date DATETIME(6)`, `fiscal_status VARCHAR(20)`, `document_total_amount DECIMAL(15,2)`, `authorization_protocol VARCHAR(17) NULL`, `authorized_at DATETIME(6) NULL`, `xml_storage_reference VARCHAR(500)`, `xml_hash CHAR(64)`, `imported_at DATETIME(6)`, FKs e nulabilidade conforme `database-model.md`. A confirmação normativa do protocolo permanece pendente. |
| 065 | Tabela `import_attempt` | `file_name VARCHAR(255)`, `status VARCHAR(30)`, `error_message TEXT NULL`, `attempted_at DATETIME(6)`, `finished_at DATETIME(6) NULL`, `user_id INT`, `fiscal_document_id INT NULL`; obrigatoriedade conforme `database-model.md`. |
| 066 | Chaves e índices | PK `id` em todas as tabelas; UNIQUE e índices iniciais conforme `database-model.md`, incluindo índices de suporte às FKs compostas. |
| 067 | Integridade entre usuários | Criar `UNIQUE(id,user_id)` em `purchase` e `fiscal_document`; FKs compostas de documento para compra e de tentativa para documento asseguram proprietário igual. |
| 068 | Restrições CHECK | Restringir estados, modelo 65, chave de acesso, hash, campos numéricos, Merchant com ao menos um identificador e consistência temporal de `ImportAttempt`; MySQL 8.0.16+ é requisito. |
| 069 | Instantes e auditoria | Java `Instant`, MySQL `DATETIME(6)`, UTC com conversão explícita; campos temporais definidos por entidade em `database-model.md`; não adicionar `updated_at` no MVP. |
| 070 | Preservação fiscal de itens | Ler valores textuais diretamente para `BigDecimal`; preservar precisão e valores originais; não arredondar, recalcular ou substituir campos fiscais; cálculos futuros serão separados. |
| 071 | Campos ausentes no XML | Valor informado como zero permanece zero; campo opcional ausente vira `NULL`; campo vazio/inválido segue validação do leiaute e não recebe valor presumido. |
| 072 | `indTot` | `includedInDocumentTotal` é obrigatório e preserva explicitamente `indTot` (0/1); não presumir valor em caso de ausência ou invalidade. |
| 073 | Validações de NFC-e | Separar validação estrutural do XML da validação fiscal; sucesso técnico não implica autorização fiscal. |
| 074 | Situação fiscal | `AUTHORIZED` exige evidência fiscal adequada; sem evidência suficiente usar `UNKNOWN`. NFC-e cancelada é preservada; eventual compra associada recebe `CANCELED` e não é contabilizada como válida. |
| 075 | Armazenamento do XML | XML original fora do MySQL, armazenamento privado; DB guarda referência relativa gerada pela aplicação e hash SHA-256 minúsculo; não sobrescrever silenciosamente; `FiscalDocumentStorage` encapsula o mecanismo. |
| 076 | Organização do armazenamento | Armazenamento local inicial organizado por ano/mês, prefixo do hash e nome baseado no SHA-256; referência não expõe CNPJ ou chave de acesso. |
| 077 | Nome de arquivo | `ImportAttempt.fileName` é `VARCHAR(255) NOT NULL` e serve somente como metadado, nunca como caminho físico. |
| 078 | Estados técnicos | `ImportAttempt.status` limitado a `IN_PROGRESS`, `SUCCESS`, `DUPLICATE`, `INVALID_XML`, `PROCESSING_ERROR`, `STORAGE_ERROR`, `UNSUPPORTED_DOCUMENT`. |
| 079 | Estados fiscais e de compra | `FiscalDocument.fiscalStatus` limitado a `AUTHORIZED`, `CANCELED`, `DENIED`, `UNKNOWN`; `Purchase.status` limitado a `PENDING`, `VALID`, `CANCELED`. `User.status` limitado a `ACTIVE`, `INACTIVE`, `BLOCKED`. |
| 080 | Chave de acesso e hash | `access_key` deve conter exatamente 44 dígitos; `xml_hash` exatamente 64 caracteres hexadecimais minúsculos. Reforçar com validação na aplicação e `CHECK` no banco. |

