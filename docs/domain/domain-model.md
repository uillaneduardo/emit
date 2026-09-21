# Modelo de domínio inicial

```text
Company
  +-- User
  +-- Customer
  +-- Category -> Model -> Template
  +-- Equipment -> Component
  +-- WorkOrder -> Equipment -> Evaluation
                              +-- FieldResponse
                              +-- TestResult
                              +-- Attachment
                              +-- PublicAccess
                              +-- AuditEvent
```

## Conceitos

Company: tenant da aplicação.

User: usuário autenticado, inicialmente administrador ou técnico.

Customer: pessoa física ou jurídica responsável pelo equipamento.

Equipment: objeto físico atendido, com categoria, modelo, identificadores e componentes.

WorkOrder: contexto operacional do atendimento; pode usar código e/ou URL de OS interna ou externa.

Template: modelo configurável de avaliação.

Evaluation: execução de um template sobre um equipamento. Tipos iniciais: entrada, diagnóstico, saída, inspeção e personalizado.

FieldResponse: resposta de campo dinâmico.

TestResult: resultado estruturado de teste técnico.

Attachment: imagem, vídeo ou documento.

PublicAccess: token público para consulta e QR Code.

AuditEvent: trilha de auditoria.

## Regra fundamental

Checklist não será uma estrutura fixa no frontend. Ele será composto por Template -> Sections -> Fields/Tests. A avaliação instancia essa configuração e preserva suas respostas, mesmo quando o template for alterado posteriormente.
