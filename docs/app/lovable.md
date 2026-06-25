# Lovable — projeto e fluxo de build

O app é construído e iterado no **Lovable** (stack padrão: Vite + React + TS, Tailwind/shadcn,
Supabase). Este documento liga o repositório ao projeto Lovable e descreve como evoluí-lo.

## O projeto

- **Nome:** English Fluency
- **Project ID:** `7a67a025-b365-4f78-8342-f3c96129b6cb`
- **Editor:** https://lovable.dev/projects/7a67a025-b365-4f78-8342-f3c96129b6cb
- **Preview:** https://preview--7a67a025-b365-4f78-8342-f3c96129b6cb.lovable.app
- **Workspace:** Jonas Silva (`2fcZSx6s73SxA57F3FDo`)

> Mesma abordagem do repositório irmão **agrodecision**: o app nasce no Lovable e o código é
> versionado no GitHub. A camada de documentação multiagente (este repo) acompanha o app.

## Configurar a voz (obrigatório para falar)

1. No editor do Lovable (ou no painel do Supabase do projeto), adicione o **secret**
   `OPENAI_API_KEY` com uma chave da OpenAI que tenha acesso à **Realtime API**.
2. Garanta que a edge function `realtime-token` está deployada.
3. Abra o preview, permita o **microfone** e toque no orb para começar a falar.

Sem o secret, o app abre normalmente e mostra um **card de setup** em PT-BR (não quebra).

## Conectar ao GitHub (`jleonardopl1/English-Fluency`)

Para o código do app morar neste repositório (ao lado da documentação multiagente):

1. No editor do Lovable: **GitHub → Connect / Create Repository** e aponte para
   `jleonardopl1/English-Fluency`.
2. O Lovable passa a sincronizar o código do app (`src/`, `supabase/`, etc.) com o repo.
3. A camada de docs deste repositório (`AGENTS-COLLAB.md`, `CLAUDE.md`, `.claude/agents/*`,
   `docs/`, `skill/`) convive com o código — exatamente como no agrodecision.

> A conexão GitHub do Lovable é feita pela interface (OAuth) e não pelo MCP. Por isso este passo
> é do dono. Enquanto não conectado, o código vive no repositório gerenciado do Lovable e a
> documentação/estrutura multiagente vive aqui.

## Como evoluir o app (via agentes)

- Mudanças são pedidas ao agente do Lovable em **linguagem natural** (descreva o que quer, não o
  como). O agente `voz-realtime`/`frontend`/`designer` formula o pedido conforme sua faixa.
- Depois de cada mudança: `reviewer` roda lint/typecheck/build; `code-reviewer` revisa o diff;
  `gerente-contexto` escreve o handoff no `AGENTS-COLLAB.md`.
- **Não remover** `.lovable/` nem `.env` (regra do `CLAUDE.md`).
