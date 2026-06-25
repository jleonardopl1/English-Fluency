# Spec — Pipeline de voz (OpenAI Realtime + WebRTC)

Dona técnica: agente `voz-realtime`. Desenho da conversa: `arquiteto-conversa`. Esta é a
referência de implementação do **MVP de voz**.

## 1. Server route `realtime-token` (TanStack Start)

Arquivo real: `src/routes/api/realtime-token.ts`. Cunha um **token efêmero** para o cliente
abrir a sessão Realtime sem ver a chave real. (No agrodecision isto seria uma Edge Function do
Supabase; aqui, como o app é TanStack Start, é uma **server route** equivalente.)

- **Secret:** `OPENAI_API_KEY` em `process.env` (Settings → Secrets no Lovable). Se ausente →
  responder `503` com `{ error: "missing_openai_api_key" }`; o front mostra o card de setup.
- **Recomendação (hardening):** devolver ao cliente **apenas** `client_secret` + `expires_at`,
  não a sessão inteira.
- **Request da função:** `POST https://api.openai.com/v1/realtime/sessions` com:
  ```json
  {
    "model": "gpt-realtime",
    "voice": "alloy",
    "modalities": ["audio", "text"],
    "input_audio_transcription": { "model": "whisper-1" },
    "instructions": "<instruções do Alex, de skill/SKILL.md>"
  }
  ```
- **Resposta ao cliente:** repassar **apenas** `client_secret.value` e o `expires_at`. Nunca o
  corpo inteiro nem a chave. CORS liberado para a origem do app.
- `verify_jwt`: se a conversa exigir login, manter `true`; no MVP anônimo pode ser `false`.

## 2. Cliente WebRTC (`src/lib/realtime-client.ts`)

```
1. const { client_secret } = await fetch('/functions/v1/realtime-token').then(r => r.json())
2. const pc = new RTCPeerConnection()
3. pc.ontrack = (e) => audioEl.srcObject = e.streams[0]          // voz do Alex
4. const mic = await navigator.mediaDevices.getUserMedia({ audio: true })
   pc.addTrack(mic.getTracks()[0])                               // microfone do aluno
5. const dc = pc.createDataChannel('oai-events')                 // eventos (transcrição, etc.)
6. const offer = await pc.createOffer(); await pc.setLocalDescription(offer)
7. POST do SDP para https://api.openai.com/v1/realtime?model=gpt-realtime
   Authorization: Bearer <client_secret>      Content-Type: application/sdp
8. await pc.setRemoteDescription({ type: 'answer', sdp: <resposta> })
```

- **Token é curtíssimo (~1 min):** use só para o passo 7. Reabrir sessão = novo token (volte ao 1).
- **HTTPS + permissão:** `getUserMedia` exige `https`/`localhost` e permissão do usuário. Trate
  "permissão negada" como um estado de UI.
- **Estados** emitidos pelo hook: `idle` · `conectando` · `ouvindo` · `falando` · `erro`.

## 3. Eventos no data channel (`oai-events`)

A OpenAI envia eventos JSON. Os que importam para o MVP:

| Evento (tipo) | Uso na UI |
| --- | --- |
| `conversation.item.input_audio_transcription.completed` | linha "You" na transcrição |
| `response.audio_transcript.delta` / `.done` | linha "Alex" (streaming) |
| `response.done` | fim do turno do Alex |
| `error` | mostrar erro amigável |

## 4. Correção (D6 — decisão pendente)

- **MVP:** a correção vem **embutida na fala do Alex** (instruções pedem ❌→✅ · razão ·
  categoria, no máx. 3–4). O front extrai do transcript do Alex para o painel de feedback.
- **Evolução:** uma edge function `coach-feedback` recebe a transcrição do turno e devolve
  correções **estruturadas** (JSON) que alimentam `errors`/`vocabulary`/`review_queue`. Decisão
  do dono + `arquiteto-conversa`.

## 5. Configuração da voz (D5 — decisão pendente)

`voice`: `alloy` (neutra) ou `verse` (mais expressiva). `model`: `gpt-realtime` (GA) com
fallback `gpt-4o-realtime-preview`. Ajustar velocidade/temperatura para soar natural sem
"correr". Validar custo por minuto antes de liberar amplo.
