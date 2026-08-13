# FASE 0 — Coleta diaria Alberta (acumulador)

> Registro duravel (versionado) da validacao de volume do GATE 0. O vault e efemero
> neste ambiente; este arquivo e a fonte de verdade que sobrevive entre execucoes.
>
> Alvo: Backend / full-stack, junior, Calgary + Edmonton (relocacao aceita).
> Queries diarias: "junior backend developer" (Calgary), "node.js developer" (Calgary),
> "junior software developer" (Edmonton). Fonte: Indeed. country=CA.
> "Relevante" = score >= 50 (pesos da secao 10, calibrados Backend/full-stack junior).

## Tally diario

| Data | Bruto | Unicas | Relevantes (>=50) | Observacao |
|------|-------|--------|-------------------|------------|
| 2026-08-13 | 19 | 17 | 5 | Dia 1 (seed manual) |

Media de relevantes/dia: (a calcular a partir de 3+ dias)
Criterio GATE 0: >= 10 vagas relevantes/dia justifica automacao.

## Nota metodologica (2026-08-13)

- Re-rodar a coleta no MESMO dia-de-sandbox devolve o mesmo instantaneo do Indeed
  (mesmas datas de publicacao). Nao gera "dia novo". Regra: uma coleta por dia REAL,
  em sessao nova (o container tende a subir com a data avancada).
- Nao carimbar linha de tally sem dado novo. Contar as mesmas vagas duas vezes
  inventaria volume que nao existe.
- Cross-check independente de re-run: estimar fluxo pela DATA DE PUBLICACAO numa
  unica coleta. Em 2026-08-13, vagas de dev na amostra Alberta: ~4 nos ultimos 14
  dias, ~7 nos ultimos 30 -> menos de 1 relevante/dia. PISO, nao real: o conector
  Indeed limita ~10 por query. Volume verdadeiro deve ser lido no proprio Indeed
  ("X jobs found") ou ampliando as queries.
- Churn observado numa 2a chamada no mesmo dia: entrou Alberta Motor Association
  (Full Stack Web Dev, Edmonton, 03-jun, score ~44, abaixo do corte). Sairam CPKC e
  Fabled. Conjunto de relevantes (>=50) inalterado.

---

## 2026-08-13 (dia 1)

Relevantes (score >= 50):

1. 62 — Junior / Intermediate Software Developer — Express Employment — Alberta — 08-12
2. 62 — Full-Stack Developer, Business Systems — AFD Petroleum — Edmonton — 08-12
3. 60 — Full Stack Developer (AI Full-Stack) — THE MAMMOTH INC — Calgary — 08-11
4. 57 — Web Developer — CTOMS — Edmonton — 08-05
5. 52 — Junior Software Developer (New Graduate) — CardGio — Calgary — 07-15

Detalhe completo das 17 unicas em `vault/JobOps/40-Metricas/FASE0-resultado-alberta.md`
(quando rodado interativamente).
