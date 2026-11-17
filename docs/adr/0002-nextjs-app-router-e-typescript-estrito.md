# ADR-0002: Next.js App Router e TypeScript estrito

- **Status:** Aceito
- **Data:** 2026-11-17
- **Decisores:** Andre Menezes

## Contexto

O painel precisa de: renderização no servidor para dados iniciais, um local seguro para segredos e chamadas a LLM, streaming de respostas do assistente e uma base tipada que reduza erros de código gerado por IA.

## Opções consideradas

1. **React + Vite (SPA)** com backend separado.
2. **Next.js com Pages Router.**
3. **Next.js com App Router**, Server Components e Route Handlers.

## Decisão

Adotamos a **opção 3**, com TypeScript em `strict: true` e `noUncheckedIndexedAccess: true`.

- **Server Components por padrão**. Client Components apenas quando necessário, empurrados para as folhas da árvore.
- Segredos e chamadas ao LLM ficam exclusivamente no servidor (Route Handlers ou Server Actions), com `import 'server-only'`.
- Versão do Next.js: a estável vigente no início do projeto, fixada no `package.json`. Atualizações de versão major exigem um novo ADR.

## Consequências

### Positivas

- Frontend e backend leve (BFF) no mesmo repositório.
- Suporte nativo a streaming e Suspense, essencial para o assistente.
- Menos JavaScript enviado ao cliente.
- Tipagem estrita pega erros comuns de código gerado por IA.

### Negativas e trade-offs

- Modelo mental mais complexo (fronteira servidor/cliente, cache).
- Agentes de IA às vezes geram padrões antigos (Pages Router, `getServerSideProps`). Isso é mitigado pelo `AGENTS.md`.

### Riscos e mitigação

- **Vazamento de segredo para o cliente**: `server-only`, revisão de `NEXT_PUBLIC_*` e checklist de segurança.
- **Uso excessivo de `'use client'`**: item de revisão obrigatório. A decisão de renderização de cada tela é documentada na SPEC.

## Como validar

- Nenhum segredo no bundle do cliente (verificação no build).
- O build falha com erros de tipo (`pnpm typecheck` no CI).
