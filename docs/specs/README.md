# Especificações (SPECs)

Toda funcionalidade é implementada **somente** a partir de uma SPEC com status `Aprovada` (ver [ADR-0001](../adr/0001-spec-driven-development-e-adrs.md)).

## Como criar uma SPEC

1. Copie `_template.md` para `SPEC-NNN-nome-em-kebab-case.md`.
2. Preencha com status `Rascunho`. A IA pode ajudar a levantar casos de erro e borda.
3. Revise e mude para `Aprovada` antes de iniciar a implementação.
4. Ao concluir, mude para `Implementada` e atualize o histórico.

## Catálogo de funcionalidades

| SPEC     | Funcionalidade                                                      | Pilar | Status    |
| -------- | ------------------------------------------------------------------- | ----- | --------- |
| SPEC-000 | Dados fictícios e camada de acesso a dados                          | 1     | Aprovada  |
| SPEC-001 | Listagem de documentos com paginação                                | 2     | Planejada |
| SPEC-002 | Busca e filtros (tipo, status, responsável)                         | 2     | Planejada |
| SPEC-003 | Detalhes do documento                                               | 2     | Planejada |
| SPEC-004 | Alteração de status e gestão de pendências                          | 2     | Planejada |
| SPEC-005 | Assistente: resumo progressivo com streaming e cancelamento         | 3     | Planejada |
| SPEC-006 | Assistente: saída estruturada (resumo, pendências, próximos passos) | 3     | Planejada |
| SPEC-007 | Assistente: tool calling com confirmação humana                     | 3     | Planejada |
| SPEC-008 | Assistente: limites, conteúdo não confiável e renderização segura   | 3     | Planejada |
| SPEC-009 | Observabilidade, limites de uso e flag do assistente                | 5     | Planejada |
