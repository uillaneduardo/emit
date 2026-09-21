# Requisitos iniciais

## 1. Perfis e acesso

- RF-001 A plataforma deve suportar no máximo dois perfis: Administrador e Técnico.
- RF-002 O Administrador deve configurar a plataforma.
- RF-003 O Técnico deve executar e preencher avaliações.
- RF-004 O controle de permissões deve permanecer simples, baseado nesses dois perfis.
- RF-005 A empresa/tenant deve ser isolada das demais empresas.
- RF-006 A configuração da instalação deve definir se Técnicos podem reabrir avaliações que já possuam laudo emitido; caso contrário, somente Administradores poderão fazê-lo.

## 2. Solicitantes

- RF-010 Cadastrar solicitantes pessoa física.
- RF-011 Cadastrar solicitantes pessoa jurídica/empresa.
- RF-012 Uma avaliação deve possuir um solicitante.
- RF-013 O solicitante pode ser identificado por dados adequados ao tipo de pessoa.
- RF-014 O cadastro do solicitante deve ser reutilizável em novas avaliações.

## 3. Equipamentos

- RF-020 Cadastrar categorias de equipamentos.
- RF-021 Cadastrar modelos.
- RF-022 Cadastrar aparelhos/equipamentos.
- RF-023 Identificar equipamento por atributos configuráveis e identificadores físicos, quando disponíveis.
- RF-024 Registrar componentes e periféricos associados ao equipamento.
- RF-025 Manter histórico de avaliações realizadas sobre o mesmo equipamento.

## 4. Avaliação — objeto central

- RF-030 Criar uma avaliação vinculada a um equipamento.
- RF-031 Vincular a avaliação a um solicitante.
- RF-032 Vincular a avaliação ao técnico responsável.
- RF-033 Permitir estruturas/templates configuráveis para orientar diferentes contextos de avaliação.
- RF-034 Permitir que uma avaliação seja iniciada, salva parcialmente, retomada e concluída posteriormente.
- RF-035 Permitir preenchimento progressivo sem exigir conclusão imediata de todos os campos.
- RF-036 Permitir campos obrigatórios somente quando definidos pela configuração aplicável.
- RF-037 Permitir observações e informações não previstas no formulário estruturado.
- RF-038 Permitir adicionar evidências durante qualquer etapa pertinente da avaliação.
- RF-039 Registrar data/hora de criação, alterações e conclusão.
- RF-040 Permitir reabrir uma avaliação concluída conforme as regras de integridade definidas neste documento.
- RF-041-A A avaliação deve possuir somente os estados `DRAFT`, `IN_PROGRESS`, `COMPLETED` e `CANCELLED`.
- RF-041-B A emissão de laudo não deve criar um estado adicional da avaliação; ela deve criar um marco de integridade e bloquear a edição normal.
- RF-041-C A reabertura deve ser tratada como uma operação/evento, retornando a avaliação à condição editável, e não como um estado permanente.
- RF-041 Preservar o estado e os dados relevantes da avaliação ao longo do tempo.
- RF-042 Possuir um campo textual opcional para o objetivo/contexto da avaliação.
- RF-043 O objetivo deve ser texto livre e não deve exigir cadastro de entidade, enum ou classificação própria.
- RF-044 Permitir objetivos como estado de entrada, estado de saída, diagnóstico, inspeção, manutenção ou orçamento sem limitar outros contextos.
- RF-045 Após a emissão de um laudo, bloquear a edição normal da avaliação.
- RF-046 Exigir uma ação explícita de reabertura para alterar uma avaliação que já originou um laudo.
- RF-047 Registrar na timeline a reabertura e as alterações posteriores.
- RF-049 Exigir um motivo informado pelo usuário em toda reabertura de avaliação que já possua laudo emitido, independentemente do perfil autorizado.
- RF-048 Uma alteração posterior nunca deve modificar o conteúdo de um laudo já emitido.

## 5. Estrutura configurável

- RF-050 O Administrador deve configurar categorias e modelos.
- RF-051 O Administrador deve configurar estruturas/templates de avaliação.
- RF-052 Templates devem possuir seções.
- RF-053 Seções devem possuir campos e testes.
- RF-054 Suportar campos de texto curto e longo, número, data/hora, seleção, múltipla seleção, booleano, escala, assinatura, componente e evidência.
- RF-055 Suportar testes técnicos estruturados com resultado, observação e evidência.
- RF-056 Templates devem ser versionados.
- RF-057 Uma avaliação deve preservar a versão do template utilizada, mesmo após alterações posteriores na configuração.

## 6. Laudo, evidências e publicação

- RF-060 Adicionar texto formatado.
- RF-061 Adicionar imagens.
- RF-062 Adicionar vídeos.
- RF-063 Adicionar documentos/arquivos.
- RF-064 Associar evidências à avaliação, seção, campo ou teste quando aplicável.
- RF-065 Permitir emitir um laudo a partir de uma avaliação concluída.
- RF-066 O laudo deve possuir versões numeradas quando uma avaliação for reaberta e emitida novamente.
- RF-067 Cada versão emitida deve preservar um snapshot imutável dos dados utilizados na emissão.
- RF-068 O laudo deve registrar o técnico responsável pela emissão, local, data e hora de emissão.
- RF-069 O laudo deve disponibilizar local para assinatura.
- RF-070 O laudo deve poder ser gerado em PDF.
- RF-071 O laudo deve poder ser consultado por uma URL pública.
- RF-072 O laudo deve possuir um QR Code para consulta/validação.
- RF-073 O QR Code deve apontar para um token público aleatório e não enumerável.
- RF-074 A consulta pública de uma versão deve apresentar o conteúdo correspondente àquela versão, não o estado atual mutável da avaliação.
- RF-075 Permitir controlar quais informações do laudo ficam públicas.
- RF-076 Permitir revogar o acesso público sem alterar o conteúdo do laudo emitido.
- RF-077 Não tratar assinatura no MVP como assinatura digital com certificado; o documento deve apenas disponibilizar o espaço/representação de assinatura definido pelo modelo do laudo.

## 7. Histórico e timeline

- RF-080 Registrar alterações relevantes realizadas na avaliação.
- RF-081 Registrar quem realizou cada alteração.
- RF-082 Registrar data/hora de cada alteração.
- RF-083 Registrar o tipo de alteração e os dados necessários para reconstruir o histórico.
- RF-084 Permitir visualizar uma timeline cronológica da avaliação.
- RF-085 Permitir identificar eventos como criação, preenchimento, alteração de respostas, inclusão/remoção de evidências, mudança de status, conclusão, emissão, reabertura e nova emissão de laudo.
- RF-086 O histórico deve ser append-only para eventos já registrados; não deve depender apenas do valor atual dos campos.
- RF-087 Avaliações que já tiveram laudo emitido devem manter histórico completo de qualquer reabertura e alteração posterior.
- RF-088 Operações que alteram o estado de integridade da avaliação devem registrar evento em transação com a alteração correspondente.

## 8. OS / referência externa

- RF-090 Permitir associar opcionalmente uma OS à avaliação.
- RF-091 Permitir registrar código da OS.
- RF-092 Permitir registrar URL de uma OS interna ou externa.
- RF-093 A ausência de OS não deve impedir uma avaliação.

## 9. Consulta pública

- RF-100 Permitir publicar uma versão específica do laudo por link público.
- RF-101 Gerar token público aleatório e não enumerável por publicação/versão, conforme o modelo de segurança adotado.
- RF-102 Gerar QR Code para a consulta pública da versão emitida.
- RF-103 Permitir controlar quais informações do laudo ficam públicas.
- RF-104 Permitir revogar o acesso público.
- RF-105 A consulta pública não deve exigir autenticação quando publicada.
- RF-106 O endereço público deve utilizar o domínio canônico persistido da instalação.

## 10. Auditoria da plataforma

- RF-110 Registrar ações administrativas relevantes.
- RF-111 Registrar alterações de configurações que afetem avaliações futuras.
- RF-112 Diferenciar histórico da avaliação de auditoria administrativa da plataforma.
- RF-113 Registrar ações de reabertura de avaliações que já possuam laudo emitido.

## 11. Configuração inicial da instalação

- RF-120 Na primeira execução após o deploy, o sistema deve detectar que a instalação ainda não foi inicializada.
- RF-121 A primeira execução deve apresentar um assistente de configuração inicial antes do uso normal da plataforma.
- RF-122 O assistente deve cadastrar os dados da empresa que emitirá os laudos.
- RF-123 O cadastro inicial deve contemplar endereço e telefone da empresa.
- RF-124 O assistente deve criar o primeiro usuário Administrador.
- RF-125 O assistente deve permitir cadastrar um usuário Técnico durante a instalação.
- RF-126 O cadastro do Técnico deve ser opcional e poderá ser realizado posteriormente pelo Administrador.
- RF-127 O assistente deve perguntar se a instalação iniciará com templates fornecidos pela própria instalação ou com configuração limpa.
- RF-128 A escolha de templates iniciais deve ser registrada na configuração da instalação.
- RF-129 O assistente deve impedir a criação de uma segunda configuração inicial concorrente.
- RF-130 A instalação somente deve ser considerada inicializada após a conclusão consistente de todas as etapas obrigatórias.

## 12. Exportação e importação de templates

- RF-140 O Administrador deve poder exportar templates configurados.
- RF-141 A exportação deve utilizar formato estruturado, versionado e independente do banco de dados.
- RF-142 O arquivo de exportação deve conter a versão do formato de exportação.
- RF-143 O arquivo deve conter a versão do sistema EMIT que gerou a exportação.
- RF-144 O arquivo deve conter metadados suficientes para identificar os templates e suas versões.
- RF-145 A importação deve validar a versão do formato antes de processar o conteúdo.
- RF-146 A importação deve verificar a compatibilidade entre a versão do sistema e a versão do formato.
- RF-147 O sistema deve diferenciar incompatibilidade obrigatória de compatibilidade parcial/migração possível.
- RF-148 A importação não deve sobrescrever silenciosamente templates existentes.
- RF-149 O processo deve apresentar ao Administrador o que será criado, atualizado ou ignorado antes da confirmação.
- RF-150 A exportação/importação deve preservar campos, seções, testes, opções, regras e configurações necessárias para reconstruir o template.
- RF-151 O formato deve permitir futura evolução através de migrações de versão.

## 13. Domínio público da instalação

- RF-160 O Administrador deve poder configurar o domínio público da instalação, por exemplo `emit.dominio.com.br`.
- RF-161 O domínio configurado deve ser utilizado como URL canônica da aplicação.
- RF-162 Links internos, links públicos de laudos, QR Codes, metadados e referências geradas pelo sistema devem utilizar o domínio configurado.
- RF-163 O domínio público configurado deve ser persistido na configuração da instalação, e não depender exclusivamente de variável de ambiente.
- RF-164 Variável de ambiente pode fornecer o domínio inicial/default durante o deploy, mas a configuração persistida deve ser a fonte de verdade após a inicialização.
- RF-165 A aplicação deve validar o formato do domínio informado.
- RF-166 A mudança do domínio deve refletir nas novas URLs geradas sem alterar tokens públicos existentes.
- RF-167 O sistema deve possuir mecanismo para informar/validar a origem pública permitida e evitar uso indevido do Host header.

## Não funcionais

- RNF-001 Responsivo, com prioridade mobile.
- RNF-002 PWA instalável.
- RNF-003 Docker Compose.
- RNF-004 Credenciais por variáveis de ambiente.
- RNF-005 PostgreSQL.
- RNF-006 Storage S3-compatible.
- RNF-007 Isolamento por empresa.
- RNF-008 Tokens públicos aleatórios.
- RNF-009 Validação de uploads.
- RNF-010 Preparação para offline futuro.
- RNF-011 Histórico, timeline, bloqueio e versões de laudo devem permanecer consistentes mesmo com preenchimento progressivo e reabertura autorizada.
