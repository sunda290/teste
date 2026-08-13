# ADR 0001: Arquitetura de cinco estagios

- Status: Aceito
- Data: 2026-08-13
- Fase: FASE 0 (validacao sem codigo)

## Contexto

JobOps reduz o custo de atencao da busca por emprego em TI no Canada. O sistema
coleta vagas, pontua contra um perfil calibrado e produz pecas de candidatura sob
medida. A decisao de aplicar e o envio permanecem humanos.

A metrica de sucesso primaria e a taxa de resposta por candidatura enviada. A
anti-metrica e o volume de candidaturas: volume alto sem resposta e falha, nao
progresso. A arquitetura precisa manter o julgamento humano no centro e isolar o
uso de LLM apenas onde ha julgamento real.

## Decisao

Adotar um pipeline de cinco estagios, cada um idempotente e re-executavel
isoladamente:

```
RECON      coleta bruta        Apify + Indeed          ->  raw_jobs
NORMALIZE  dedupe e limpeza     regras deterministicas  ->  jobs
SCORE      priorizacao          Claude Sonnet           ->  scores
DRAFT      pecas                Claude Opus             ->  artifacts
TRACK      funil e metricas     SQL + vault             ->  reports
```

Regra de ouro: **nenhum LLM no estagio NORMALIZE**. Deduplicacao e limpeza sao
deterministicas e testaveis. Modelo so entra onde ha julgamento (SCORE, DRAFT).

Consequencias diretas:

- `fingerprint` = SHA256 de `normalized_company + normalized_title + normalized_location`
  e a chave de dedupe entre fontes e a base do ghost-detector.
- Pesos de scoring vivem em `src/config/scoring-weights.ts` e so mudam por ADR.
- Cada estagio tem sua propria tabela de saida, permitindo re-execucao isolada.
- Cada fase de implementacao termina em um gate humano; nada avanca sem aprovacao.

## Alternativas descartadas

1. **Pipeline monolitico com LLM ponta a ponta.** Um unico prompt recebendo vagas
   brutas e devolvendo pecas prontas. Descartado: nao e testavel, mistura
   deduplicacao (deterministica) com julgamento (probabilistico), e torna o custo
   e a reproducao impossiveis de auditar. Viola a separacao necessaria para os gates.

2. **LLM na deduplicacao (NORMALIZE).** Usar embeddings ou um modelo para decidir
   se duas vagas sao a mesma. Descartado: dedupe precisa ser deterministica,
   reproduzivel e coberta por testes acima de 80 por cento. Um modelo introduz
   nao-determinismo onde a regra de fingerprint resolve com custo zero.

3. **Supabase como backend.** Descartado por R5: padrao de stack AlphaOps exige
   PostgreSQL local ou gerenciado acessado por REST API propria.

4. **Auto-apply automatizado.** Submeter formularios de candidatura pelo sistema.
   Descartado por R2: e detectavel, descartado por ATS e queima a empresa como lead.
   O envio permanece humano.

5. **ORM pesado (Prisma, TypeORM).** Descartado: obscurece o SQL, dificulta a
   auditoria das migrations e das queries. Usamos `pg` com SQL explicito e
   migrations numeradas.

## Notas

Os pesos de scoring da secao 10 do bootstrap sao hipoteses iniciais, nao verdades.
Recalibrar no GATE 3 com dados reais. `weights_hash` na tabela `scores` permite
re-pontuar tudo quando os pesos mudarem, sem perder historico.
