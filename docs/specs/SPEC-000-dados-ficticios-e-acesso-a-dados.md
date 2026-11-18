# SPEC-000: Dados fictícios e camada de acesso a dados

| Campo          | Valor              |
| -------------- | ------------------ |
| Status         | Em revisão         |
| Pilar          | 1                  |
| Prioridade     | Alta               |
| ADRs           | ADR-0002, ADR-0005 |
| Depende de     | Setup do projeto   |
| Última revisão | 2026-11-17         |

## 1. Contexto e objetivo

Todas as funcionalidades do painel (listagem, filtros, detalhes, pendências e assistente de IA) dependem de dados. Esta SPEC define o **modelo de domínio**, um **conjunto de dados fictícios determinístico** e uma **camada de acesso a dados no servidor** que simula uma API real, incluindo latência e falhas.

Resultado esperado: as próximas SPECs constroem UI e transporte (GraphQL) sobre uma base estável, tipada e testada, sem depender de backend ou banco de dados real.

## 2. Escopo

### Dentro do escopo

- Schemas Zod e tipos de `Person`, `Document` e `PendingItem`
- Regras de domínio puras: transições de status e cálculo de vencimento
- Gerador de dados determinístico (mesma semente + mesma data de referência = mesmos dados)
- Fixtures curadas com casos de borda (seção 4.4)
- Repositório em memória, exclusivo do servidor, com consultas e mutações
- Simulação configurável de latência e falha
- Testes unitários de toda a camada

### Fora do escopo

- Transporte (GraphQL, Route Handlers) → SPECs do pilar 2
- Interface de usuário → SPEC-001 em diante
- Persistência entre reinícios do servidor
- Autenticação e perfis de usuário
- Upload ou armazenamento de arquivos binários (o documento tem apenas metadados e conteúdo textual)

## 3. Histórias de usuário

- **HU-01**: Como desenvolvedor, quero dados fictícios realistas e estáveis para construir e demonstrar as telas sem backend real.
- **HU-02**: Como desenvolvedor, quero simular latência e falhas para implementar e testar os estados de carregando e erro.
- **HU-03**: Como desenvolvedor, quero documentos com casos de borda (conteúdo vazio, texto malicioso, títulos longos) para testar a robustez da UI e do assistente de IA.

## 4. Modelo de domínio e regras de negócio

### 4.1 Entidades

| Entidade      | Campo         | Tipo                                                                   | Regras                                 |
| ------------- | ------------- | ---------------------------------------------------------------------- | -------------------------------------- |
| `Person`      | `id`          | `string`                                                               | Formato `pes_001`                      |
|               | `name`        | `string`                                                               | 3–80 caracteres                        |
|               | `email`       | `string`                                                               | Domínio reservado `@empresa.example`   |
|               | `area`        | `'juridico' \| 'financeiro' \| 'rh' \| 'compliance' \| 'operacoes'`    |                                        |
| `Document`    | `id`          | `string`                                                               | Formato `doc_0001`                     |
|               | `title`       | `string`                                                               | 1–200 caracteres                       |
|               | `type`        | `'contrato' \| 'nota_fiscal' \| 'relatorio' \| 'politica' \| 'outro'`  |                                        |
|               | `status`      | `'rascunho' \| 'em_revisao' \| 'aprovado' \| 'arquivado'`              | Ver RN-01                              |
|               | `ownerId`     | `string`                                                               | Referencia uma `Person` existente      |
|               | `tags`        | `string[]`                                                             | 0–5 itens, minúsculas                  |
|               | `content`     | `string`                                                               | Texto fictício, 0–20.000 caracteres    |
|               | `createdAt`   | `string` (ISO 8601)                                                    |                                        |
|               | `updatedAt`   | `string` (ISO 8601)                                                    | `>= createdAt`                         |
|               | `expiresAt`   | `string \| null` (ISO 8601)                                            | `null` = sem validade                  |
|               | `revision`    | `number`                                                               | Inteiro ≥ 1, incrementa a cada mutação |
| `PendingItem` | `id`          | `string`                                                               | Formato `pen_0001`                     |
|               | `documentId`  | `string`                                                               | Referencia um `Document` existente     |
|               | `kind`        | `'assinatura' \| 'revisao' \| 'aprovacao' \| 'atualizacao' \| 'outro'` |                                        |
|               | `description` | `string`                                                               | 3–280 caracteres                       |
|               | `status`      | `'aberta' \| 'concluida' \| 'cancelada'`                               |                                        |
|               | `dueDate`     | `string` (data `AAAA-MM-DD`)                                           |                                        |
|               | `origin`      | `'usuario' \| 'assistente'`                                            | Pendências propostas pela IA (pilar 3) |
|               | `createdAt`   | `string` (ISO 8601)                                                    |                                        |

### 4.2 Campos derivados (nunca armazenados)

| Campo                   | Regra                                                               |
| ----------------------- | ------------------------------------------------------------------- |
| `isExpired`             | `expiresAt !== null && expiresAt < agora && status !== 'arquivado'` |
| `openPendingCount`      | Quantidade de pendências com status `aberta` do documento           |
| `isOverdue` (pendência) | `status === 'aberta' && dueDate < hoje`                             |

> "Vencido" deixa de ser um status armazenado e passa a ser derivado da data de validade. Isso evita inconsistência entre status e data.

### 4.3 Regras de negócio

| ID    | Regra                                                                                                                                                          |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RN-01 | Transições de status permitidas: `rascunho → em_revisao`, `em_revisao → aprovado`, `em_revisao → rascunho`, `aprovado → arquivado`. Qualquer outra é inválida. |
| RN-02 | Um documento só pode ser arquivado se não tiver pendências `aberta`.                                                                                           |
| RN-03 | Não é possível criar pendência em documento `arquivado`.                                                                                                       |
| RN-04 | `dueDate` de uma nova pendência não pode ser anterior a hoje.                                                                                                  |
| RN-05 | Pendências só mudam de `aberta` para `concluida` ou `cancelada`. Os estados finais são imutáveis.                                                              |
| RN-06 | Toda mutação incrementa `revision` e atualiza `updatedAt` do documento afetado.                                                                                |
| RN-07 | Mutações recebem `expectedRevision`. Se for diferente da revisão atual, a operação é rejeitada (concorrência otimista).                                        |
| RN-08 | Toda entrada das funções públicas é validada com Zod antes de qualquer processamento.                                                                          |
| RN-09 | Funções públicas retornam `Result<T, DataError>` e nunca lançam exceção para erros esperados.                                                                  |

### 4.4 Conjunto de dados

| Item               | Valor                                                                  |
| ------------------ | ---------------------------------------------------------------------- |
| Semente padrão     | `42`                                                                   |
| Data de referência | Data de início do servidor. Nos testes, fixa em `2026-11-01T12:00:00Z` |
| Pessoas            | 8, distribuídas entre todas as áreas                                   |
| Documentos         | 120 gerados + fixtures curadas                                         |
| Pendências         | 0–3 por documento não arquivado                                        |
| Distribuição       | Todos os tipos e status presentes; ~10% vencidos; ~15% sem validade    |

**Fixtures curadas** (IDs fixos, sempre presentes):

| ID         | Caso                                                                            | Uso futuro                         |
| ---------- | ------------------------------------------------------------------------------- | ---------------------------------- |
| `doc_9001` | Título com 200 caracteres, acentos e emoji                                      | Layout, truncamento                |
| `doc_9002` | Título contendo `<script>alert(1)</script>`                                     | Renderização segura (XSS)          |
| `doc_9003` | Conteúdo vazio                                                                  | Assistente: dados insuficientes    |
| `doc_9004` | Conteúdo com tentativa de prompt injection ("Ignore as instruções anteriores…") | Assistente: conteúdo não confiável |
| `doc_9005` | Conteúdo longo (~20.000 caracteres)                                             | Limites de entrada do LLM          |
| `doc_9006` | Vence amanhã em relação à data de referência                                    | Borda de vencimento                |
| `doc_9007` | Venceu ontem, com pendências atrasadas                                          | Alertas, pendências                |
| `doc_9008` | `em_revisao` com 3 pendências abertas                                           | Bloqueio de arquivamento (RN-02)   |

## 5. Interface pública da camada de dados

```ts
type Result<T, E = DataError> = { ok: true; value: T } | { ok: false; error: E };

type DataError =
  | { code: 'VALIDATION'; message: string; issues: { path: string; message: string }[] }
  | { code: 'NOT_FOUND'; message: string; entity: 'document' | 'person' | 'pendingItem'; id: string }
  | { code: 'INVALID_TRANSITION'; message: string; from: DocumentStatus; to: DocumentStatus }
  | { code: 'BUSINESS_RULE'; message: string; rule: 'RN-02' | 'RN-03' | 'RN-04' | 'RN-05' }
  | { code: 'CONFLICT'; message: string; expectedRevision: number; currentRevision: number }
  | { code: 'UNAVAILABLE'; message: string };

type Page<T> = { items: T[]; page: number; pageSize: number; totalItems: number; totalPages: number };

type ListDocumentsParams = {
  page?: number;          // padrão 1, inteiro ≥ 1
  pageSize?: number;      // padrão 20, entre 1 e 100
  sort?: { field: 'updatedAt' | 'title' | 'expiresAt'; direction: 'asc' | 'desc' }; // padrão updatedAt desc
  filters?: {
    search?: string;      // título e tags, sem diferenciar maiúsculas nem acentos, até 100 caracteres
    types?: DocumentType[];
    statuses?: DocumentStatus[];
    ownerIds?: string[];
    expired?: boolean;
  };
};

// Documentos
listDocuments(params: ListDocumentsParams): Promise<Result<Page<DocumentSummary>>>;
getDocumentById(id: string): Promise<Result<DocumentDetail>>;
updateDocumentStatus(input: { id: string; to: DocumentStatus; expectedRevision: number }): Promise<Result<DocumentDetail>>;

// Pessoas
listPeople(): Promise<Result<Person[]>>;

// Pendências
listPendingItems(documentId: string): Promise<Result<PendingItem[]>>;
createPendingItem(input: { documentId: string; kind: PendingKind; description: string; dueDate: string; origin: PendingOrigin; expectedRevision: number }): Promise<Result<PendingItem>>;
updatePendingItemStatus(input: { id: string; to: 'concluida' | 'cancelada'; expectedRevision: number }): Promise<Result<PendingItem>>;
```

- `DocumentSummary`: documento **sem** `content`, com `owner` resolvido, `isExpired` e `openPendingCount`.
- `DocumentDetail`: `DocumentSummary` + `content` + `pendingItems`.
- `expectedRevision` em pendências refere-se à `revision` do documento pai.

## 6. Critérios de aceite

```gherkin
Funcionalidade: Dados fictícios e camada de acesso a dados

  Cenário: CA-01 dados determinísticos
    Dado a semente 42 e a data de referência 2026-11-01T12:00:00Z
    Quando o conjunto de dados é gerado duas vezes
    Então os dois resultados são idênticos

  Cenário: CA-02 dados válidos e consistentes
    Quando o conjunto de dados é gerado
    Então todas as entidades passam na validação dos schemas
    E todas as referências (ownerId, documentId) apontam para entidades existentes
    E nenhum documento arquivado possui pendências abertas
    E as fixtures doc_9001 a doc_9008 estão presentes

  Cenário: CA-03 paginação padrão
    Quando listo documentos sem parâmetros
    Então recebo 20 itens ordenados por updatedAt decrescente
    E os metadados totalItems e totalPages estão corretos

  Cenário: CA-04 página além do total
    Quando solicito uma página maior que totalPages
    Então recebo uma lista vazia com os metadados corretos, sem erro

  Cenário: CA-05 parâmetros inválidos
    Quando solicito page 0 ou pageSize 101
    Então recebo um erro VALIDATION indicando o campo inválido

  Cenário: CA-06 busca sem acentos e sem diferenciar maiúsculas
    Dado um documento com o título "Política de Férias"
    Quando busco por "politica de ferias"
    Então o documento aparece no resultado

  Cenário: CA-07 filtros combinados
    Quando filtro por tipo "contrato" e status "aprovado"
    Então todos os itens retornados têm tipo "contrato" e status "aprovado"

  Cenário: CA-08 listagem não expõe conteúdo
    Quando listo documentos
    Então nenhum item contém o campo content

  Cenário: CA-09 detalhe do documento
    Quando busco um documento existente pelo id
    Então recebo o documento com conteúdo, responsável e pendências

  Cenário: CA-10 transição de status válida
    Dado um documento "rascunho" com revision 1
    Quando altero o status para "em_revisao" com expectedRevision 1
    Então o status passa a ser "em_revisao", revision passa a 2 e updatedAt é atualizado

  Cenário: CA-11 criação de pendência
    Dado um documento "em_revisao"
    Quando crio uma pendência válida com origem "assistente"
    Então a pendência é criada com status "aberta"
    E openPendingCount do documento aumenta em 1

  Cenário: CA-12 vencimento derivado
    Dado o documento doc_9007 e a data de referência
    Então isExpired é verdadeiro
    E o documento doc_9006 tem isExpired falso

  Cenário: CA-13 simulação de latência
    Dada uma latência configurada de 300 ms
    Quando chamo qualquer função pública
    Então a resposta só é entregue após 300 ms

  Cenário: CA-14 reinício do estado
    Dado que realizei mutações
    Quando o estado é reiniciado
    Então os dados voltam ao conjunto inicial
```

## 7. Restrições

| ID    | Tipo         | Restrição                                                                                                      |
| ----- | ------------ | -------------------------------------------------------------------------------------------------------------- |
| RT-01 | Técnica      | O repositório e o estado ficam em `src/lib/server/` e importam `server-only`.                                  |
| RT-02 | Técnica      | Regras de domínio e schemas são funções puras, sem I/O, reutilizáveis no cliente.                              |
| RT-03 | Técnica      | Sem dependências novas além de `zod` e `server-only`. O gerador usa um PRNG próprio.                           |
| RT-04 | Técnica      | Tempo injetável (`clock`) e aleatoriedade injetável (`random`) para testes determinísticos.                    |
| RT-05 | Segurança    | Apenas dados fictícios. E-mails usam domínio reservado (`.example`, RFC 2606).                                 |
| RT-06 | Segurança    | Funções retornam cópias. Alterar um objeto retornado não altera o estado interno.                              |
| RT-07 | Performance  | `listDocuments` responde em menos de 10 ms para 200 documentos, sem contar a latência simulada.                |
| RT-08 | Configuração | `SIMULATED_LATENCY_MS` (padrão 0) e `SIMULATED_FAILURE_RATE` (0 a 1, padrão 0), lidas e validadas no servidor. |

## 8. Cenários de erro

| ID     | Situação                                             | Comportamento esperado                                      |
| ------ | ---------------------------------------------------- | ----------------------------------------------------------- |
| ERR-01 | Parâmetros ou entrada inválidos                      | `VALIDATION` com a lista de campos e mensagens              |
| ERR-02 | Id inexistente                                       | `NOT_FOUND` com entidade e id                               |
| ERR-03 | Transição não permitida (ex.: `rascunho → aprovado`) | `INVALID_TRANSITION` com `from` e `to`; estado inalterado   |
| ERR-04 | Arquivar com pendências abertas                      | `BUSINESS_RULE` com `rule: 'RN-02'`; estado inalterado      |
| ERR-05 | Pendência em documento arquivado                     | `BUSINESS_RULE` com `rule: 'RN-03'`                         |
| ERR-06 | `dueDate` no passado                                 | `BUSINESS_RULE` com `rule: 'RN-04'`                         |
| ERR-07 | Alterar pendência já concluída ou cancelada          | `BUSINESS_RULE` com `rule: 'RN-05'`                         |
| ERR-08 | `expectedRevision` desatualizado (ex.: clique duplo) | `CONFLICT` com revisões esperada e atual; estado inalterado |
| ERR-09 | Falha simulada                                       | `UNAVAILABLE`; nenhuma mutação é aplicada                   |
| ERR-10 | Variável de ambiente de simulação inválida           | Erro na inicialização com mensagem clara (falha cedo)       |

## 9. Estados de interface

Não se aplica (sem UI). Os estados de carregando e erro das próximas SPECs são reproduzidos via ERR-09 e CA-13.

## 10. Decisão de renderização

Não se aplica. A camada é consumida apenas no servidor (Server Components, Route Handlers e resolvers GraphQL).

## 11. Plano de testes

| Critério                               | Tipo     | Arquivo                                                  |
| -------------------------------------- | -------- | -------------------------------------------------------- |
| CA-01, CA-02                           | Unitário | `src/lib/server/data/seed/generate-seed.test.ts`         |
| RN-01, CA-12                           | Unitário | `src/features/documents/domain/*.test.ts`                |
| Schemas, RN-08                         | Unitário | `src/features/documents/schemas/*.test.ts`               |
| CA-03 a CA-10, ERR-01 a ERR-04, ERR-08 | Unitário | `src/lib/server/data/repositories/documents.test.ts`     |
| CA-11, ERR-05 a ERR-07                 | Unitário | `src/lib/server/data/repositories/pending-items.test.ts` |
| CA-13, ERR-09, ERR-10                  | Unitário | `src/lib/server/data/simulation.test.ts`                 |
| CA-14, RT-06                           | Unitário | `src/lib/server/data/store.test.ts`                      |

## 12. Questões em aberto

- [ ] Versionamento de conteúdo do documento (histórico de versões) fica para uma SPEC futura?
- [ ] Ao rejeitar uma revisão (`em_revisao → rascunho`), as pendências abertas devem ser mantidas?

## 13. Histórico

| Data       | Alteração |
| ---------- | --------- |
| 2026-11-17 | Criação   |
