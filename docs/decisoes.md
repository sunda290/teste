# Log de decisoes (espelho durable)

> Copia versionada de `vault/JobOps/00-Comando/Decisoes.md`. O vault e efemero
> neste ambiente; este arquivo preserva o registro entre execucoes. Append-only.

- 2026-08-13 | Bootstrap executado. CLAUDE.md, ADR 0001, estrutura do vault e perfil criados. Projeto entra em FASE 0.
- 2026-08-13 | Perfil skills-matrix preenchido a partir dos repos publicos (github.com/sunda290). Lacuna confirmada: PostgreSQL.
- 2026-08-13 | Alvo revisado para Alberta (Calgary/Edmonton), relocacao aceita. Toronto/GTA vira fallback. Motivo: maior taxa de acerto junior e mercado mais full-stack.
- 2026-08-13 | Coleta diaria de Alberta adotada em modo manual (agendamento automatico bloqueado por aprovacao). Acumulador durable em docs/fase0/coleta-alberta.md.
- 2026-08-13 | SINAL DE PRODUTO: contato de RH (grupo BR de TI no Canada) apontou potencial de negocio no "metodo". Hipotese a explorar via conversa com RH (dor candidato vs recrutador; disposicao a pagar; concorrentes). NAO altera a disciplina da FASE 0. Reframe: valor esta no metodo (scoring calibrado, deteccao de ghost, qualidade > volume), nao em automacao de auto-apply. Contato de RH e possivel sócia/consultora de dominio.
- 2026-08-13 | INSIGHT ESTRATEGICO (2a resposta do RH):
    (a) Validacao convergente: o RH chegou a mesma tese (qualidade > volume, fugir do "bot infinito", estrutura antes de automacao) partindo do lado do recrutador. Reforca que o ativo e o metodo.
    (b) BIFURCACAO a decidir de proposito: candidate-side (JobOps atual) vs recruiter-side (AI de triagem/entrevista por WhatsApp, ranking por qualidade de resposta). No recruiter-side o RH e o especialista de dominio E o provavel pagador (empresa paga software de RH; candidato desempregado nao).
    (c) SEGMENTACAO de nicho (dada pelo RH): funciona para cargos profissionais/conhecimento, nao para operacional/trade. Beachhead natural = TI, onde o candidato ja esta.
    Cautelas: deteccao de "candidato lendo resposta" e dificil e eticamente arriscada (falso positivo: nervosismo, neurodivergencia, sotaque). Contexto do RH e Brasil (entrevista por WhatsApp e cultura de la); nao assumir transferencia 1:1 para o mercado do Canada.
    Proximo passo sugerido: perguntas ao RH (ja rascunhadas) para localizar dor+pagador e validar se o mercado alvo e BR ou CA.
