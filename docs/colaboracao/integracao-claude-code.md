# Integração: AGENTS-COLLAB × Claude Code

Como os conceitos da metodologia se materializam neste repositório (Claude Code / Claude
Agent SDK).

## Subagentes (`.claude/agents/*.md`)

Cada agente é um arquivo Markdown com frontmatter YAML:

```markdown
---
name: nome-em-kebab-case
description: Quando este agente deve ser acionado (usado para delegação automática).
# tools: Read, Grep, Glob, Bash   # opcional; se omitido, herda todas as ferramentas
# model: sonnet | opus | haiku | inherit   # opcional
---

Corpo = prompt de sistema do agente: missão, faixa, o que NÃO faz, convenções, handoff.
```

- **Acionar manualmente:** peça "use o agente `voz-realtime` para ..." ou invoque pela
  ferramenta Task/Agent com `subagent_type` igual ao `name`.
- **Delegação automática:** o `description` é o que o orquestrador lê para escolher o agente.
  Por isso ele começa com "Use este agente quando ...".
- **Ferramentas:** os agentes read-only (`coordenador`, `arquiteto-conversa`, `reviewer`,
  `code-reviewer`) têm `tools` restrito (sem `Edit`/`Write`). Os demais herdam tudo (inclusive
  os MCPs de Supabase e Lovable, quando disponíveis).

## Bootstrap = hook `SessionStart`

`.claude/settings.json` registra um hook que imprime `docs/colaboracao/bootstrap.md` no início
de cada sessão. A saída entra no contexto do agente — é como garantimos que quem chega leia o
estado vivo antes de agir. O hook falha em silêncio (`|| true`) para nunca travar a sessão.

## Faixas e ferramentas relevantes

- `supabase-db` usa o **MCP do Supabase** (`apply_migration`, `execute_sql`, `get_advisors`,
  `generate_typescript_types`). Migrations no remoto exigem aprovação do dono.
- `voz-realtime` mexe na edge function `realtime-token` e no cliente WebRTC; a chave OpenAI é um
  **secret** (Supabase/Lovable), nunca commitada.
- `reviewer` roda `npm run lint` / `npm run typecheck` / `npm run build` e coleta
  `get_advisors`.
- O app é construído/iterado no **Lovable** (MCP do Lovable: `send_message`, `get_diff`, etc.);
  ver `docs/app/lovable.md`.

## Promoção de conhecimento

Quando uma decisão do `AGENTS-COLLAB.md` estabiliza, o `gerente-contexto`/`documentation-writer`
a move para o `CLAUDE.md` (seção apropriada) e a remove do estado vivo. O `CLAUDE.md` já pede,
nas suas instruções, que seja atualizado ao fim de toda sessão relevante — os dois protocolos
se encaixam.
