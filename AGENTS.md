# AGENTS.md — Instruções para agentes de IA

> Fonte única de instruções para qualquer agente de IA (Claude Code, Copilot, Cursor etc.) que trabalhe neste repositório. Leia este arquivo inteiro antes de qualquer tarefa.

## 1. Contexto do projeto

Painel de gestão de documentos digitais com assistente de IA integrado.

- Funcionalidades: `docs/specs/`
- Decisões arquiteturais: `docs/adr/`
- Processo de desenvolvimento com IA: `docs/process/ai-development-cycle.md`

## 2. Stack

| Camada           | Tecnologia                                     |
| ---------------- | ---------------------------------------------- |
| Framework        | Next.js (App Router)                           |
| Linguagem        | TypeScript em modo `strict`                    |
| UI               | React (Server Components por padrão)           |
| Dados no cliente | TanStack Query                                 |
| API              | GraphQL tipado                                 |
| Validação        | Zod                                            |
| Testes           | Vitest + Testing Library, MSW, Playwright, axe |
| Gerenciador      | pnpm                                           |

## 3. Comandos

```bash
pnpm dev          # servidor de desenvolvimento
pnpm lint         # ESLint
pnpm typecheck    # next typegen + tsc --noEmit
pnpm test         # testes unitários e de componentes (Vitest)
pnpm test:e2e     # testes ponta a ponta (Playwright)
pnpm validate     # lint + typecheck + test (rodar antes de concluir qualquer tarefa)
```

## 4. Fluxo obrigatório de trabalho

Toda tarefa segue o ciclo descrito em `docs/process/ai-development-cycle.md`:

1. **Entender**: leia a SPEC indicada. Se não houver SPEC, **pare e peça uma**. Não implemente funcionalidade sem especificação.
2. **Planejar**: apresente um plano (arquivos, passos, testes, riscos) e **aguarde aprovação** antes de editar código.
3. **Implementar em ciclos ReAct**: raciocinar → agir (um passo pequeno) → observar (rodar typecheck/testes) → repetir.
4. **Revisar**: rode `pnpm validate`, faça a autorrevisão (seção 10) e entregue o relatório final.

## 5. Arquitetura de pastas

```text
src/
├── app/                      # rotas, layouts e route handlers (App Router)
├── features/<dominio>/       # módulos por domínio (documents, assistant…)
│   ├── domain/               # regras de domínio puras (transições, cálculos)
│   ├── components/
│   ├── hooks/
│   ├── api/                  # queries, mutations e chaves do TanStack Query
│   ├── schemas/              # schemas Zod (fonte dos tipos)
│   └── types.ts
├── components/ui/            # componentes genéricos e reutilizáveis, sem regra de negócio
├── lib/                      # utilitários puros
├── lib/server/               # código exclusivo do servidor (importa 'server-only')
├── mocks/                    # dados fictícios e handlers MSW
└── test/                     # setup e utilitários de teste
e2e/                          # testes Playwright
```

Regras:

- Regra de negócio fica em `features/`, nunca em `components/ui/`.
- Uma feature não importa arquivos internos de outra feature. Use o `index.ts` público dela.
- Código com segredos ou chamadas a LLM fica em `lib/server/` ou em route handlers e importa `server-only`.

## 6. Componentes React

- **Server Component por padrão.** Use `'use client'` só quando houver estado, efeitos, eventos ou APIs do navegador, e sempre na folha da árvore.
- Arquivos em `kebab-case.tsx`, componentes em `PascalCase`, apenas **named exports** (exceto onde o Next.js exige `default`).
- Props tipadas com `type` explícito, sem `React.FC`.
- Todo componente interativo precisa ser acessível: HTML semântico, rótulos, foco visível e navegação por teclado.
- Trate todos os estados de UI definidos na SPEC: carregando, vazio, erro, sucesso (e, quando houver, interrompido/confirmação).
- Teste colocalizado: `document-table.tsx` → `document-table.test.tsx`.

## 7. TypeScript

- Proibido `any`. Use `unknown` e faça o estreitamento.
- Proibido `@ts-ignore`. `@ts-expect-error` só com comentário justificando.
- Evite o non-null assertion (`!`). Prefira tratar o caso nulo.
- Dados externos (API, formulários, resposta de LLM) **sempre** passam por schema Zod. Os tipos derivam do schema com `z.infer`.
- Use union types discriminadas para estados (`{ status: 'error'; error: AppError }`).

## 8. Testes

- Cada critério de aceite (`CA-xx`) da SPEC precisa de ao menos um teste que o referencie no nome:
  `it('CA-03: exibe estado vazio quando não há documentos', …)`.
- Teste comportamento, não implementação. Use queries por papel (`getByRole`).
- **Nunca altere um teste existente só para fazê-lo passar.** Se o teste estiver errado, explique o motivo no relatório.
- Toda correção de bug inclui um teste de regressão e um registro em `docs/testing/failures/`.

## 9. Segurança e dependências

- Nunca leia, exiba ou faça commit de `.env*`, chaves ou tokens.
- Variáveis `NEXT_PUBLIC_*` são públicas: nunca coloque segredos nelas.
- Proibido `dangerouslySetInnerHTML` e renderização de HTML arbitrário vindo de API ou LLM.
- Use apenas dados fictícios. Nunca dados reais de pessoas ou empresas.
- **Não instale dependências sem aprovação.** Antes de propor uma, siga `docs/process/security-dependencies-review.md` e confirme que o pacote existe e é o correto (cuidado com nomes alucinados ou parecidos).

## 10. Autorrevisão antes de concluir

- [ ] Todos os critérios de aceite da SPEC estão cobertos por testes
- [ ] `pnpm validate` passou sem erros nem warnings novos
- [ ] Nenhum `any`, `console.log` esquecido, código comentado ou TODO sem issue
- [ ] Estados de carregando, vazio e erro tratados
- [ ] Acessibilidade verificada (teclado, rótulos, foco)
- [ ] Nenhum segredo exposto ao cliente
- [ ] Nenhuma dependência nova sem aprovação
- [ ] Alterações restritas ao escopo da SPEC

## 11. Quando parar e perguntar

- A SPEC está ambígua, contraditória ou ausente
- A solução exige uma dependência nova ou muda uma decisão registrada em ADR
- O mesmo erro persiste após **3 tentativas** de correção
- A tarefa exige alterar arquivos fora do escopo do plano aprovado

## 12. Git

- Nunca trabalhe direto em `main` ou `develop`.
- Siga as convenções de `CONTRIBUTING.md` (branches e Conventional Commits).
- Não faça push nem abra PR sem pedido explícito.

## 13. Formato do relatório final

```text
## Resumo
O que foi feito, em 2–3 linhas.

## SPEC e critérios atendidos
SPEC-xxx: CA-01 ✅, CA-02 ✅, CA-03 ⚠️ (motivo)

## Arquivos alterados
- caminho/arquivo.tsx: o que mudou

## Validação
Saída resumida de lint, typecheck e testes.

## Riscos, decisões e pendências
O que o revisor humano precisa olhar com atenção.
```
