# Elenco de agentes — English Fluency

Subagentes especializados (metodologia **AGENTS-COLLAB**, ver `docs/colaboracao/metodologia.md`).
Cada um tem uma **faixa** clara para evitar sobreposição. O estado vivo do projeto fica em
`AGENTS-COLLAB.md`; as convenções permanentes em `CLAUDE.md`; a pedagogia do coach em `skill/`.

## Mapa de faixas

| Agente | Aciona quando... | Edita código? |
| --- | --- | --- |
| `coordenador` | a tarefa cruza várias áreas ou precisa ser sequenciada/roteada | ❌ planeja |
| `gerente-contexto` | encerrar sessão, escrever handoff, compactar/promover o estado vivo | ✍️ só docs de estado |
| `arquiteto-conversa` | desenhar a sessão de voz: turn-taking, latência, barge-in, política de correção | ❌ só spec |
| `voz-realtime` | implementar o pipeline Realtime: edge `realtime-token`, WebRTC, áudio, data channel | ✅ |
| `supabase-db` | implementar schema do progresso/RLS/SM-2, aplicar migration, gerar types | ✅ |
| `pedagogo` | comportamento do Alex: correção, CEFR, PT-BR, currículo (mantém `skill/`) | ✅ docs de pedagogia |
| `pronuncia` | pronúncia americana: sons, acento, *connected speech* | ✅ docs de pronúncia |
| `frontend` | tela de conversa, orb, transcrição, painel de feedback, rotas, hooks, Query | ✅ |
| `designer` | design system, tokens, tema, estados do orb, a11y | ✅ visual |
| `tester` | estratégia e escrita de testes (montar o harness) | ✅ testes |
| `reviewer` | portão de qualidade (lint/typecheck/build/advisors) | ❌ reporta |
| `code-reviewer` | revisão de um diff/PR (correção, segurança, reuso) | ❌ reporta |
| `documentation-writer` | README/docs/CLAUDE.md voltados a humanos | ✍️ só docs |

## Como acionar

- **Manual:** "use o agente `voz-realtime` para ..." ou a ferramenta Task/Agent com
  `subagent_type` igual ao `name` do agente.
- **Automático:** o orquestrador lê o campo `description` de cada agente para delegar. Por isso
  cada `description` começa com "Use este agente quando ...".

## Convenção de revisão (duas faixas, de propósito)

- `reviewer` = **saúde do projeto** (passa lint/types/build? respeita convenções? o que dizem os
  advisors do Supabase?).
- `code-reviewer` = **qualidade do diff** (este conjunto de mudanças tem bug, risco de
  segurança, duplicação?).

## Por que estes agentes (e não os do agrodecision)

Este projeto **espelha a estruturação** do repositório irmão `agrodecision`, mas troca as faixas
de domínio: no lugar de `geo-pipeline`/`chatbot`/`edge-functions` de mercado agro, entram os
especialistas de **voz e ensino** — `arquiteto-conversa`, `voz-realtime`, `pedagogo`,
`pronuncia`. Os papéis de processo (`coordenador`, `gerente-contexto`, `reviewer`,
`code-reviewer`, `documentation-writer`, `tester`, `supabase-db`, `frontend`, `designer`) são os
mesmos. Detalhes de integração com o Claude Code: `docs/colaboracao/integracao-claude-code.md`.
