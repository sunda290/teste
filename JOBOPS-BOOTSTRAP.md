# JobOps: Pipeline de Prospecção e Candidatura Assistida

**Bootstrap para Claude Code**
Versão 1.0 · AlphaOps · Hamilton, Ontario

---

## 0. Como usar este arquivo

Este documento é a instrução única de bootstrap. Abra o Claude Code na pasta vazia do projeto e execute:

```bash
claude
```

Depois, no prompt:

```
Leia JOBOPS-BOOTSTRAP.md por completo antes de escrever qualquer código.
Execute apenas a FASE 0. Pare no GATE 0 e aguarde minha aprovação explícita.
Não avance de fase sem aprovação.
```

O Claude Code não deve gerar código de fases futuras. Cada fase termina em um gate humano.

---

## 1. Objetivo

Reduzir o custo de atenção da busca por emprego em TI no Canadá. O sistema coleta vagas, pontua contra um perfil calibrado, e produz peças de candidatura sob medida. A decisão de aplicar e o envio permanecem humanos.

**Métrica de sucesso primária:** taxa de resposta por candidatura enviada.
**Métrica secundária:** tempo do operador por candidatura enviada.
**Anti-métrica:** volume de candidaturas. Volume alto sem resposta é falha, não progresso.

---

## 2. Restrições invioláveis

Estas restrições são de arquitetura, não de estilo. Qualquer implementação que as viole deve ser rejeitada no code review.

| ID | Restrição | Razão |
|----|-----------|-------|
| R1 | Nenhum cookie ou credencial de sessão do LinkedIn entra no sistema | Fluxos com cookie carregam risco de restrição de conta. O ativo é a conta, não o dado. |
| R2 | Zero auto-apply. O sistema nunca submete formulário | Candidatura automatizada é detectável, descartada por ATS e queima a empresa como lead |
| R3 | Toda peça gerada passa por revisão humana antes de sair | Responsabilidade é do candidato, não do modelo |
| R4 | Dados pessoais de recrutadores tratados sob PIPEDA. Nome e cargo apenas. Sem e-mail scraping, sem enriquecimento de contato | Conformidade e reputação |
| R5 | Sem Supabase. PostgreSQL local ou gerenciado, acessado por REST API própria | Padrão de stack AlphaOps |
| R6 | Nenhuma afirmação sobre elegibilidade migratória é gerada pelo sistema. Apenas sinalização de TEER para verificação humana | O sistema não dá parecer legal |

---

## 3. Stack

- **Runtime:** Node.js 22 LTS, TypeScript strict
- **API:** Express
- **Banco:** PostgreSQL 16
- **Coleta:** Apify API via `apify-client`
- **Coleta secundária:** conector Indeed
- **Inferência:** Anthropic API, Claude Sonnet para scoring em lote, Claude Opus para redação de peças
- **Memória de projeto:** vault Obsidian versionada
- **Orquestração:** Claude Code com subagents e slash commands
- **Testes:** Vitest
- **Lint:** ESLint + Prettier

Proibido: ORM pesado. Use `pg` com SQL explícito e migrations numeradas.

---

## 4. Arquitetura lógica

Cinco estágios, cada um idempotente e re-executável isoladamente.

```
RECON      coleta bruta        Apify + Indeed        ->  raw_jobs
NORMALIZE  dedupe e limpeza    regras determinísticas ->  jobs
SCORE      priorização         Claude Sonnet          ->  scores
DRAFT      peças               Claude Opus            ->  artifacts
TRACK      funil e métricas    SQL + vault            ->  reports
```

Regra de ouro: **nenhum LLM no estágio NORMALIZE**. Deduplicação e limpeza são determinísticas e testáveis. Modelo só entra onde há julgamento.

---

## 5. Estrutura do repositório

```
alphaops-jobops/
├── .claude/
│   ├── agents/
│   │   ├── recon.md
│   │   ├── analyst.md
│   │   ├── writer.md
│   │   ├── tracker.md
│   │   └── auditor.md
│   ├── commands/
│   │   ├── recon.md
│   │   ├── score.md
│   │   ├── draft.md
│   │   ├── report.md
│   │   └── gate.md
│   └── settings.json
├── CLAUDE.md
├── JOBOPS-BOOTSTRAP.md
├── docs/
│   ├── adr/
│   │   └── 0001-arquitetura-de-cinco-estagios.md
│   └── runbook.md
├── migrations/
│   ├── 001_init.sql
│   ├── 002_scores.sql
│   └── 003_applications.sql
├── src/
│   ├── config/
│   │   ├── env.ts
│   │   └── scoring-weights.ts
│   ├── collectors/
│   │   ├── apify-linkedin.ts
│   │   ├── indeed.ts
│   │   └── types.ts
│   ├── normalize/
│   │   ├── dedupe.ts
│   │   ├── ghost-detector.ts
│   │   └── teer-mapper.ts
│   ├── scoring/
│   │   ├── prompt.ts
│   │   ├── score-batch.ts
│   │   └── calibration.ts
│   ├── drafting/
│   │   ├── resume-bullets.ts
│   │   └── cover-letter.ts
│   ├── vault/
│   │   ├── writer.ts
│   │   └── templates/
│   ├── api/
│   │   ├── server.ts
│   │   └── routes/
│   └── db/
│       ├── client.ts
│       └── queries.ts
├── profile/
│   ├── master-resume.md
│   ├── skills-matrix.yaml
│   └── exclusions.yaml
├── tests/
├── .env.example
└── package.json
```

---

## 6. Vault Obsidian

Caminho definido em `OBSIDIAN_VAULT_PATH`. O vault é memória persistente do projeto, não backup. O Claude Code lê o vault antes de decisões e escreve depois.

```
JobOps/
├── 00-Comando/
│   ├── Estado Atual.md          # dashboard vivo, sobrescrito a cada /report
│   ├── Decisoes.md              # log append-only, uma linha por decisão
│   └── Gates.md                 # registro de aprovação de cada gate
├── 10-Vagas/
│   └── {YYYY-MM-DD}-{empresa}-{cargo}.md
├── 20-Empresas/
│   └── {empresa}.md             # histórico, contatos, candidaturas prévias
├── 30-Pecas/
│   └── {job_id}/
│       ├── cover-letter.md
│       └── resume-bullets.md
├── 40-Metricas/
│   └── {YYYY-Www}.md            # relatório semanal
└── 50-Perfil/
    ├── Curriculo Master.md
    ├── Matriz de Competencias.md
    └── Restricoes e Exclusoes.md
```

**Frontmatter obrigatório em 10-Vagas:**

```yaml
---
job_id:
source: linkedin | indeed
company:
title:
location:
posted_at:
scraped_at:
score:
teer:
status: novo | triado | descartado | aplicado | resposta | entrevista | oferta | recusa
decided_at:
decided_by: humano
---
```

Status é atualizado por você, não pelo sistema. O sistema só escreve `novo` e `triado`.

---

## 7. Orquestra de agents

Cinco subagents, responsabilidade única cada. Definidos em `.claude/agents/`.

### 7.1 `recon`
Coleta. Chama Apify e Indeed, grava em `raw_jobs`, nunca interpreta conteúdo.
- Ferramentas: Bash, Read, Write
- Proibido: chamar Anthropic API, editar arquivos fora de `src/collectors/`

### 7.2 `analyst`
Pontuação. Lê `jobs` + `profile/`, produz `scores` com justificativa.
- Ferramentas: Read, Write, Bash
- Saída obrigatória: JSON validado por schema Zod. Sem prosa.
- Proibido: alterar pesos de scoring por conta própria. Pesos vivem em `scoring-weights.ts` e só mudam por ADR.

### 7.3 `writer`
Redação de peças. Só roda sob `/draft <job_id>` explícito.
- Ferramentas: Read, Write
- Regra rígida: nunca usar travessão. Nunca inventar experiência ausente do `master-resume.md`. Se falta evidência para uma exigência da vaga, sinalizar a lacuna em vez de preencher.

### 7.4 `tracker`
Estado do funil e métricas. Lê banco, escreve vault.
- Ferramentas: Read, Write, Bash
- Proibido: escrever em `src/`

### 7.5 `auditor`
Roda antes de cada gate. Verifica R1 a R6, cobertura de testes, migrations aplicadas, ausência de segredo em commit.
- Ferramentas: Read, Bash, Grep
- Saída: relatório PASS/FAIL por restrição. Um único FAIL bloqueia o gate.

---

## 8. Slash commands

| Comando | Ação |
|---------|------|
| `/recon` | Roda coleta nas queries configuradas, reporta contagem bruta e após dedupe |
| `/score` | Pontua vagas não pontuadas, imprime top 10 com justificativa em uma linha |
| `/draft <job_id>` | Gera cover letter e bullets, grava em `30-Pecas/`, exibe diff |
| `/report` | Atualiza `Estado Atual.md` e o relatório semanal |
| `/gate <n>` | Invoca `auditor`, imprime PASS/FAIL, registra em `Gates.md` |

---

## 9. Modelo de dados

```sql
-- 001_init.sql
CREATE TABLE companies (
  id            BIGSERIAL PRIMARY KEY,
  name          TEXT NOT NULL,
  normalized    TEXT NOT NULL UNIQUE,
  linkedin_url  TEXT,
  headcount     INTEGER,
  industry      TEXT,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE raw_jobs (
  id            BIGSERIAL PRIMARY KEY,
  source        TEXT NOT NULL,
  source_id     TEXT NOT NULL,
  payload       JSONB NOT NULL,
  scraped_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (source, source_id)
);

CREATE TABLE jobs (
  id            BIGSERIAL PRIMARY KEY,
  company_id    BIGINT REFERENCES companies(id),
  fingerprint   TEXT NOT NULL UNIQUE,
  title         TEXT NOT NULL,
  location      TEXT,
  remote        BOOLEAN,
  description   TEXT NOT NULL,
  apply_url     TEXT,
  posted_at     TIMESTAMPTZ,
  first_seen_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_seen_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  repost_count  INTEGER NOT NULL DEFAULT 0,
  teer          SMALLINT,
  noc_code      TEXT
);

CREATE INDEX idx_jobs_posted ON jobs (posted_at DESC);
```

`fingerprint` = SHA256 de `normalized_company + normalized_title + normalized_location`. É a chave de dedupe entre fontes e a base do `ghost-detector`: se `repost_count >= 3` em 90 dias, a vaga é sinalizada como provável vaga fantasma.

```sql
-- 002_scores.sql
CREATE TABLE scores (
  id             BIGSERIAL PRIMARY KEY,
  job_id         BIGINT NOT NULL REFERENCES jobs(id) ON DELETE CASCADE,
  total          SMALLINT NOT NULL CHECK (total BETWEEN 0 AND 100),
  breakdown      JSONB NOT NULL,
  rationale      TEXT NOT NULL,
  model          TEXT NOT NULL,
  weights_hash   TEXT NOT NULL,
  scored_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

`weights_hash` permite re-pontuar tudo quando os pesos mudarem, sem perder o histórico.

```sql
-- 003_applications.sql
CREATE TABLE applications (
  id           BIGSERIAL PRIMARY KEY,
  job_id       BIGINT NOT NULL REFERENCES jobs(id),
  applied_at   TIMESTAMPTZ NOT NULL,
  channel      TEXT NOT NULL,
  artifacts    JSONB NOT NULL,
  status       TEXT NOT NULL DEFAULT 'enviada',
  responded_at TIMESTAMPTZ,
  outcome      TEXT,
  notes        TEXT
);
```

---

## 10. Scoring

Pesos iniciais em `src/config/scoring-weights.ts`. São hipóteses, não verdades. Recalibrar no GATE 3 com dados reais.

```typescript
export const WEIGHTS = {
  technicalMatch:      30,  // competência real, não keyword
  recency:             20,  // <24h = full, decai linear até 0 em 14 dias
  seniorityFit:        15,  // penaliza over e under qualification
  employerViability:   15,  // tamanho, setor, histórico de contratação
  teerAlignment:       10,  // sinalização, não decisão
  ghostPenalty:       -20,  // repost_count e linguagem genérica
  applicantVolume:    -10,  // número de candidatos quando disponível
} as const;
```

O `analyst` retorna breakdown por dimensão. Se o total não for reprodutível a partir do breakdown, a resposta é rejeitada e re-solicitada.

---

## 11. Fases e gates

Nenhuma fase começa sem o gate anterior aprovado. Cada gate exige `auditor` em PASS e sua aprovação escrita em `00-Comando/Gates.md`.

### FASE 0: Validação sem código
**Duração:** 3 dias. **Nenhuma linha de código de produção.**

1. Rodar o actor Apify manualmente pelo console, 3 execuções, queries distintas
2. Rodar o conector Indeed manualmente, mesmas queries
3. Exportar CSV, colar em planilha, deduplicar à mão
4. Pontuar 30 vagas manualmente com os pesos da seção 10
5. Escrever `profile/master-resume.md` e `profile/skills-matrix.yaml`

**GATE 0.** Aprovar se, e somente se:
- O volume diário de vagas relevantes justifica automação. Menos de 10 por dia, o projeto não se paga
- A pontuação manual produziu ranking que você reconhece como correto
- Custo estimado Apify por mês está dentro do orçamento definido

Se GATE 0 falhar, o projeto encerra aqui. Isso é um resultado válido.

### FASE 1: RECON + NORMALIZE
Coletores, schema, dedupe, ghost-detector, testes. Sem LLM.

**GATE 1:**
- Dedupe com precisão verificada em amostra de 50 pares
- Duas execuções consecutivas produzem zero duplicata nova
- Cobertura de testes acima de 80% em `src/normalize/`
- `auditor` PASS em R1, R4, R5

### FASE 2: SCORE
Prompt, batch, validação de schema, persistência.

**GATE 2:**
- Concordância entre pontuação do modelo e sua pontuação manual da FASE 0 acima de 70% no top 10
- Custo por 100 vagas pontuadas documentado
- Toda pontuação tem breakdown reprodutível

### FASE 3: DRAFT
Geração de peças. Aqui o sistema começa a produzir valor externo.

**GATE 3:**
- 5 peças geradas e revisadas por você linha a linha
- Zero afirmação não sustentada pelo `master-resume.md`
- Zero travessão na saída
- Pesos de scoring recalibrados com o que você aprendeu

### FASE 4: TRACK e operação
Relatórios, funil, agendamento.

**GATE 4:**
- 20 candidaturas enviadas com dados no banco
- Taxa de resposta calculada e registrada
- Decisão documentada: escalar, ajustar o perfil de busca, ou mudar de canal

Só agendar cron depois do GATE 4. Automatizar um funil que não converte multiplica o desperdício.

---

## 12. Ambiente

```bash
# .env.example
APIFY_TOKEN=
APIFY_ACTOR_LINKEDIN=apimaestro/linkedin-jobs-scraper-api
ANTHROPIC_API_KEY=
DATABASE_URL=postgresql://jobops:senha@localhost:5432/jobops
OBSIDIAN_VAULT_PATH=/Users/andre/Obsidian/JobOps
APIFY_MONTHLY_BUDGET_USD=15
LOG_LEVEL=info
```

`src/config/env.ts` valida tudo com Zod na inicialização. Falta de variável derruba o processo no boot, não em runtime.

---

## 13. Repositório GitHub

```bash
gh repo create alphaops-jobops --private --description "Pipeline de prospeccao e candidatura assistida"
git init
git branch -M main
```

`.gitignore` obrigatório: `.env`, `profile/master-resume.md`, `data/`, `*.csv`.

O currículo master e o vault contêm dados pessoais e não vão para o repositório, mesmo privado. O vault tem versionamento próprio, em repositório separado ou sync do Obsidian.

**Branch por fase:** `fase/1-recon`, `fase/2-score`. Merge em `main` somente após gate aprovado. A tag do gate marca o commit: `git tag gate-1`.

**Commits:** Conventional Commits. `feat(recon):`, `fix(normalize):`, `docs(adr):`.

---

## 14. CLAUDE.md do projeto

O Claude Code deve gerar `CLAUDE.md` na raiz contendo, no máximo, 100 linhas:

1. As seis restrições da seção 2, íntegras
2. Stack e comandos de build, teste e lint
3. A regra dos gates: nunca avançar de fase sem aprovação escrita
4. Caminho do vault e obrigação de ler `00-Comando/Estado Atual.md` no início de cada sessão
5. Padrão de commit e branch
6. Proibição de travessão em qualquer saída destinada a terceiros

`CLAUDE.md` é contrato operacional, não documentação. Se ficar longo, ninguém lê, inclusive o modelo.

---

## 15. Primeira ação do Claude Code

Nesta ordem, e apenas isto:

1. Ler este arquivo por completo
2. Criar `CLAUDE.md` conforme a seção 14
3. Criar `docs/adr/0001-arquitetura-de-cinco-estagios.md` registrando a decisão e as alternativas descartadas
4. Criar a estrutura de pastas do vault e `00-Comando/Estado Atual.md` com status `FASE 0 em andamento`
5. Criar `profile/master-resume.md` e `profile/skills-matrix.yaml` como esqueletos para preenchimento humano
6. Imprimir o checklist da FASE 0 e parar

**Não escrever código de aplicação. Não instalar dependências. Parar no GATE 0.**
