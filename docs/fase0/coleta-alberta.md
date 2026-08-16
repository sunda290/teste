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
| 2026-08-16 | 20 | 19 | 4 | Dia 2 real (Aug 16). Churn alto. Fluxo novo: ~1-2 relevantes em 3 dias |

Estoque de relevantes por snapshot: ~4-5 (estavel).
FLUXO NOVO (o que importa pro GATE 0): entre 13 e 16-ago (3 dias reais), postagens
novas datadas foram Cold Bore (13-ago) e Raise (14-ago) = 2 vagas, 1 relevante.
Ou seja, entrada de ~0.3-0.5 relevante/dia. Bem abaixo do criterio de 10/dia.
Criterio GATE 0: >= 10 vagas relevantes/dia justifica automacao.
Ressalva: conector Indeed limita ~10 por query; numero real deve ser lido no Indeed.

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

---

## 2026-08-16 (dia 2 real)

Coleta: 20 brutas, 19 unicas de dev (removido 1 machinist CNC). Data de hoje: 2026-08-16.

Relevantes (score >= 50):

1. 60 — Intermediate Software Developer — Wiq Technologies — Calgary — 08-12 — https://to.indeed.com/aag94x6trpnk  (NOVA)
2. 57 — Software Engineer — Cold Bore Technology — Calgary — 08-13 — https://to.indeed.com/aa6sqlmnqjs8  (NOVA)
3. 57 — Full-Stack Software Developer — AFD Petroleum — Edmonton — 08-12 — https://to.indeed.com/aa8sk7zpcrhh
4. 50 — Web Developer — CTOMS — Edmonton — 08-05 — https://to.indeed.com/aafdwryrfvhq

Logo abaixo do corte: Raise (Senior SWE Microservice, Calgary, 08-14, ~48, senior).

Entraram vs dia 1: Wiq (60), Cold Bore SWE (57), Raise (48), Metrolinx (~35), Patterson-UTI (~38).
Sairam vs dia 1: Express Employment (era 62), CardGio (52), THE MAMMOTH (60).

Leitura: o estoque de relevantes fica em ~4-5 por snapshot, mas o FLUXO de vagas
realmente novas e baixo (~1 relevante a cada 2-3 dias na amostra). Dois dias reais
apontam para volume bem abaixo dos 10/dia do GATE 0. Confirmar com o total real do
Indeed antes de decidir.
