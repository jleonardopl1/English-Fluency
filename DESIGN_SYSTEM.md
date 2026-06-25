# DESIGN_SYSTEM.md — English Fluency

Identidade visual **moderna, calma e premium**, mobile-first. O herói é o **orb de voz**, que
comunica o estado da conversa sem texto. Dono: agente `designer`. (Os tokens reais vivem no
código do app — Tailwind/CSS vars; este arquivo é o resumo de intenção.)

## Princípios
- **Calmo para falar.** Cores suaves e muito respiro — o aluno precisa se sentir à vontade para
  errar em voz alta.
- **Tokens, não cores soltas.** Nada de hex hardcoded nos componentes; tudo via CSS vars / Tailwind.
- **Acessível por padrão.** Contraste AA, foco visível, alvos ≥ 44px, `aria-live` na
  transcrição, respeitar `prefers-reduced-motion`.

## Estados do orb de voz
| Estado | Sensação | Animação |
| --- | --- | --- |
| `idle` | em repouso, convidativo | respiração lenta |
| `conectando` | preparando | shimmer/spinner sutil |
| `ouvindo` | captando o aluno | pulsa com o volume do microfone |
| `Alex falando` | resposta nativa | onda ativa sincronizada com a fala |
| `erro` | algo a resolver | estático + cor de alerta + card de ajuda |

## Tipografia & idioma
- UI em **PT-BR**; conteúdo falado em inglês. Fonte legível e amigável; hierarquia clara
  (título da tela, transcrição, feedback).

## Tema
- Claro e escuro. O escuro é o padrão confortável para sessões longas à noite.
