# Revisão de dependências, comportamento e segurança

Checklist usado pelo revisor humano em todo código gerado ou alterado por IA.

## 1. Dependências

Antes de aprovar qualquer pacote novo:

- [ ] **Necessidade**: não dá para resolver com a plataforma (Web APIs, React, Next.js) ou com uma dependência já existente?
- [ ] **Existência e nome correto**: o pacote existe no npm e é o oficial? Agentes de IA podem sugerir pacotes inexistentes ou com nomes parecidos com os legítimos (_slopsquatting_ / _typosquatting_).
- [ ] **Manutenção**: última publicação recente, issues respondidas, mais de um mantenedor.
- [ ] **Popularidade**: downloads semanais e repositório público coerentes.
- [ ] **Licença** compatível (MIT, Apache-2.0, BSD).
- [ ] **Tamanho no bundle** aceitável (se for usada no cliente).
- [ ] **Tipos TypeScript** nativos ou `@types` mantidos.
- [ ] **Vulnerabilidades**: `pnpm audit` sem alertas altos ou críticos.
- [ ] **Scripts de instalação** (`postinstall`) conhecidos e justificados.
- [ ] `pnpm-lock.yaml` atualizado no mesmo commit.

```bash
pnpm view <pacote> name version license repository maintainers time.modified
pnpm audit --prod
```

## 2. Comportamento

- [ ] O código faz **exatamente** o que a SPEC pede, nem mais nem menos.
- [ ] Todos os estados de UI (carregando, vazio, erro, sucesso) existem e foram testados.
- [ ] Erros são tratados com mensagem útil ao usuário e sem vazar detalhes internos.
- [ ] Sem efeitos colaterais inesperados: requisições duplicadas, re-renderizações em loop, vazamento de listeners.
- [ ] Os testes verificam o comportamento correto, não apenas "passam".
- [ ] Nomes, estrutura e padrões seguem o `AGENTS.md`.

## 3. Segurança

| Área                 | Verificação                                                          |
| -------------------- | -------------------------------------------------------------------- |
| Segredos             | Nenhum segredo em código, logs, `NEXT_PUBLIC_*` ou Client Components |
| Fronteira servidor   | Código sensível importa `server-only`                                |
| XSS                  | Sem `dangerouslySetInnerHTML`, sem HTML arbitrário vindo de API/LLM  |
| Validação de entrada | Toda entrada externa passa por schema Zod no servidor                |
| Autorização          | Ações que alteram dados são validadas no servidor, não só na UI      |
| Dados                | Apenas dados fictícios, sem dados pessoais reais                     |
| IA (pilar 3)         | Conteúdo de documentos tratado como não confiável (prompt injection) |
| Logs                 | Sem dados sensíveis em `console` ou ferramentas de observabilidade   |

## 4. Resultado da revisão

```text
Revisão: SPEC-NNN / PR #xx
Dependências: ✅ / ⚠️ (detalhe)
Comportamento: ✅ / ⚠️ (detalhe)
Segurança: ✅ / ⚠️ (detalhe)
Decisão: Aprovado | Ajustes necessários
```
