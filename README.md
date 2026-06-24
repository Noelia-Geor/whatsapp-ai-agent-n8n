# 🤖 WhatsApp AI Agent with n8n

Agente conversacional para WhatsApp construido con **n8n**, **Evolution API** y **Groq/OpenAI**.  
Diseñado para automatizar la atención al cliente, captación de leads y gestión de citas en negocios reales.

> ⚡ En producción real con clientes activos desde 2025.

---

## 🎯 ¿Qué hace este agente?

- Responde consultas de clientes por WhatsApp de forma autónoma 24/7
- Clasifica leads automáticamente según intención y nivel de interés
- Gestiona citas con integración directa a Calendly
- Envía recordatorios automáticos 24h antes de cada cita
- Genera informes semanales de actividad en Google Sheets
- Reactiva leads inactivos con follow-up automatizado
- Escala conversaciones complejas al equipo humano cuando es necesario

---

## 🏗️ Arquitectura

```
WhatsApp (usuario)
       ↓
Evolution API v2  ←→  Webhook
       ↓
    n8n (orquestador)
       ↓
  ┌────┴────┐
  │         │
Groq      OpenAI
(Llama)  (GPT-4o)
  │         │
  └────┬────┘
       ↓
Google Sheets (memoria + logs)
       ↓
Calendly (gestión de citas)
       ↓
WhatsApp (respuesta)
```

---

## 🛠️ Stack tecnológico

| Componente | Tecnología |
|---|---|
| Automatización | n8n (self-hosted) |
| WhatsApp API | Evolution API v2 |
| LLM Principal | Groq (Llama 3.3 70B) |
| LLM Premium | OpenAI GPT-4o-mini |
| Base de datos | Google Sheets |
| Citas | Calendly Webhook |
| Infraestructura | Docker + EasyPanel + Hostinger VPS |

---

## 📋 Workflows incluidos

### 1. `main-agent.json`
Workflow principal del agente. Recibe mensajes de WhatsApp, procesa con LLM y responde.  
Incluye: detección de intención, gestión de contexto, clasificación de lead, routing a subflows.

### 2. `appointment-reminder-24h.json`
Revisa citas programadas en Google Sheets y envía recordatorio automático 24h antes.  
Ejecutado por cron job diario.

### 3. `weekly-report.json`
Genera informe semanal de métricas: mensajes procesados, leads captados, citas agendadas.  
Envía resumen al equipo por WhatsApp.

### 4. `inactive-followup.json`
Detecta leads que no han respondido en X días y envía mensaje de reactivación automático.

### 5. `calendly-webhook.json`
Recibe eventos de Calendly (nueva cita, cancelación, reprogramación) y actualiza Google Sheets.

---

## ⚙️ Requisitos

- n8n self-hosted (v1.0+)
- Evolution API v2
- Cuenta de Groq (free tier suficiente para empezar)
- Cuenta de OpenAI (para plan premium)
- Google Sheets + Service Account
- Cuenta de Calendly (plan básico)
- VPS con Docker (recomendado: Hostinger KVM2+)

---

## 🚀 Instalación

### 1. Clonar el repositorio
```bash
git clone https://github.com/Noelia-Geor/whatsapp-ai-agent-n8n.git
cd whatsapp-ai-agent-n8n
```

### 2. Importar workflows en n8n
1. Abre tu instancia de n8n
2. Ve a **Workflows** → **Import from file**
3. Importa cada archivo `.json` de la carpeta `/workflows`

### 3. Configurar credenciales en n8n
- **Evolution API:** URL de tu instancia + API Key
- **OpenAI:** API Key
- **Groq:** API Key
- **Google Sheets:** Service Account JSON
- **Calendly:** Webhook URL

### 4. Activar workflows
Activa primero `main-agent.json`, luego el resto en cualquier orden.

---

## 📁 Estructura del repositorio

```
whatsapp-ai-agent-n8n/
├── workflows/
│   ├── main-agent.json
│   ├── appointment-reminder-24h.json
│   ├── weekly-report.json
│   ├── inactive-followup.json
│   └── calendly-webhook.json
├── prompts/
│   └── system-prompt.md
├── docs/
│   └── architecture.md
└── README.md
```

---

## 💡 Decisiones técnicas

**¿Por qué n8n y no código Python puro?**  
Velocidad de implementación y mantenimiento. n8n permite modificar lógica de negocio sin tocar código, lo cual es crítico cuando los clientes piden cambios frecuentes en producción.

**¿Por qué Groq como LLM principal?**  
Latencia sub-segundo y tier gratuito generoso. En conversaciones de WhatsApp, la velocidad de respuesta es un factor crítico de experiencia de usuario.

**¿Por qué Google Sheets como base de datos?**  
Para clientes pequeños/medianos, Google Sheets ofrece visibilidad directa de los datos sin necesidad de un dashboard adicional. El cliente puede ver y editar su CRM sin formación técnica.

---

## 📊 Métricas en producción

- Tiempo de respuesta promedio: < 3 segundos
- Disponibilidad: 24/7
- Coste operativo estimado plan básico: < 10€/mes

---

## 🔒 Seguridad y privacidad

- Ningún dato de conversación se almacena en servidores externos salvo los LLM configurados
- Google Sheets con acceso restringido por Service Account
- Evolution API desplegada en VPS privado
- Sin dependencia de servicios de terceros para el core del agente

---

## 👤 Autora

**Noelia Soto** — AI Automation Specialist & Technical Founder  
[LinkedIn](https://linkedin.com/in/noeliasotovega) · [GeorLabs](https://georlabs.com) · [Email](mailto:sotoveganoelia@gmail.com)

---

## 📄 Licencia

MIT License — libre para usar, modificar y distribuir con atribución.
