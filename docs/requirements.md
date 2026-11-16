# Requisitos do produto

## 1. Visão

A Plataforma de Organização Financeira pretende organizar compras e despesas com o mínimo possível de digitação manual. O núcleo do produto atenderá pessoas físicas e empresas, sem manter produtos separados para cada público.

## 2. Objetivo do primeiro incremento

O primeiro incremento deverá importar XML de NFC-e (modelo 65), preservar o documento original, extrair os dados necessários e persistir o documento e, quando aplicável, a compra e seus itens.

## 3. Requisitos funcionais

### RF-01 — Usuários

O sistema deverá possuir a entidade `User`. Cada compra, documento fiscal e tentativa de importação deverá pertencer a um usuário. O e-mail será único globalmente.

### RF-02 — Estabelecimentos

O sistema deverá manter estabelecimentos em `Merchant`, compartilhados como cadastro de referência. No primeiro incremento, `taxId` será o CNPJ brasileiro. A associação de estabelecimento a compra ou documento fiscal será opcional.

### RF-03 — Importação de NFC-e

O sistema deverá aceitar XML de NFC-e, modelo 65. Outros modelos serão identificados como não suportados, sem processamento específico nesta versão.

### RF-04 — Preservação do XML

O XML original deverá ser preservado fora do banco relacional. O banco armazenará a referência ao arquivo e seu hash SHA-256.

### RF-05 — Extração de dados

O sistema deverá extrair os dados fiscais e comerciais necessários ao modelo de domínio, preservando os valores fornecidos no XML. O sistema não deverá substituir valores fiscais de origem por valores recalculados.

### RF-06 — Duplicidade

A mesma chave de acesso não poderá ser registrada duas vezes para o mesmo usuário. Uma nova tentativa deverá ser registrada como `DUPLICATE` e poderá referenciar o documento já existente. Usuários diferentes poderão importar a mesma chave.

### RF-07 — Situação fiscal

A situação fiscal deverá ser separada do resultado técnico da importação. Os estados inicialmente previstos são `AUTHORIZED`, `CANCELED`, `DENIED` e `UNKNOWN`. A autorização não poderá ser inferida apenas da leitura bem-sucedida do XML.

### RF-08 — Compras

Uma compra poderá existir sem documento fiscal e sem itens. Uma NFC-e autorizada poderá originar uma compra. Documento cancelado deverá ser preservado, mas não contabilizado como compra válida.

### RF-09 — Itens

Uma compra poderá possuir zero ou vários itens. Os itens originados de NFC-e deverão preservar descrição, unidade, quantidade, preço e valores relevantes do XML.

### RF-10 — Histórico de importação

Cada operação de importação deverá gerar um `ImportAttempt`, inicialmente em `IN_PROGRESS` e depois atualizado para um resultado final. Tentativas sem documento identificado também deverão ser registradas.

### RF-11 — Consistência entre arquivo e banco

A gravação do arquivo e a persistência no MySQL deverão seguir a estratégia de compensação definida na DEC-025 e DEC-050. A aplicação deverá tentar remover somente o arquivo criado pela operação que falhou. Falha na compensação deverá ser registrada para limpeza posterior.

### RF-12 — Merchant

Um Merchant identificado pelo CNPJ deverá ser reutilizado. Informações ausentes poderão ser preenchidas por novas importações; informações já preenchidas não serão sobrescritas automaticamente por valores divergentes.

## 4. Requisitos não funcionais

### RNF-01 — Segurança e privacidade

O sistema deverá proteger credenciais, XMLs e dados pessoais, aplicar autorização por proprietário e considerar os princípios e obrigações aplicáveis da LGPD.

### RNF-02 — Integridade

As regras críticas de propriedade, unicidade e relacionamento deverão ser validadas na aplicação e, quando possível, reforçadas por restrições no banco.

### RNF-03 — Testabilidade

O desenvolvimento deverá ser acompanhado por testes automatizados. Regras de negócio críticas e correções de defeitos deverão possuir testes correspondentes.

### RNF-04 — Rastreabilidade

O sistema deverá preservar o XML original, seu hash e o histórico técnico das tentativas de importação.

### RNF-05 — Evolução incremental

A solução deverá permitir adicionar novos processadores de documentos e novas implementações de armazenamento sem antecipar integrações que não fazem parte do primeiro incremento.

### RNF-06 — Recuperabilidade

Deverão existir backups do banco e dos XMLs, armazenados separadamente do ambiente principal, e procedimentos testados de restauração.

## 5. Fora do escopo inicial

- Processamento de NF-e modelo 55, CF-e-SAT modelo 59 e ECF.
- Classificação ou normalização automática de produtos.
- Integração com adquirentes, bancos, cartões ou Open Finance.
- Recuperação automática de NFC-e por CPF ou consulta automática a portais estaduais.
- Armazenamento em nuvem.
- Compartilhamento de compras entre familiares ou múltiplos perfis financeiros.
- Painéis avançados de métricas e monitoramento.
- Interface completa de usuário e autenticação pronta para produção, embora o domínio já contemple usuário e segurança.

## 6. Critério geral de conclusão

Uma funcionalidade não será considerada concluída apenas por compilar ou executar. Ela deverá atender às regras funcionais, aos testes aplicáveis e aos requisitos de segurança correspondentes.
