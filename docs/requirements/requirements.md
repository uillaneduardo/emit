# Requisitos iniciais

## 1. Perfis e acesso

- RF-001 A plataforma deve suportar no máximo dois perfis: Administrador e Técnico.
- RF-002 O Administrador deve configurar a plataforma.
- RF-003 O Técnico deve executar e preencher avaliações.
- RF-004 O controle de permissões deve permanecer simples, baseado nesses dois perfis.
- RF-005 A empresa/tenant deve ser isolada das demais empresas.

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
- RF-033 Permitir tipos de avaliação configuráveis, com tipos iniciais como diagnóstico, inspeção, entrada e saída.
- RF-034 Permitir que uma avaliação seja iniciada, salva parcialmente, retomada e concluída posteriormente.
- RF-035 Permitir preenchimento progressivo sem exigir conclusão imediata de todos os campos.
- RF-036 Permitir campos obrigatórios somente quando definidos pela configuração aplicável.
- RF-037 Permitir observações e informações não previstas no formulário estruturado.
- RF-038 Permitir adicionar evidências durante qualquer etapa pertinente da avaliação.
- RF-039 Registrar data/hora de criação, alterações e conclusão.
- RF-040 Permitir reabrir ou revisar uma avaliação conforme regra definida pelo Administrador.
- RF-041 Preservar o estado e os dados relevantes da avaliação ao longo do tempo.

## 5. Estrutura configurável

- RF-050 O Administrador deve configurar categorias e modelos.
- RF-051 O Administrador deve configurar estruturas/templates de avaliação.
- RF-052 Templates devem possuir seções.
- RF-053 Seções devem possuir campos e testes.
- RF-054 Suportar campos de texto curto e longo, número, data/hora, seleção, múltipla seleção, booleano, escala, assinatura, componente e evidência.
- RF-055 Suportar testes técnicos estruturados com resultado, observação e evidência.
- RF-056 Templates devem ser versionados.
- RF-057 Uma avaliação deve preservar a versão do template utilizada, mesmo após alterações posteriores na configuração.

## 6. Evidências e laudo

- RF-060 Adicionar texto formatado.
- RF-061 Adicionar imagens.
- RF-062 Adicionar vídeos.
- RF-063 Adicionar documentos/arquivos.
- RF-064 Associar evidências à avaliação, seção, campo ou teste quando aplicável.
- RF-065 Gerar/organizar o conteúdo da avaliação em formato de laudo.
- RF-066 Permitir identificar claramente situação, conclusão e observações técnicas do laudo.
- RF-067 Preparar o domínio para futura geração de PDF e assinatura digital.

## 7. Histórico e timeline

- RF-070 Registrar alterações relevantes realizadas na avaliação.
- RF-071 Registrar quem realizou cada alteração.
- RF-072 Registrar data/hora de cada alteração.
- RF-073 Registrar o tipo de alteração e os dados necessários para reconstruir o histórico.
- RF-074 Permitir visualizar uma timeline cronológica da avaliação.
- RF-075 Permitir identificar eventos como criação, preenchimento, alteração de respostas, inclusão/remoção de evidências, mudança de status e conclusão.
- RF-076 O histórico deve ser append-only para eventos já registrados; não deve depender apenas do valor atual dos campos.
- RF-077 Avaliações concluídas devem continuar com histórico completo de alterações posteriores, caso alterações sejam permitidas.

## 8. OS / referência externa

- RF-080 Permitir associar opcionalmente uma OS à avaliação.
- RF-081 Permitir registrar código da OS.
- RF-082 Permitir registrar URL de uma OS interna ou externa.
- RF-083 A ausência de OS não deve impedir uma avaliação.

## 9. Consulta pública

- RF-090 Permitir publicar uma avaliação/laudo por link público.
- RF-091 Gerar token público aleatório e não enumerável.
- RF-092 Gerar QR Code para a consulta pública.
- RF-093 Permitir controlar quais informações da avaliação ficam públicas.
- RF-094 Permitir revogar o acesso público.
- RF-095 A consulta pública não deve exigir autenticação quando publicada.

## 10. Auditoria da plataforma

- RF-100 Registrar ações administrativas relevantes.
- RF-101 Registrar alterações de configurações que afetem avaliações futuras.
- RF-102 Diferenciar histórico da avaliação de auditoria administrativa da plataforma.

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
- RNF-011 Histórico e timeline devem permanecer consistentes mesmo com preenchimento progressivo.
