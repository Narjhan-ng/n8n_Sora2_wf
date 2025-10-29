# 🎬 Sora 2 Agent Workflow

Un workflow n8n avanzato che combina **AI generative** (Google Gemini) con **Sora 2 text-to-video** per trasformare prompt testuali in video cinematografici.

## 📋 Descrizione Generale

Questo workflow offre due modalità operative:

1. **Modalità Manuale** - Esecuzione diretta con un prompt predefinito
2. **Modalità Chat** - Interazione tramite chat dove un AI Agent ottimizza i prompt prima della generazione video

## 🔄 Flusso di Esecuzione

### **Ramo 1: Simple Version (Manuale)**

```
Manual Trigger 
    ↓
Video Prompt (prompt fisso)
    ↓
HTTP Request → Kie.AI (crea task video)
    ↓
Wait (10 secondi)
    ↓
HTTP Request1 → Kie.AI (recupera stato)
    ↓
If (verifica se success)
    ├→ ✅ Success: Edit Fields → estrae URL video
    └→ ❌ Pending: Ritorna a Wait
```

### **Ramo 2: Agent Version (Chat)**

```
Chat Trigger
    ↓
AI Agent (Google Gemini)
    ├→ Riceve prompt grezzo da utente
    └→ Elabora e ottimizza prompt per Sora 2
    ↓
Send Video Prompt → Kie.AI (crea task)
    ↓
Wait1 (10 secondi)
    ↓
Get Video → Kie.AI (verifica completamento)
    ↓
If1 (controlla stato)
    ├→ ✅ Success: Video URL → estrae e ritorna URL
    └→ ❌ Pending: Ritorna a Wait1
```

## 🛠️ Componenti Principali

### Nodes Utilizzati

| Node | Funzione |
|------|----------|
| **When clicking 'Execute workflow'** | Trigger manuale per avviare il workflow |
| **When chat message received** | Trigger per chat interattiva |
| **AI Agent** | Ottimizza i prompt usando Google Gemini 2.5 Pro |
| **Google Gemini Chat Model** | Modello LLM per ottimizzazione prompt |
| **HTTP Request (Kie.AI)** | Invia prompt a Sora 2 API per generare video |
| **Wait** | Pausa prima di verificare lo stato |
| **If** | Verifica se il video è completato (state === "success") |
| **Edit Fields / Set** | Estrae l'URL video dal risultato |

## 🎯 Cosa Fa

### Modalità Manuale
- Prende un prompt hardcodato: *"Una navicella spaziale che spara un raggio di energia e distrugge un pianeta."*
- Lo invia a Sora 2 con parametri:
  - `n_frames: 1` (numero di frame)
  - `aspect_ratio: landscape`
  - `remove_watermark: true`
- Attende il completamento e restituisce l'URL del video

### Modalità Chat
- L'utente invia un messaggio con un concetto video
- **Google Gemini** trasforma il concetto in un prompt dettagliato e cinematografico
- Il prompt ottimizzato viene inviato a Sora 2 con:
  - `n_frames: 2` (numero di frame)
  - `aspect_ratio: landscape`
  - `remove_watermark: true`
- Polling continuo dello stato fino al completamento
- Restituisce l'URL del video generato

## ⚙️ Configurazione Richiesta

### Credenziali Necessarie

1. **Kie.AI API Key**
   - Endpoint: `https://api.kie.ai/api/v1/jobs/createTask`
   - Autenticazione: HTTP Header Auth
   - ID Placeholder: `YOUR_KIE_AI_CREDENTIAL_ID`

2. **Google Gemini API Key** (solo per modalità Chat)
   - Modello: `models/gemini-2.5-pro`
   - ID Placeholder: `YOUR_GOOGLE_GEMINI_CREDENTIAL_ID`

### Setup

1. Importa il file JSON in n8n
2. Configura le credenziali:
   - Sostituisci `YOUR_KIE_AI_CREDENTIAL_ID` con il tuo Kie.AI API key
   - Sostituisci `YOUR_GOOGLE_GEMINI_CREDENTIAL_ID` con il tuo Google Gemini API key
3. Aggiorna i `webhookId` se necessario
4. Attiva il workflow

## 🌐 API Endpoints Utilizzati

### Kie.AI
- **Create Task**: `POST https://api.kie.ai/api/v1/jobs/createTask`
  - Genera il video da prompt
  
- **Record Info**: `GET https://api.kie.ai/api/v1/jobs/recordInfo?taskId={taskId}`
  - Verifica lo stato e recupera l'URL finale

## 📊 Parametri Sora 2

```json
{
  "model": "sora-2-text-to-video",
  "input": {
    "prompt": "{{ $json.video_prompt }}",
    "aspect_ratio": "landscape",
    "n_frames": "1 o 2",
    "remove_watermark": true
  }
}
```

## ⏱️ Timing

- **Wait nodes**: 10 secondi tra verifiche
- **Polling**: Continua a controllare fino a `state === "success"`
- **Timeout**: Se il video non si completa, il polling continua indefinitamente (considerare un limite massimo di tentativi)

## 🎨 Modalità AI Agent - System Prompt

L'AI Agent è configurato come un **esperto di prompt video AI** che:
- Trasforma concetti grezzi in descrizioni cinematografiche dettagliate
- Ottimizza per realismo e qualità visiva
- Specifica:
  - Soggetti principali (aspetto, movimento, abbigliamento)
  - Ambientazione (luogo, illuminazione, atmosfera)
  - Stile camera (angolazione, lente, movimento)
  - Effetti e illuminazione realistici
  - Tono emotivo della scena

## 🚀 Utilizzo

### Manuale
1. Clicca "Execute workflow"
2. Il sistema genera il video con il prompt predefinito
3. Attendi il completamento e ricevi l'URL

### Chat
1. Invia un messaggio (es: "Una balena che nuota nell'oceano al tramonto")
2. L'AI Agent elabora il prompt
3. Sora 2 genera il video
4. Ricevi l'URL del video completato

## 📝 Note

- I video vengono generati con `remove_watermark: true`
- Il workflow mantiene uno stato di polling fino al completamento
- Per miglioramenti futuri: aggiungere un massimo di tentativi di polling per evitare loop infiniti
- Considerare l'aggiunta di una callback URL (ngrok/webhook) per notifiche push anzichè polling

## 🔐 Sicurezza

Tutti gli ID sensibili sono stati rimpiazzati con placeholder. Prima di usare:
- Sostituisci tutti i placeholder con valori reali
- Mantieni le API key private
- Non condividere il file con credenziali compilate

---

**Workflow Name**: Sora 2 Agent  
**Type**: n8n Automation  
**Ultimo Update**: Ottobre 2025
