# English Fluency — CLAUDE.md

## Objetivo
App de **fluência em inglês americano por VOZ** para falantes de português brasileiro. O usuário
**conversa falando**; o coach **Alex** **responde falando como um nativo**, mantém uma conversa
real, corrige com leveza a cada turno e acompanha a evolução (nível CEFR, streak, erros
recorrentes, vocabulário, repetição espaçada). **Meta: o aluno falar como um americano nativo.**

## Colaboração entre agentes (AGENTS-COLLAB)
Este projeto adota a metodologia **AGENTS-COLLAB** para coordenar múltiplos agentes entre sessões.
- **Ordem de leitura:** `AGENTS-COLLAB.md` (estado vivo: decisões ativas, armadilhas, handoff) →
  este `CLAUDE.md` (convenções permanentes) → `docs/` e specs → `skill/` (pedagogia) → código.
- **Elenco de 13 agentes** especializados em `.claude/agents/` (mapa de faixas em
  `.claude/agents/README.md`).
- **Bootstrap:** o hook `SessionStart` (`.claude/settings.json`) injeta
  `docs/colaboracao/bootstrap.md` no contexto no início de cada sessão.
- **Handoff:** ao encerrar, atualize o *Handoff mais recente* do `AGENTS-COLLAB.md` (template em
  `docs/colaboracao/handoff-template.md`). Decisão estável → promova para cá.
- Metodologia detalhada: `docs/colaboracao/metodologia.md`.

## Stack
- **App (full-stack):** **TanStack Start** (TypeScript) — React 19 + Vite + servidor **Nitro** ·
  roteamento `@tanstack/react-router` · dados `@tanstack/react-query` · UI **Tailwind v4** +
  shadcn/ui (Radix) · toasts `sonner`. (Stack padrão atual do Lovable.)
- **Voz (núcleo do produto):** **OpenAI Realtime API** (fala → fala) sobre **WebRTC**. Baixa
  latência, interrompível. A chave fica no servidor; o cliente usa **token efêmero**.
- **Emissor do token:** **server route** `src/routes/api/realtime-token.ts` — lê
  `process.env.OPENAI_API_KEY`, faz `POST /v1/realtime/sessions` (modelo `gpt-realtime`, fallback
  `gpt-4o-realtime-preview`, voz americana, VAD do servidor) e devolve a sessão efêmera.
- **Cliente de voz:** `src/lib/realtime-client.ts` (WebRTC + data channel). Instruções do Alex em
  `src/lib/alex-prompt.ts` (derivadas de `skill/`). Correções **estruturadas** via tool-call
  `submit_corrections` no data channel.
- **Persistência/auth do progresso (planejado, P2):** **Supabase** (Postgres + RLS), espelhando o
  agrodecision. Ainda não adicionado ao app.
- **Cérebro pedagógico:** persona **Alex** em `skill/SKILL.md` + `skill/references/` (mantida
  pelos agentes `pedagogo`/`pronuncia`).
- **Integração:** **Lovable** ativa — NÃO remover `.lovable/` nem `.env`.

## Convenções (inegociáveis)
- **Segredos no servidor.** `OPENAI_API_KEY` NUNCA vai ao cliente. O front só recebe um token
  efêmero curto, emitido pela edge function `realtime-token`.
- **Conversa em inglês, UI em PT-BR.** A fala com o Alex é em inglês; rótulos, botões e
  explicações da interface são em português natural.
- **Pedagogia (guardrail do produto):** manter o aluno **falando** (volume de output é o que
  gera fluência). **Comunicação antes de perfeição** na conversa. Corrigir todo turno, mas no
  **máximo 3–4 correções** de maior valor (bloqueia sentido > padrão recorrente > naturalidade)
  — nunca sufocar. Empurrar para o natural/nativo só depois que a frase já está correta.
- Antes de qualquer operação destrutiva (apagar dados, migration no remoto): inspecionar o
  estado e **CONFIRMAR** a estratégia comigo.
- **Nunca force-push.** Preservar histórico. **PR para a main, nunca commit direto.**
- **Sempre preservar `.lovable/` e `.env`.**
- **Privacidade:** áudio e progresso são do usuário. Não persistir áudio; transcrição/erros só
  com consentimento. Nada de venda de dados.
- Ao concluir cada sessão com mudanças relevantes: **ATUALIZAR este `CLAUDE.md`** (foco/fase,
  decisões, schema recente, arquivos tocados; resumir o debugging) e o handoff do
  `AGENTS-COLLAB.md` antes de encerrar — sempre via branch + PR.

## Comandos (scripts reais do `package.json`)
- dev: `npm run dev` · build: `npm run build` · `npm run build:dev` · preview: `npm run preview`
- lint: `npm run lint` (ESLint) · format: `npm run format` (Prettier)
- typecheck: `npx tsc --noEmit` (não há script dedicado — rode antes de entregar)
- *(quando o Supabase entrar, P2)* tipos: `npm run db:types` · migrations: `supabase db push`

## Schema (progresso do aluno — resumo; **planejado P2**, backend Supabase)
- `profiles` — `id` (= `auth.uid`), `name`, `l1` (default `pt-BR`), `cefr_level` (A1–C2),
  `streak_count`, `last_session_at`.
- `sessions` — `id`, `user_id`, `mode` (chat/roleplay/pronounce/test/review), `started_at`,
  `ended_at`, `summary`.
- `errors` — `id`, `user_id`, `category`, `original`, `corrected`, `note`, `recurring_count`,
  `first_seen`, `last_seen`.
- `vocabulary` — `id`, `user_id`, `term`, `meaning`, `example`, `learned_at`.
- `review_queue` — `id`, `user_id`, `item_type`, `item_ref`, `ease`, `interval_days`, `due_at`,
  `reps` (algoritmo **SM-2**).
- **RLS** isola tudo por `auth.uid()`.

## Server routes / funções
- `src/routes/api/realtime-token.ts` — cunha o token efêmero da OpenAI Realtime (chave no
  servidor), injeta instruções do Alex + voz americana + VAD. Sem `OPENAI_API_KEY` → 503 e o
  front mostra o **card de setup**.
- *(futuro)* `coach-feedback` — estruturaria correções por turno para `errors`/`vocabulary`/
  `review_queue`. **Nota:** hoje as correções já saem estruturadas via tool-call
  `submit_corrections` no data channel (ver `docs/app/voz-realtime.md`).

## Voz / Realtime (resumo — detalhe em `docs/app/voz-realtime.md`)
- Modelo realtime da OpenAI (`gpt-realtime` / `gpt-4o-realtime-preview`), voz americana natural.
- Fluxo: front pede token efêmero → abre WebRTC com a OpenAI → envia a trilha do microfone →
  recebe o áudio do Alex → usa o **data channel** para eventos (transcrição parcial, correções).
- Sem `OPENAI_API_KEY`: o app mostra um **card de setup** (PT-BR) e não quebra.

## Foco atual
**MVP de voz:** tela de conversa + pipeline realtime (edge function + WebRTC) ponta a ponta;
transcrição ao vivo; painel de feedback. Depois: modos (role-play, pronúncia, teste, revisão),
auth + progresso (SM-2). Metodologia AGENTS-COLLAB + elenco de 13 agentes adotados (ver
`AGENTS-COLLAB.md`).

## Decisões pendentes
- Voz/modelo exatos da OpenAI (`alloy` vs `verse`; `gpt-realtime` GA vs `*-preview`) e custo/min.
- Estratégia de correção: **em tempo real** (no data channel) vs **pós-turno** (edge function
  `coach-feedback`). Ver `AGENTS-COLLAB.md`.
- Persistir transcrições? Default: **não** persistir áudio; texto só com consentimento.

## Fora de escopo
- Trocar de stack; reescrever a integração Lovable; reescrever histórico.

## Instruções de compactação
- Ao compactar, preserve: convenções, decisões, schema recente, arquivos tocados e a fase atual.
  Resuma o debugging.
- Preserve também a camada de colaboração: a ordem de leitura (`AGENTS-COLLAB.md` → `CLAUDE.md`)
  e o elenco de agentes em `.claude/agents/`.
