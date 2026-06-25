# Arquitetura do app — English Fluency

App **voice-first**: o aluno fala, o coach **Alex** responde falando como um nativo, corrige com
leveza e acompanha a evolução. Construído no **Lovable** (ver `docs/app/lovable.md`).

## Visão em camadas

```
┌─────────────────────────────────────────────────────────────┐
│  Cliente (React + Vite + TS, Tailwind/shadcn, TanStack Query)│
│  • Tela de conversa: orb de voz + estados                    │
│  • Transcrição ao vivo (You / Alex)                          │
│  • Painel de feedback (❌→✅ · razão · categoria)            │
│  • Modos: Conversa, Role-play, Pronúncia, Teste, Revisão     │
│  • Progresso (nível CEFR, streak, revisão)                   │
└───────────────┬───────────────────────────┬─────────────────┘
                │ 1. pede token efêmero      │ 4. áudio + eventos (WebRTC)
                ▼                            ▼
┌──────────────────────────────┐   ┌────────────────────────────┐
│ Server route (TanStack/Nitro) │   │  OpenAI Realtime API        │
│ api/realtime-token            │   │  (gpt-realtime, voz US)     │
│ • lê process.env.OPENAI_API…  │   │  • fala → fala              │
│ • POST /v1/realtime/sessions  │──▶│  • transcrição da entrada   │
│ • devolve a sessão efêmera    │ 2 │  • data channel de eventos  │
└──────────────────────────────┘   └────────────────────────────┘
                │ 3. (front abre WebRTC direto com a OpenAI usando o token)
                ▼
┌──────────────────────────────────────────────────────────────┐
│ Supabase (Postgres + RLS + Auth) — PLANEJADO (P2)             │
│ profiles · sessions · errors · vocabulary · review_queue(SM-2)│
└──────────────────────────────────────────────────────────────┘
```

## Por que esse desenho

- **Latência baixa e voz nativa:** o áudio trafega **direto** entre o navegador e a OpenAI por
  WebRTC; o backend só entra para **emitir o token efêmero** (passo 1–2). Isso é o que faz soar
  como conversa de verdade.
- **Segredo no servidor:** a `OPENAI_API_KEY` vive como secret e é lida na **server route**
  (`process.env`), nunca no bundle do cliente. O cliente recebe só a sessão efêmera (vida curta),
  suficiente para abrir a conexão.
- **Pedagogia plugável:** as instruções do Alex vêm de `skill/SKILL.md` (mantidas pelo
  `pedagogo`) e são injetadas na sessão Realtime pela edge function. Trocar o comportamento do
  coach não exige tocar no pipeline de rede.

## Componentes principais (alvo)

| Camada | Arquivo (real / alvo) | Dono (agente) |
| --- | --- | --- |
| Token (server route) | `src/routes/api/realtime-token.ts` | `voz-realtime` |
| Cliente de voz | `src/lib/realtime-client.ts` | `voz-realtime` |
| Instruções do Alex | `src/lib/alex-prompt.ts` (deriva de `skill/`) | `pedagogo` / `voz-realtime` |
| Tela de conversa + orb | `src/routes/*` + `src/components/VoiceOrb.tsx` | `frontend` / `designer` |
| Transcrição/feedback | `src/components/Transcript.tsx`, `FeedbackPanel.tsx`, `SetupCard.tsx` | `frontend` |
| Progresso *(P2)* | tabelas Supabase + hook de progresso | `supabase-db` / `pedagogo` |
| Pedagogia do Alex | `skill/SKILL.md` + `skill/references/` | `pedagogo` / `pronuncia` |

## Estados da conversa

`idle` → `conectando` → `ouvindo` (aluno fala) → `Alex falando` → (loop) · e `erro`
(sem `OPENAI_API_KEY`, permissão de microfone negada, rede). Cada estado tem um visual do orb
(ver `DESIGN_SYSTEM.md`) e é exposto pelo hook de voz.

## Privacidade

Áudio **não** é persistido. Transcrição e erros só são guardados com consentimento, e sempre
isolados por `auth.uid()` via RLS. Ver as regras no `CLAUDE.md`.
