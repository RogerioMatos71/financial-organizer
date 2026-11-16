# Segurança e proteção de dados

## 1. Princípios

Segurança e privacidade são requisitos desde a concepção. O sistema deverá proteger dados pessoais, financeiros e fiscais contra acesso, alteração, perda ou divulgação não autorizados.

## 2. Autenticação e credenciais

- Nunca armazenar senhas em texto puro.
- Utilizar algoritmo de hash de senha apropriado, como Argon2id ou BCrypt, com parâmetros atuais e biblioteca mantida.
- Não implementar criptografia ou hash de senha próprio.
- Segredos e credenciais não poderão ser versionados no Git.
- A aplicação deverá usar usuário MySQL próprio, com privilégio mínimo; não utilizar root.
- Recuperação de senha e sessões deverão ser projetadas com controles seguros antes de disponibilizar autenticação real.

## 3. Autorização e isolamento

- Cada `Purchase`, `FiscalDocument` e `ImportAttempt` pertence a um `User`.
- Toda operação de leitura, alteração, exportação ou exclusão deverá validar o proprietário.
- Não confiar em `user_id` enviado pelo cliente como prova de autorização.
- Reforçar integridade por FKs compostas quando aplicável.
- `Merchant` é compartilhado como cadastro de referência; isso não concede acesso a compras ou documentos de outros usuários.

## 4. Proteção de XML

- Armazenar XML fora de diretórios públicos.
- Gerar referências e nomes internos seguros; nunca usar caminho arbitrário fornecido pelo usuário.
- Validar tamanho, formato, codificação e estrutura antes do processamento.
- Configurar parser XML para desabilitar DTDs e entidades externas, prevenindo XXE e acessos externos.
- Não resolver URLs ou recursos externos a partir do XML.
- Preservar bytes originais e calcular SHA-256 para verificação de integridade.
- O hash não prova autenticidade fiscal nem substitui validação de autorização.
- Controlar acesso aos arquivos pelo proprietário do documento.

## 5. Banco de dados e aplicação

- Usar consultas parametrizadas e APIs de persistência seguras.
- Validar entradas e limitar tamanho de arquivos e campos.
- Evitar registrar senhas, tokens, XML completo ou dados pessoais desnecessários em logs.
- Restringir exposição da porta MySQL; no ambiente local, expor apenas o necessário.
- Separar credenciais por ambiente e manter permissões mínimas.

## 6. Transporte e implantação

HTTPS será obrigatório antes de disponibilizar acesso pela rede ou para usuários externos. Não considerar a execução local inicial como justificativa para publicar serviços sem proteção.

## 7. Backup e recuperação

- Criar backups periódicos do MySQL e dos XMLs.
- Armazenar cópias separadas do ambiente principal.
- Proteger backups com controles de acesso e, quando apropriado, criptografia.
- Testar restauração periodicamente.
- Tratar banco e arquivos como conjunto: restaurar somente um deles pode produzir referências quebradas ou documentos ausentes.

## 8. LGPD e retenção

Definir finalidade, necessidade, acesso, retenção e atendimento aos direitos dos titulares. Avaliar bases legais e obrigações de conservação fiscal antes de excluir ou anonimizar documentos. A aplicação deverá manter apenas dados necessários e evitar exposição em logs e relatórios.

## 9. Incidentes e evolução

Definir procedimento para identificar, conter, avaliar e tratar incidentes. Monitoramento avançado e infraestrutura específica de nuvem ficam fora do MVP, mas não dispensam controles básicos, registros operacionais e processo de backup.

## 10. Critério de conclusão

Nenhuma funcionalidade será considerada concluída sem atender os controles de segurança aplicáveis e possuir testes correspondentes para seus fluxos críticos.
