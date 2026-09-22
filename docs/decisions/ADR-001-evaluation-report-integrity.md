# ADR-001 — Integridade da avaliação e dos laudos

- **Status:** Aceito
- **Data:** 2026-09-21

## Contexto

A Avaliação é o registro técnico de trabalho do EMIT. Ela pode ser preenchida progressivamente, receber alterações e manter uma timeline.

O Laudo, porém, é um documento emitido a partir de uma avaliação. Depois de emitido, seu conteúdo precisa continuar representando exatamente o que foi apresentado no momento da emissão.

Permitir que o Laudo consulte diretamente o estado atual da Avaliação criaria o risco de um documento já emitido mudar indiretamente quando a avaliação fosse alterada.

Também é necessário permitir correções reais após a emissão sem perder rastreabilidade.

## Decisão

Adotar quatro regras complementares:

1. **A avaliação é mutável enquanto não estiver protegida por uma emissão.**
2. **A emissão de um laudo cria um marco de integridade e bloqueia a edição normal da avaliação.**
3. **Para alterar uma avaliação que já originou um laudo, é necessária uma reabertura explícita e autorizada.**
4. **Cada emissão produz uma versão imutável do laudo com snapshot próprio.**

O histórico da avaliação é mantido por eventos append-only.

## Fluxo

```text
Avaliação
   |
   | concluir + emitir
   v
Laudo v1 [imutável]
   |
   | reabrir
   v
Avaliação [editável novamente]
   |
   | alterações auditadas
   v
concluir + emitir
   |
   v
Laudo v2 [imutável]
```

## Estado e reabertura

A avaliação mantém um conjunto pequeno de estados:

- `DRAFT`
- `IN_PROGRESS`
- `COMPLETED`
- `CANCELLED`

A emissão do laudo não cria um estado `REPORT_ISSUED`. O bloqueio da avaliação após emissão é uma condição de integridade separada do status, podendo ser representado por um atributo como `lockedAt`.

Também não será criado um estado permanente `REOPENED`. Reabertura é uma operação explícita registrada na timeline que torna novamente editável uma avaliação já protegida, normalmente retornando-a para `IN_PROGRESS`.

## Reabertura

Não será criada uma senha exclusiva para cada avaliação.

A operação deve usar a autenticação e autorização do usuário do EMIT. A aplicação deve registrar:

- usuário que reabriu;
- data/hora;
- motivo, quando exigido pela política;
- laudos já emitidos relacionados à avaliação.

As alterações realizadas após a reabertura continuam sendo registradas na timeline.

## Snapshot do laudo
### Evidências e arquivos

As evidências fazem parte do processo técnico da avaliação, mas sua inclusão no laudo é uma decisão de composição documental.

O MVP deve aceitar imagem, áudio e vídeo como evidências e permitir visualização/reprodução básica diretamente na aplicação. Para imagem, isso significa visualização; para áudio e vídeo, controles simples de reprodução. Além do upload, o técnico deve poder iniciar uma gravação de áudio ou vídeo diretamente da tela da avaliação e, após confirmar o resultado, vinculá-lo automaticamente à avaliação. A captura depende das permissões e capacidades do navegador; falhas de permissão ou suporte não devem bloquear o restante da avaliação. Não faz parte desta decisão implementar recursos avançados de player, edição, transcodificação ou streaming adaptativo.

Uma evidência utilizada na versão emitida deve ser preservada por referência de armazenamento imutável. Isso impede que alterações posteriores no arquivo original modifiquem o conteúdo de um laudo histórico.

A seleção de evidências para o laudo deve ocorrer na composição da versão emitida. Evidências não selecionadas podem continuar existindo apenas no contexto interno da avaliação.


O snapshot representa a composição documental relevante utilizada na emissão. O Laudo é uma representação da Avaliação, combinando as informações registradas pelo técnico e outras informações relevantes em uma apresentação amigável e padronizada.

Ele permite que:

- o PDF histórico permaneça consistente;
- a consulta pública continue mostrando a versão emitida;
- a emissão de uma nova versão não altere versões anteriores;
- o sistema audite a relação entre avaliação e documento emitido.

O snapshot não é um dump do banco e não precisa replicar todas as tabelas do domínio. Deve preservar os dados e a composição necessários para reproduzir a versão emitida, incluindo, conforme aplicável, emissor, solicitante, equipamento, objetivo, local, técnico, resultados, testes, observações, conclusão, evidências, textos/rótulos, template/versão, identidade visual e metadados de emissão. Evidências que integram o documento devem ter referências de armazenamento imutáveis.

A mesma versão pode ser entregue digitalmente por URL/QR Code ou em PDF para impressão A4. A consulta pública deve representar o laudo emitido e aplicar minimização de dados pessoais e sensíveis, sem expor automaticamente todo o conteúdo interno da avaliação.

## QR Code e consulta pública

Cada versão publicada do laudo deve possuir uma forma de consulta pública baseada em token aleatório e não enumerável.

O QR Code aponta para essa consulta.

O QR Code não deve conter o conteúdo do laudo.

Uma consulta à versão 1 deve continuar apresentando a versão 1 mesmo quando existir uma versão 2.

## Consequências

### Positivas

- preserva a integridade documental;
- mantém rastreabilidade de correções;
- permite corrigir avaliações sem apagar o histórico;
- permite múltiplas emissões sem sobrescrever documentos anteriores;
- simplifica a explicação do comportamento do QR Code.

### Custos

- o domínio precisa armazenar versões de laudo;
- será necessário definir um formato de snapshot;
- a aplicação precisa implementar o fluxo de reabertura;
- PDFs e publicações precisam estar associados à versão correta.

## Fora do escopo desta decisão

- assinatura digital com certificado;
- validade jurídica específica do documento;
- workflow de aprovação complexo;
- múltiplos níveis de revisão;
- retenção legal específica por tipo de documento.
