────────────────────────────────────────────────────────────────────
🇺🇸 English Fluency · protocolo de colaboração entre agentes (AGENTS-COLLAB)
────────────────────────────────────────────────────────────────────
ANTES de agir, leia nesta ordem:
  1. AGENTS-COLLAB.md  → o ESTADO VIVO (decisões ativas, armadilhas, handoff).
  2. CLAUDE.md         → convenções permanentes (regras inegociáveis).
  3. docs/ e specs     → detalhe da área que você vai tocar (app, voz).
  4. skill/            → a pedagogia do Alex (o que ele diz e ensina).

Regras que ninguém quebra:
  • A conversa falada é em INGLÊS; a UI e as explicações em PT-BR natural.
  • SEGREDO NO SERVIDOR: a OPENAI_API_KEY nunca vai ao cliente — só token efêmero
    via a edge function realtime-token.
  • Pedagogia: manter o aluno FALANDO; comunicação antes de perfeição; no máximo
    3–4 correções por turno. Nunca sufocar, nunca ridicularizar um erro.
  • Nunca force-push. PR para a main, nunca commit direto. Preservar .lovable/ e .env.
  • Inspecionar e CONFIRMAR com o dono antes de qualquer operação destrutiva ou de
    mudar banco remoto.
  • Privacidade: não persistir áudio; transcrição/erros só com consentimento.

Quem é quem: .claude/agents/README.md (elenco de 13 agentes e suas faixas).

AO SAIR: atualize o "Handoff mais recente" em AGENTS-COLLAB.md
(template: docs/colaboracao/handoff-template.md). Fail-closed: não deixe o
próximo agente adivinhar.
────────────────────────────────────────────────────────────────────
