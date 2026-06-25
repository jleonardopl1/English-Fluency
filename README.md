# 🇺🇸 English Fluency — fale e fique fluente em inglês americano

> Um app de **conversa por voz** com **Alex**, seu coach pessoal de inglês americano para
> falantes de **português brasileiro**. Você **fala**, ele **responde falando como um nativo**,
> conversa de verdade, corrige com leveza e acompanha sua evolução — até você falar como um
> americano. 🎙️

Este repositório é, ao mesmo tempo:

1. **O app** (construído no **Lovable**) — voz em tempo real via **OpenAI Realtime + WebRTC**.
2. **A documentação e a estrutura multiagente** que organiza como ele é construído e mantido,
   seguindo a metodologia **AGENTS-COLLAB** do repositório irmão *agrodecision*.
3. **O cérebro pedagógico** do Alex (pasta `skill/`), reaproveitável como skill/plugin do Claude.

---

## 🗣️ O que o app faz

- Você toca no **orb de voz** e **fala em inglês** (mesmo errado!).
- O **Alex** responde **falando**, com voz americana natural e baixa latência — interrompível,
  como uma conversa real.
- A cada turno, um **painel de feedback** mostra as correções mais importantes
  (❌ errado → ✅ certo · motivo curto · categoria), no máximo 3–4 — sem te sufocar.
- Modos: **Conversa Livre**, **Role-play** (entrevista, restaurante, aeroporto, reunião),
  **Pronúncia**, **Teste de Nível** e **Revisão** (repetição espaçada).
- **Progresso** entre sessões: nível CEFR, streak, erros recorrentes, vocabulário e fila SM-2.

A pedagogia (tom, motor de correção, interferência do PT-BR, pronúncia americana, currículo
A1→C2) vive em [`skill/`](skill/) e é injetada nas instruções da sessão de voz.

---

## 🚀 Rodando / configurando

O app é construído no **Lovable** — detalhes e links em
[`docs/app/lovable.md`](docs/app/lovable.md).

**Para o app falar, configure a chave da OpenAI (Realtime):**

1. Adicione o secret `OPENAI_API_KEY` (no Lovable ou no Supabase do projeto).
2. Garanta o deploy da edge function `realtime-token`.
3. Abra o preview, **permita o microfone** e fale com o Alex.

Sem a chave, o app abre e mostra um **card de setup** em PT-BR (não quebra). Arquitetura
completa em [`docs/app/arquitetura.md`](docs/app/arquitetura.md) e o pipeline de voz em
[`docs/app/voz-realtime.md`](docs/app/voz-realtime.md).

---

## 🤖 Estrutura multiagente (AGENTS-COLLAB)

Seguindo o esquema do repositório **agrodecision** (sem modificá-lo), a construção é coordenada
por um **elenco de 13 agentes** especializados e três camadas de documentação:

| Camada | Arquivo | Papel |
| --- | --- | --- |
| Estado vivo | [`AGENTS-COLLAB.md`](AGENTS-COLLAB.md) | o *agora*: decisões ativas, armadilhas, roster, handoff |
| Convenções | [`CLAUDE.md`](CLAUDE.md) | fonte da verdade: objetivo, stack, regras, schema, comandos |
| Ponteiro | [`AGENTS.md`](AGENTS.md) | aponta para os dois acima |

**Ordem de leitura:** `AGENTS-COLLAB.md → CLAUDE.md → docs/ e specs → skill/ → código.`

Os agentes ficam em [`.claude/agents/`](.claude/agents/) (mapa de faixas no
[README](.claude/agents/README.md)): `coordenador`, `gerente-contexto`, `arquiteto-conversa`,
`voz-realtime`, `supabase-db`, `pedagogo`, `pronuncia`, `frontend`, `designer`, `tester`,
`reviewer`, `code-reviewer`, `documentation-writer`. A metodologia está em
[`docs/colaboracao/`](docs/colaboracao/).

---

## 📁 O que tem dentro

```
English-Fluency/
├── AGENTS-COLLAB.md          # estado vivo do projeto (o agora)
├── CLAUDE.md                 # convenções permanentes (fonte da verdade)
├── AGENTS.md                 # ponteiro fino para os dois acima
├── README.md                 # este arquivo
├── DESIGN_SYSTEM.md          # intenção de design (orb de voz, tokens, a11y)
├── LICENSE                   # MIT
├── .claude/
│   ├── settings.json         # hook SessionStart (bootstrap)
│   └── agents/               # elenco de 13 agentes especializados
├── docs/
│   ├── colaboracao/          # metodologia AGENTS-COLLAB, bootstrap, handoff, integração
│   └── app/                  # arquitetura, pipeline de voz, Lovable
├── skill/                    # 🧠 a pedagogia do Alex (skill/plugin do Claude)
│   ├── SKILL.md              #    comportamento principal do coach
│   ├── references/           #    PT-BR interference, pronúncia, CEFR, correção, SM-2...
│   ├── templates/            #    progress.md (memória), log de sessão
│   └── examples/
└── .claude-plugin/           # marketplace (aponta o plugin para ./skill)
```

---

## 🧠 O coach pedagógico (skill/)

A pasta [`skill/`](skill/) é o "cérebro" do Alex e também um **skill/plugin do Claude** instalável
por conta própria — útil para praticar inglês por texto no Claude, ou como base de conhecimento
das instruções de voz do app. Veja [`skill/SKILL.md`](skill/SKILL.md). Diferencial: tudo é
**focado em brasileiros aprendendo inglês americano** — dos falsos cognatos ao "i" no fim das
palavras.

---

## 🙏 Inspiração & créditos

- Estruturação multiagente inspirada no repositório **agrodecision** (metodologia
  **AGENTS-COLLAB.md** — github.com/Rlealbarili/Agents-Collab.md).
- Pedagogia construída a partir do melhor de coaches de idiomas open-source (interview-coach,
  fluent/SM-2, english-coach, lang-tutor), adaptada ao falante de PT-BR.

*Licença MIT. Bora ficar fluente — you've got this. 🚀*
