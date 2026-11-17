# Documentação

| Pasta                   | Conteúdo                                                 |
| ----------------------- | -------------------------------------------------------- |
| [`adr/`](./adr)         | Architecture Decision Records: o **porquê** das decisões |
| [`specs/`](./specs)     | Especificações de funcionalidades: o **quê** construir   |
| [`process/`](./process) | Como trabalhamos com IA: ciclo, revisão e segurança      |
| [`testing/`](./testing) | Estratégia de testes e registro de falhas                |

## Índice de ADRs

| ADR                                                          | Título                                  | Status |
| ------------------------------------------------------------ | --------------------------------------- | ------ |
| [0001](./adr/0001-spec-driven-development-e-adrs.md)         | Spec Driven Development e ADRs          | Aceito |
| [0002](./adr/0002-nextjs-app-router-e-typescript-estrito.md) | Next.js App Router e TypeScript estrito | Aceito |
| [0003](./adr/0003-estrategia-de-testes.md)                   | Estratégia de testes                    | Aceito |
| [0004](./adr/0004-fluxo-de-branches-e-commits.md)            | Fluxo de branches e commits             | Aceito |

## Glossário do domínio

| Termo             | Definição                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------- |
| **Documento**     | Arquivo digital gerenciado pelo sistema (contrato, nota fiscal, relatório…), com metadados.       |
| **Tipo**          | Categoria do documento: `contrato`, `nota_fiscal`, `relatorio`, `politica`, `outro`.              |
| **Status**        | Ciclo de vida: `rascunho` → `em_revisao` → `aprovado` → `arquivado`. Pode ficar `vencido`.        |
| **Responsável**   | Pessoa fictícia encarregada do documento.                                                         |
| **Pendência**     | Ação necessária sobre um documento (ex.: "assinatura faltante"), com prazo e status.              |
| **Assistente**    | Recurso de IA que resume documentos, aponta pendências e propõe ações com confirmação humana.     |
| **Ação proposta** | Alteração sugerida pelo assistente que só é executada após confirmação e autorização no servidor. |
