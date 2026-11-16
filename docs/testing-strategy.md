# Estratégia de testes

## 1. Objetivo

Os testes automatizados acompanharão o desenvolvimento para verificar regras de negócio, integração entre componentes e prevenção de regressões.

## 2. Ferramentas

- JUnit 5 como base.
- Mockito quando for útil isolar dependências.
- Spring Boot Test para testes de integração com o contexto Spring.
- Testcontainers para testes que precisem de uma instância isolada de MySQL, quando essa etapa for introduzida.

As ferramentas serão adicionadas conforme a necessidade, sem criar infraestrutura de teste desnecessária antecipadamente.

## 3. Níveis de teste

### Unitários

Testar regras e componentes isoladamente: validação de chave de acesso, normalização de CNPJ, interpretação de status, regras de associação e mapeamento de valores fiscais.

### Integração

Testar repositórios JPA, restrições do MySQL, transações e integração entre serviços. Usar banco isolado, nunca o banco de desenvolvimento com dados reais.

### Funcionais

Testar fluxos completos, como importar uma NFC-e válida, importar documento cancelado, receber XML inválido e tentar importar uma chave duplicada.

### Regressão

Toda correção de defeito relevante deverá acrescentar um teste que reproduza o problema, evitando que volte a ocorrer.

## 4. Casos críticos mínimos

- Mesmo usuário importa a mesma chave duas vezes: um documento e uma tentativa DUPLICATE.
- Usuários diferentes importam a mesma chave: registros independentes por usuário.
- Tentativa sem documento identificado permanece registrada.
- Documento cancelado é preservado e não contabilizado como compra válida.
- Compra e documento associados pertencem ao mesmo usuário.
- Tentativa não pode referenciar documento de outro usuário.
- XML original permanece idêntico ao conteúdo recebido; hash SHA-256 confere.
- Falha de persistência após salvar XML aciona compensação; falha na compensação é registrada.
- XML malicioso ou inválido não deve provocar acesso externo ou leitura arbitrária.
- Campos decimais preservam precisão e valores de origem.

## 5. Critério de execução

Executar a suíte aplicável antes de integrar alterações e antes de considerar uma funcionalidade concluída. Testes devem ser determinísticos, isolados e não depender de serviços externos reais.

## 6. Cobertura

Não será adotada uma meta percentual arbitrária como substituto de qualidade. A prioridade será cobrir regras de domínio, segurança, integridade, fluxos críticos e correções de defeitos. Métricas de cobertura poderão ser usadas como indicador auxiliar.
