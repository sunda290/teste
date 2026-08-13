# CLAUDE.md — JobOps

Contrato operacional. Se ficar longo, ninguem le. Maximo 100 linhas.

## 1. Restricoes invioláveis (R1 a R6)

- **R1** Nenhum cookie ou credencial de sessao do LinkedIn entra no sistema. O ativo e a conta, nao o dado.
- **R2** Zero auto-apply. O sistema nunca submete formulario.
- **R3** Toda peca gerada passa por revisao humana antes de sair. Responsabilidade e do candidato.
- **R4** Dados de recrutadores sob PIPEDA. Nome e cargo apenas. Sem e-mail scraping, sem enriquecimento.
- **R5** Sem Supabase. PostgreSQL local ou gerenciado, acessado por REST API propria.
- **R6** Nenhuma afirmacao sobre elegibilidade migratoria. Apenas sinalizacao de TEER para verificacao humana.

Qualquer implementacao que viole R1 a R6 e rejeitada no code review.

## 2. Stack e comandos

- Runtime: Node.js 22 LTS, TypeScript strict. API: Express. Banco: PostgreSQL 16.
- Coleta: Apify (`apify-client`) e conector Indeed. Inferencia: Anthropic (Sonnet scoring, Opus redacao).
- DB: `pg` com SQL explicito e migrations numeradas. Proibido ORM pesado.
- Testes: Vitest. Lint: ESLint + Prettier.

Comandos (definidos quando o codigo existir, a partir da FASE 1):

```
npm run build     # tsc
npm test          # vitest
npm run lint      # eslint + prettier --check
npm run migrate   # aplica migrations numeradas
```

Nao instalar dependencias nem escrever codigo de aplicacao antes do GATE 0.

## 3. Regra dos gates

- Cinco fases. Nenhuma fase comeca sem o gate anterior aprovado.
- Cada gate exige `auditor` em PASS e aprovacao escrita em `00-Comando/Gates.md`.
- Nunca avancar de fase sem aprovacao humana explicita e por escrito.
- Se GATE 0 falhar, o projeto encerra. Isso e um resultado valido.

## 4. Vault Obsidian

- Caminho em `OBSIDIAN_VAULT_PATH`. E memoria persistente do projeto, nao backup.
- No inicio de CADA sessao, ler `00-Comando/Estado Atual.md` antes de qualquer decisao.
- Escrever no vault depois de decisoes. Log de decisoes em `00-Comando/Decisoes.md` (append-only).
- O sistema so escreve status `novo` e `triado`. Demais status sao definidos por humano.
- Curriculo master e vault contem dados pessoais e NAO vao para o repositorio de codigo.

## 5. Commits e branches

- Conventional Commits: `feat(recon):`, `fix(normalize):`, `docs(adr):`.
- Branch por fase: `fase/1-recon`, `fase/2-score`. Merge em `main` so apos gate aprovado.
- Tag do gate marca o commit: `git tag gate-1`.

## 6. Proibicao de travessao

- Nunca usar travessao (o caractere em-dash) em qualquer saida destinada a terceiros.
- Vale para cover letters, bullets de curriculo e qualquer peca enviada a empresas.
- Nunca inventar experiencia ausente do `master-resume.md`. Faltando evidencia, sinalizar a lacuna.

## 7. Estado atual

Fase corrente: **FASE 0 (Validacao sem codigo)**. Aguardando GATE 0.
Nao gerar codigo de fases futuras. Cada fase termina em gate humano.
