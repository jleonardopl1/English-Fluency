# AGENTS-COLLAB.md — Estado vivo do English Fluency

> **O que é isto.** Este arquivo é o *estado atual* do projeto — o **agora**, não o
> histórico. Ele existe para resolver a "amnésia entre agentes": quando vários agentes de
> IA (e o dono do produto) trabalham no mesmo projeto em sessões diferentes, às vezes em
> paralelo, cada um chega sem saber das decisões recentes. Aqui ficam as **decisões
> ativas**, as **armadilhas descobertas**, o **elenco de agentes** e o **último handoff**.
>
> **Leia isto em ≤ 3 minutos.** Se ficar mais longo, é hora de *promover* o que estabilizou
> para o `CLAUDE.md` (convenções permanentes) ou para uma spec em `docs/`.
>
> Metodologia inspirada em **AGENTS-COLLAB.md** (github.com/Rlealbarili/Agents-Collab.md) e
> na estruturação multiagente do repositório irmão **agrodecision**, adaptada à stack de
> voz deste projeto. Detalhes em `docs/colaboracao/metodologia.md`.

---

## Como usar (ordem de leitura)

```
AGENTS-COLLAB.md  (este — o agora)
        ↓
CLAUDE.md         (convenções permanentes — a fonte da verdade do projeto)
        ↓
docs/ e specs     (detalhes de uma área: app, voz)
        ↓
skill/            (pedagogia do Alex — o cérebro do coach)
        ↓
código
```

- **Ao chegar:** leia este arquivo antes do `CLAUDE.md`. Veja Decisões Ativas e Armadilhas.
- **Ao sair (handoff):** atualize a seção *Handoff mais recente* — o que mudou, o que foi
  testado, o que ficou bloqueado, qual o próximo passo. *Fail-closed:* não deixe o próximo
  agente adivinhar.
- **Quando uma decisão estabiliza:** mova-a para o `CLAUDE.md` e remova daqui (princípio
  "promove para cima"). Este arquivo é vivo, não um log que só cresce.

---

## Elenco de agentes (roster)

Definições completas em `.claude/agents/`. Cada agente tem uma faixa (lane) clara.

| Agente | Faixa (o que faz) | Não faz |
| --- | --- | --- |
| `coordenador` | Orquestra: lê o pedido, escolhe o(s) agente(s), sequencia o trabalho, é dono das Decisões Ativas. | Não implementa código. |
| `gerente-contexto` | Mantém **este** arquivo, escreve/valida handoffs, compacta, promove decisões para o `CLAUDE.md`. | Não decide arquitetura. |
| `arquiteto-conversa` | **Estratégia** da conversa por voz: desenho da sessão Realtime, turn-taking, latência, interrupção (barge-in), VAD, política de correção. | Não implementa a função/o front. |
| `voz-realtime` | **Implementa** o pipeline de voz: edge function `realtime-token`, cliente WebRTC, captura/playback de áudio, data channel, secrets. | Não desenha a pedagogia. |
| `supabase-db` | **Implementa** schema do progresso, RLS, fila SM-2, aplica migrations (com aprovação), gera `types.ts`. | Não desenha o modelo do zero. |
| `pedagogo` | Cérebro do **Alex**: motor de correção, nível CEFR, ensino adaptativo, interferência do PT-BR, currículo. Mantém `skill/`. | Não escreve código de UI. |
| `pronuncia` | Pronúncia americana: sons, acento tônico, *connected speech*, *coaching* de sotaque. Mantém `skill/references/american-pronunciation.md`. | Não mexe no pipeline de rede. |
| `frontend` | App React: tela de conversa, orb de voz, transcrição, painel de feedback, rotas, hooks, TanStack Query. | Não decide design tokens. |
| `designer` | Design system, tokens, tema, estados do orb (idle/ouvindo/falando), acessibilidade. | Não escreve lógica de dados. |
| `tester` | Estratégia e escrita/execução de testes (hoje inexistentes no repo). | Não corrige o bug — reporta e cobre. |
| `reviewer` | Portão de qualidade: `lint`, `typecheck`, `build`, advisors do Supabase. Read-only. | Não edita; reporta. |
| `code-reviewer` | Revisão de diff/PR: bugs de correção, reuso, simplificação, segurança da mudança. Read-only. | Não edita; reporta. |
| `documentation-writer` | README, docs PT-BR, mantém o `CLAUDE.md` preciso (junto do `gerente-contexto`). | Não muda comportamento de código. |

---

## Decisões ativas
<!-- Formato: D# (data, quem decidiu) — decisão. -->

- **D1** (2026-06-25, `coordenador` + dono) — **Camadas de documentação.** O `CLAUDE.md` é a
  fonte das **convenções permanentes**; `AGENTS.md` é só um ponteiro fino; **este**
  `AGENTS-COLLAB.md` é o **estado vivo**. Ordem de leitura acima.
- **D2** (2026-06-25, `coordenador` + dono) — **Elenco de 13 agentes** adotado (ver roster),
  espelhando a estruturação do repositório **agrodecision** e adaptando as faixas para um
  produto de **voz** (entram `arquiteto-conversa`, `voz-realtime`, `pedagogo`, `pronuncia`;
  saem os agentes de geo/chatbot do agro). Revisão separada em `reviewer` (qualidade) e
  `code-reviewer` (diff).
- **D3** (2026-06-25, dono) — **Motor de voz: OpenAI Realtime API (fala → fala) sobre WebRTC.**
  Escolhido por soar mais nativo e ter baixa latência. A `OPENAI_API_KEY` fica **no servidor**;
  o cliente recebe só token efêmero via `realtime-token`.
- **D4** (2026-06-25, `coordenador`) — **Segredo no servidor é inegociável.** Nenhuma chave de
  API no bundle do cliente. Toca a regra do `CLAUDE.md`.
- **D5** (2026-06-25, dono — **resolvida**) — **Voz do Alex: `verse`** (mais expressiva e natural
  que `alloy`). Modelo `gpt-realtime` (fallback `gpt-4o-realtime-preview`). Resta observar o custo
  por minuto na prática.
- **D6** (2026-06-25, build do Lovable — **parcialmente resolvida**) — **Onde nasce a correção.**
  O MVP já emite correções **estruturadas** via tool-call `submit_corrections` no data channel
  (não só texto solto). **Reforçado (2026-06-25):** o `ALEX_INSTRUCTIONS` agora obriga o Alex a
  chamar `submit_corrections` **a cada turno** (lista vazia se nada) — painel consistente. Falta
  decidir se uma função `coach-feedback` dedicada persiste isso em `errors`/`review_queue` (P2)
  ou se o próprio cliente grava.
- **D7** (2026-06-25, build do Lovable) — **Stack real do app: TanStack Start** (React 19 + Vite
  + Nitro), não Vite SPA + Supabase. O token nasce numa **server route**
  (`src/routes/api/realtime-token.ts`), não numa Edge Function Deno. O **esquema multiagente**
  (este documento + `.claude/agents/*`) é o que espelha o agrodecision — **não** a stack. Supabase
  fica para a persistência do progresso (P2).

---

## Armadilhas conhecidas (traps)
<!-- Coisas que mordem quem não sabe. Curtas e específicas. -->

- 🔑 **Sem `OPENAI_API_KEY` o app não fala.** Esperado, não bug. A edge function devolve um
  erro claro e o front mostra um **card de setup** em PT-BR. Configure o secret no Supabase
  (e/ou nos secrets do Lovable) antes de testar voz.
- 🎙️ **WebRTC exige HTTPS + permissão de microfone.** No preview do Lovable é https (ok); em
  dev local lembre que `getUserMedia` só roda em `https`/`localhost`. Sem permissão, a sessão
  nunca inicia — trate o estado "permissão negada" na UI.
- ⏱️ **Token efêmero é de uso único.** Na GA, o token vem no campo top-level **`value`** (`ek_...`);
  use-o **só** para abrir a conexão WebRTC, não o guarde. Reabrir sessão = novo token.
- 🔀 **Realtime é GA, não beta.** Token: `POST /v1/realtime/client_secrets` (corpo com `session.audio`
  aninhado); SDP: `POST /v1/realtime/calls?model=…`. O antigo `/v1/realtime/sessions` retorna
  **"Invalid URL"**. Os eventos de transcrição do Alex viraram `response.output_audio_transcript.*`
  (o cliente trata GA **e** legado).
- 🗣️ **Risco de sufocar o aluno.** A tentação é corrigir tudo. O guardrail do produto é
  **máx. 3–4 correções por turno** e **comunicação antes de perfeição**. Não relaxe isso nas
  instruções do Alex (ver `skill/references/correction-engine.md`).
- 🇧🇷 **Interferência do PT-BR é a arma secreta — e a pegadinha.** Falsos cognatos
  (*pretend≠pretender*, *push≠puxar*), 3ª pessoa do singular sem `-s`, `-i` final, sons de
  *th*. O Alex deve conhecê-los (ver `skill/references/pt-br-interference.md`), mas explicar
  **sem** condescendência.
- 🔒 **Não persistir áudio.** Privacidade por padrão: áudio não é guardado; transcrição/erros
  só com consentimento explícito do aluno.

---

## Dívida técnica priorizada (backlog)
<!-- O projeto está no MVP. Itens nascem aqui e viram tarefas dos agentes. -->

- **P0 — fazer a voz funcionar ponta a ponta:**
  1. `voz-realtime`: edge function `realtime-token` + cliente WebRTC + playback. (em andamento)
  2. `frontend`: tela de conversa com orb e estados (idle/ouvindo/Alex falando).
- **P1 — feedback e transcrição:**
  3. Transcrição ao vivo (You/Alex) via data channel.
  4. Painel de feedback (❌→✅ · razão · categoria) — fonte da correção conforme D6.
- **P2 — progresso e auth:**
  5. `supabase-db`: auth e tabelas (`profiles`, `sessions`, `errors`, `vocabulary`,
     `review_queue`) + RLS por `auth.uid()`.
  6. `pedagogo`: onboarding/placement (CEFR) e SM-2 (ver `skill/references/spaced-repetition.md`).
- **P3 — modos e qualidade:**
  7. Modos: role-play, pronúncia, teste de nível, revisão.
  8. `tester`: harness de testes (hoje inexistente). `reviewer`: portão lint/typecheck/build.

---

## Estado atual por área (snapshot)

- **Voz:** MVP **funcionando** no Lovable (`7a67a025-b365-4f78-8342-f3c96129b6cb`): server route
  `api/realtime-token`, cliente WebRTC (`src/lib/realtime-client.ts`), `VoiceOrb`/`Transcript`/
  `FeedbackPanel`/`SetupCard`. Voz **`verse`**; correções via `submit_corrections` a cada turno.
  `OPENAI_API_KEY` configurada pelo dono; **voz confirmada funcionando**. `typecheck` (tsgo) ✅.
- **Pedagogia:** `skill/` trazido do trabalho anterior (SKILL.md + 11 referências + templates +
  exemplos). É a base de conhecimento do Alex; será injetada nas instruções da sessão Realtime.
- **Banco:** ainda não criado (P2). Schema-alvo definido no `CLAUDE.md`.
- **Docs:** camada AGENTS-COLLAB (este arquivo, `CLAUDE.md`, `AGENTS.md`, `.claude/agents/*`,
  `docs/colaboracao/*`, `docs/app/*`) montada nesta sessão.

---

## Handoff mais recente
<!-- Sempre o topo = o mais recente. Use o template em docs/colaboracao/handoff-template.md -->

### 2026-06-25 (c) · `voz-realtime`
- **Objetivo da sessão:** corrigir erro de voz em produção — "Invalid URL (POST /v1/realtime/sessions)".
- **O que mudou (app Lovable):** migração para os endpoints **GA** da OpenAI Realtime — token via
  `POST /v1/realtime/client_secrets` (corpo `session.audio` aninhado; token no top-level `value`)
  e SDP via `POST /v1/realtime/calls`. O cliente passou a tratar os eventos GA
  (`response.output_audio_transcript.*`) além dos legados. Commit Lovable `1d92f97`.
- **O que foi testado:** `typecheck` do Lovable ✅. **Falta o dono retestar a voz** no preview
  (refresh forte para descartar a sessão antiga).
- **Bloqueios / pendências:** aguardando confirmação do dono de que a voz conecta com os endpoints GA.
- **Próximo passo sugerido:** dono testa; se OK, seguir para P2 (progresso) ou os modos.

### 2026-06-25 (b) · `voz-realtime` + `pedagogo` + `gerente-contexto`
- **Objetivo da sessão:** ligar a voz (chave do dono) e aplicar 2 refinamentos pós-teste.
- **O que mudou (app Lovable):** voz `alloy`→`verse` (corpo principal **e** fallback de
  `realtime-token`); `ALEX_INSTRUCTIONS` agora exige, a cada turno, **resposta falada + chamada de
  `submit_corrections`** (lista vazia se nada) — painel de feedback consistente.
- **O que foi testado:** `typecheck` do Lovable (tsgo) ✅; preview renderiza; **voz confirmada
  funcionando pelo dono**. Revisão de código do caminho de voz ponta a ponta OK. (Smoke test HTTP
  do endpoint não rodou: o egress do sandbox bloqueia `*.lovable.app` — limitação de ambiente, não
  do app.)
- **Bloqueios / pendências:** nenhum. Observar custo/min da OpenAI na prática.
- **Próximo passo sugerido:** P2 (progresso: auth + CEFR + streak + SM-2 no Supabase) ou os modos
  (role-play/pronúncia/teste/revisão).

### 2026-06-25 · `coordenador` + `gerente-contexto` + `voz-realtime`
- **Objetivo da sessão:** montar o projeto seguindo a estruturação multiagente do
  **agrodecision** (sem tocar naquele repo), criar o app de voz no **Lovable** e documentar tudo
  no GitHub (`jleonardopl1/English-Fluency`, branch `claude/brave-albattani-fi9ubq`).
- **O que mudou:** criados `AGENTS-COLLAB.md`, `AGENTS.md`, `CLAUDE.md`, `.claude/agents/*`
  (13 + README), `.claude/settings.json` (hook SessionStart de bootstrap), `docs/colaboracao/*`
  (metodologia, bootstrap, template de handoff, integração Claude Code) e `docs/app/*`
  (arquitetura, voz-realtime, lovable). Pedagogia anterior reorganizada sob `skill/`. Projeto
  **Lovable** "English Fluency" criado e **1º build do MVP concluído** (server route
  `api/realtime-token`, WebRTC, orb, transcrição, painel de feedback). Docs **reconciliadas** com
  a stack real (TanStack Start — ver D7).
- **O que foi testado:** documentação revisada manualmente; `typecheck`/`lint` do app **não**
  rodados nesta sessão. **App ainda não testado em voz** — depende do `OPENAI_API_KEY` (secret do
  dono).
- **Bloqueios / pendências:** o dono precisa colar `OPENAI_API_KEY` (Settings → Secrets no
  Lovable) para a voz funcionar. D5 (voz/modelo exatos) em aberto. Conexão do app ao GitHub
  `jleonardopl1/English-Fluency` é um passo manual do dono (ver `docs/app/lovable.md`).
- **Próximo passo sugerido:** dono adiciona a chave e testa a voz no preview; depois `frontend`/
  `designer` lapidam os estados do orb e a transcrição; em seguida `supabase-db` inicia o
  progresso (P2: auth + tabelas + SM-2).
