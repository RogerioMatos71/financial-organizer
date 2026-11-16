# Arquitetura

## 1. Diretrizes

A arquitetura deverá manter responsabilidades bem definidas, permitir evolução incremental e evitar abstrações genéricas sem necessidade concreta.

## 2. Tecnologias

- Java 21
- Spring Boot
- Maven
- Spring Data JPA
- MySQL
- Docker Compose para o banco no ambiente de desenvolvimento
- JUnit 5 e ferramentas complementares de teste

A aplicação será executada localmente pela IDE no início. O banco será executado em contêiner. A conteinerização da aplicação poderá ser avaliada posteriormente.

## 3. Componentes lógicos previstos

### Importação

Responsável por receber o arquivo, registrar `ImportAttempt`, validar o tipo de documento, coordenar o processamento e atualizar o resultado da tentativa.

### Processador de NFC-e

Componente específico para o modelo 65. Não haverá, no primeiro incremento, processadores para NF-e, CF-e ou ECF. Uma abstração pequena poderá permitir a inclusão futura de outros processadores.

### Domínio de compras e documentos

Responsável pelas regras de `Purchase`, `PurchaseItem`, `FiscalDocument`, `Merchant` e sua consistência.

### Persistência

Spring Data JPA e MySQL serão usados para dados estruturados. As regras de propriedade e unicidade deverão ser reforçadas por restrições relacionais.

### Armazenamento de XML

A interface `FiscalDocumentStorage` definirá operações essenciais de armazenamento e recuperação. A primeira implementação será local, em disco, fora de diretórios públicos. Não haverá integração com provedores de nuvem no MVP.

## 4. Fluxo de importação proposto

1. Receber o XML e identificar o usuário responsável.
2. Criar `ImportAttempt` com status `IN_PROGRESS`.
3. Validar o conteúdo XML com parser seguro e verificar se o documento é NFC-e modelo 65.
4. Extrair e validar os dados necessários, incluindo chave de acesso e situação fiscal quando verificável.
5. Verificar duplicidade por `(user_id, access_key)`.
6. Preservar o arquivo original no armazenamento local e calcular SHA-256 sobre os bytes originais.
7. Persistir `FiscalDocument`, `Merchant` quando identificável e `Purchase`/`PurchaseItem` quando aplicável, respeitando as regras fiscais.
8. Finalizar `ImportAttempt` com o resultado correspondente e `finishedAt`.

O sequenciamento exato entre armazenamento e transação de banco deverá respeitar a estratégia de compensação: se a persistência falhar após a gravação do arquivo, tentar remover somente o arquivo recém-criado. Se a remoção falhar, registrar a ocorrência para limpeza posterior.

## 5. Transações e consistência

A transação MySQL protege as alterações relacionais. O sistema de arquivos não participa da transação do banco; por isso será utilizada compensação explícita. A rotina administrativa de verificação de arquivos órfãos será simples no MVP.

## 6. Datas e horários

O modelo inicial armazenará instantes normalizados em UTC. No Java, a representação de domínio será `Instant`. No MySQL, a decisão prevê `DATETIME(6)` com valores UTC, sem guardar o deslocamento/fuso original. A implementação deverá configurar explicitamente a conversão JDBC/JPA; `DATETIME` não converte fuso automaticamente.

## 7. Segurança arquitetural

A autorização deverá ser aplicada nas operações de serviço e reforçada por chaves e restrições no banco quando possível. O acesso a XMLs será mediado pelo componente de armazenamento, sem exposição por caminho público ou nome fornecido diretamente pelo usuário.

## 8. Evolução futura

Pontos de extensão previstos, mas não implementados inicialmente:

- Novos processadores de documentos fiscais.
- Outras fontes de compras e transações.
- Implementações de `FiscalDocumentStorage` em nuvem.
- Execução da aplicação em contêiner.
- Autenticação e interface completas.

A arquitetura não deverá incluir dependências de provedores específicos antes de existir necessidade real.
