# Modelo de domínio

Este documento descreve as entidades, responsabilidades e relacionamentos do primeiro incremento. Os tipos físicos e constraints estão detalhados em [database-model.md](database-model.md).

## 1. Entidades

### User

Representa o proprietário dos dados.

Atributos: `id`, `name`, `email`, `passwordHash`, `status`, `createdAt`.

Status: `ACTIVE`, `INACTIVE`, `BLOCKED`. O e-mail é único globalmente e normalizado para minúsculas antes de persistir/consultar. Senhas nunca serão armazenadas em texto puro; `passwordHash` comporta o algoritmo de hash seguro escolhido.

### Merchant

Representa um estabelecimento comercial compartilhado como cadastro de referência, sem vínculo direto com `User`.

Atributos: `id`, `taxId`, `legalName`, `tradeName`, `createdAt`.

No primeiro incremento, `taxId` representa CNPJ brasileiro, armazenado como texto sem pontuação e em maiúsculas; o campo aceita o formato alfanumérico oficial vigente. O CNPJ é opcional, mas único quando informado. Deve existir ao menos uma informação entre CNPJ, razão social e nome fantasia. Se o CNPJ for informado, o registro será localizado e reutilizado; campos ausentes poderão ser preenchidos, mas divergências não substituirão automaticamente valores existentes.

### Purchase

Representa uma compra do usuário.

Atributos: `id`, `purchaseDate`, `totalAmount`, `subtotalAmount`, `discountAmount`, `freightAmount`, `additionalAmount`, `status`, `createdAt`, `user`, `merchant`.

`totalAmount` é obrigatório. Os componentes são opcionais e permanecem ausentes (`NULL`) quando não informados na origem. `merchant` é opcional; `user` é obrigatório. `purchaseDate` representa o instante da compra; `createdAt`, o instante de cadastro no sistema.

Status: `PENDING`, `VALID`, `CANCELED`. Uma compra pode existir sem documento fiscal e sem itens. O total de origem não será substituído pela soma recalculada dos itens.

### PurchaseItem

Representa um item da compra.

Atributos: `id`, `description`, `quantity`, `unitOfMeasure`, `unitPrice`, `grossAmount`, `discountAmount`, `freightAmount`, `insuranceAmount`, `additionalAmount`, `includedInDocumentTotal`, `purchase`.

`description` preserva `xProd`; `quantity` preserva `qCom`; `unitOfMeasure` preserva o código `uCom`; `unitPrice` preserva a precisão de `vUnCom`; `grossAmount` corresponde a `vProd`. Os componentes opcionais correspondem a `vDesc`, `vFrete`, `vSeg` e `vOutro`. `includedInDocumentTotal` preserva `indTot` e é obrigatório.

Quantidade e preço/valores usam `BigDecimal` em Java. Não usar `double`/`float`, arredondar ou recalcular os valores recebidos do XML. Um campo monetário ausente é `NULL`; um valor explicitamente zero permanece zero.

### FiscalDocument

Representa o documento fiscal eletrônico.

Atributos: `id`, `accessKey`, `documentNumber`, `series`, `model`, `issueDate`, `fiscalStatus`, `documentTotalAmount`, `authorizationProtocol`, `authorizedAt`, `xmlStorageReference`, `xmlHash`, `importedAt`, `user`, `merchant`, `purchase`.

`user` é obrigatório. `merchant` e `purchase` são opcionais. A unicidade da chave de acesso é por usuário. O primeiro incremento aceita exclusivamente NFC-e modelo `65`. Número e série são armazenados como texto para preservar zeros à esquerda. O XML original é armazenado fora do MySQL; `xmlHash` é SHA-256 calculado sobre os bytes originais.

Status fiscal: `AUTHORIZED`, `CANCELED`, `DENIED`, `UNKNOWN`. Uma importação tecnicamente bem-sucedida não prova, por si só, que o documento está autorizado. `AUTHORIZED` exige evidência fiscal adequada; sem evidência suficiente, usar `UNKNOWN`.

### ImportAttempt

Representa uma operação individual de importação.

Atributos: `id`, `fileName`, `status`, `errorMessage`, `attemptedAt`, `finishedAt`, `user`, `fiscalDocument`.

`user` é obrigatório. `fiscalDocument` é opcional. `finishedAt` fica nulo enquanto a tentativa está em andamento. A duração é calculada por `finishedAt - attemptedAt`, não armazenada.

Status: `IN_PROGRESS`, `SUCCESS`, `DUPLICATE`, `INVALID_XML`, `PROCESSING_ERROR`, `STORAGE_ERROR`, `UNSUPPORTED_DOCUMENT`.

## 2. Relacionamentos e cardinalidades

- `User 1 — 0..N Purchase`: cada compra pertence a um usuário.
- `User 1 — 0..N FiscalDocument`: cada documento pertence a um usuário.
- `User 1 — 0..N ImportAttempt`: cada tentativa pertence a um usuário.
- `Merchant 0..1 — 0..N Purchase`: uma compra pode não ter estabelecimento; um estabelecimento pode aparecer em várias compras.
- `Merchant 0..1 — 0..N FiscalDocument`: um documento pode não ter estabelecimento; um estabelecimento pode aparecer em vários documentos.
- `Purchase 1 — 0..N PurchaseItem`: cada item pertence a uma compra; uma compra pode não possuir itens.
- `Purchase 0..1 — 0..1 FiscalDocument`: a associação é opcional e um-para-um.
- `FiscalDocument 0..1 — 0..N ImportAttempt`: uma tentativa pode não identificar documento; várias tentativas podem referenciar o mesmo documento.

## 3. Regras de domínio

1. Uma NFC-e duplicada para o mesmo usuário não cria novo `FiscalDocument` nem nova `Purchase`; a tentativa é registrada como `DUPLICATE` e pode referenciar o documento existente.
2. Usuários diferentes podem importar a mesma chave de acesso.
3. Compra e documento fiscal associados devem pertencer ao mesmo usuário; o banco reforça essa regra por FK composta.
4. Tentativa e documento fiscal referenciado devem pertencer ao mesmo usuário; o banco reforça essa regra por FK composta.
5. Documento fiscal cancelado é preservado, mas não deve ser contabilizado como compra válida. Se houver `Purchase` associada, seu status será `CANCELED`.
6. Situação fiscal e resultado técnico de importação são conceitos separados.
7. `Merchant` é compartilhado; compras, documentos e tentativas são sempre individualizados por usuário.
8. XML original e hash são preservados para rastreabilidade e verificação de integridade.
9. Exclusões físicas seguem as regras restritivas descritas no modelo de banco; documentos fiscais e tentativas são históricos e não são excluídos pelo fluxo normal.
