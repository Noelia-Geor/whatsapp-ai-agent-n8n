# 🤖 WhatsApp AI Agent with n8n

Agente conversacional para WhatsApp construido con **n8n**, **Evolution API** y **OpenAI/Groq**.  
Diseñado para automatizar la atención al cliente, captación de leads y gestión de citas en negocios reales.

> ⚡ En producción real con clientes activos desde 2025.  
> 🎯 Demo funcional disponible — escribe a [sotoveganoelia@gmail.com](mailto:sotoveganoelia@gmail.com) para acceso.

---

## 🎯 ¿Qué hace este agente?

- Responde consultas de clientes por WhatsApp de forma autónoma 24/7
- Clasifica leads automáticamente según intención y nivel de interés
- Gestiona citas con integración directa a Calendly
- Transcribe mensajes de audio con Whisper
- Envía recordatorios automáticos 24h antes de cada cita
- Genera informes semanales de actividad en Google Sheets
- Reactiva leads inactivos con follow-up automatizado (+48h sin respuesta)
- Detecta si el bot está pausado por el equipo humano y para automáticamente
- Escala conversaciones complejas al equipo cuando es necesario

---

## 🏗️ Arquitectura del sistema

```
┌─────────────────────────────────────────────────────────┐
│                   FLUJO PRINCIPAL                        │
│                                                         │
│  WhatsApp (usuario)                                     │
│        │                                                │
│        ▼                                                │
│  Evolution API v2  ──webhook──►  n8n                    │
│                                   │                     │
│                          ┌────────▼────────┐            │
│                          │  Filtrar mensaje │            │
│                          │  - fromMe: false │            │
│                          │  - event: upsert │            │
│                          └────────┬────────┘            │
│                                   │                     │
│                          ┌────────▼────────┐            │
│                          │   ¿Es audio?    │            │
│                          └───┬─────────┬───┘            │
│                         SÍ  │         │  NO             │
│                              ▼         ▼                │
│                        Whisper    Buscar cliente        │
│                       (OpenAI)    en Google Sheets      │
│                              │         │                │
│                              └────┬────┘                │
│                                   │                     │
│                          ┌────────▼────────┐            │
│                          │  ¿Bot pausado?  │            │
│                          └───┬─────────┬───┘            │
│                         SÍ  │         │  NO             │
│                              ▼         ▼                │
│                            STOP    Agente de IA         │
│                                    (OpenAI/Groq)        │
│                                        │                │
│                                   Parsear output        │
│                                        │                │
│                          ┌─────────────┼──────────┐    │
│                          ▼             ▼          ▼    │
│                    ¿Cita?       ¿Llamada?    Responder  │
│                    Calendly     Avisar       WhatsApp   │
│                    URL          comercial               │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              WORKFLOWS AUTOMÁTICOS                       │
│                                                         │
│  📊 REPORTE SEMANAL (Lunes 9:00)                        │
│  Cron ──► Leer Sheets ──► Calcular métricas ──► WhatsApp│
│                                                         │
│  🔔 SEGUIMIENTO INACTIVOS (Cada 6h)                     │
│  Cron ──► Leer Sheets ──► Filtrar +48h ──► Mensaje      │
│           ──► Marcar como inactivo en Sheets            │
│                                                         │
│  📅 RECORDATORIO CITAS 24H (Diario 9:00)               │
│  Cron ──► Leer Sheets ──► Filtrar citas mañana          │
│           ──► Recordatorio WhatsApp ──► Marcar enviado  │
│                                                         │
│  🗓️ WEBHOOK CALENDLY                                    │
│  Calendly ──► Parsear datos ──► Actualizar Sheets        │
│              ──► Avisar comercial por WhatsApp          │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Stack tecnológico

| Componente | Tecnología |
|---|---|
| Automatización | n8n (self-hosted) |
| WhatsApp API | Evolution API v2 |
| LLM Principal | OpenAI GPT-4o-mini |
| LLM Alternativo | Groq (Llama 3.3 70B) |
| Transcripción audio | OpenAI Whisper |
| Base de datos | Google Sheets (Service Account) |
| Citas | Calendly Webhook |
| Memoria conversacional | Buffer Window (15 mensajes) |
| Infraestructura | Docker + EasyPanel + VPS |

---

## 📋 Workflows incluidos

### `workflows/geor-agent-workflow.json`
Workflow principal completo con los 4 sistemas integrados:
- **Agente principal** — recibe, procesa y responde mensajes WhatsApp
- **Reporte semanal** — métricas automáticas cada lunes
- **Seguimiento inactivos** — reactivación cada 6h
- **Recordatorio citas** — aviso 24h antes
- **Webhook Calendly** — sincronización automática de reservas

---

## ⚙️ Requisitos

- n8n self-hosted (v1.0+)
- Evolution API v2
- OpenAI API Key
- Groq API Key (opcional, alternativa gratuita)
- Google Sheets + Service Account
- Calendly (plan básico)
- VPS con Docker

---

## 🚀 Instalación

### 1. Importar workflow en n8n
1. Abre tu instancia de n8n
2. **Workflows → Import from file**
3. Selecciona `workflows/geor-agent-workflow.json`

### 2. Configurar credenciales
Sustituye todos los placeholders `YOUR_*` en los nodos HTTP:

| Placeholder | Valor |
|---|---|
| `YOUR_EVOLUTION_API_URL` | URL de tu instancia Evolution API |
| `YOUR_EVOLUTION_API_KEY` | API Key de Evolution API |
| `YOUR_INSTANCE_NAME` | Nombre de tu instancia WhatsApp |
| `YOUR_GOOGLE_SHEET_ID` | ID de tu Google Sheet de leads |
| `YOUR_COMMERCIAL_PHONE_NUMBER` | Número del comercial (con prefijo país) |

### 3. Configurar webhook en Evolution API
Apunta el webhook de tu instancia a:
```
https://TU_N8N_URL/webhook/geor-whatsapp-agent
```

### 4. Activar workflows
Activa el workflow principal. Los cron jobs se activan automáticamente.

---

## 📁 Estructura del repositorio

```
whatsapp-ai-agent-n8n/
├── workflows/
│   └── geor-agent-workflow.json
└── README.md
```

---

## 💡 Decisiones técnicas

**¿Por qué n8n y no código Python puro?**
Velocidad de implementación y mantenimiento. Modificar lógica de negocio sin tocar código es crítico cuando los clientes piden cambios en producción.

**¿Por qué Google Sheets como base de datos?**
Para clientes pequeños/medianos ofrece visibilidad directa sin dashboard adicional. El cliente ve y edita su CRM sin formación técnica.

**¿Por qué memoria de 15 mensajes?**
Balance entre contexto conversacional y coste de tokens. Suficiente para conversaciones comerciales típicas de 5-10 minutos.

**¿Por qué bot_pausado flag?**
Permite al equipo humano tomar el control de una conversación sin desactivar el agente globalmente.

---

## 📊 Métricas en producción

- Tiempo de respuesta promedio: < 3 segundos
- Disponibilidad: 24/7
- Coste operativo estimado: < 10€/mes por cliente

---

## 🔒 Seguridad

- Credenciales nunca hardcodeadas — todas por variables de entorno en n8n
- Google Sheets con acceso restringido por Service Account
- Evolution API en VPS privado
- Flag `bot_pausado` para control manual en cualquier momento

---

## 👤 Autora

**Noelia Soto** — AI Automation Specialist & Technical Founder  
[LinkedIn](https://linkedin.com/in/noeliasotovega) · [GeorLabs](https://georlabs.com) · [Email](mailto:sotoveganoelia@gmail.com)

---

## 📄 Licencia

MIT License — libre para usar, modificar y distribuir con atribución.
