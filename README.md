# Gestión de Consultas de Leads

**Ecosistema de Automatización IA Autónomo para Negocios**
Entrega Final · Automatización de Procesos con IA · Coderhouse
Fabrizio Balbi · Inmobiliaria Balbi, Rosario

Sistema que centraliza las consultas de clientes llegadas por WhatsApp, teléfono, correo, Instagram y el CRM inmobiliario. Cada consulta se registra en Notion, la IA redacta un borrador de respuesta y un revisor humano lo aprueba antes de que salga al cliente.

---

## Contenido

| Archivo | Qué es |
| --- | --- |
| `Documentacion-Entrega-Final.pdf` | Documentación completa. |
| `Gestion de Consultas de Leads.json` | Lógica del flujo exportada desde n8n. 21 nodos. |
| `screenshots/` | Capturas de evidencia del sistema funcionando. |
| `Notion-Dashboard` | Enlace al panel de control público. |

---

## Enlaces

**Dashboard de control (público)**
https://evanescent-silkworm-a3b.notion.site/AI-Dashboard-3d5f48dfcec2808f8420c14178fd2158

**Base de datos en modo lectura**
https://evanescent-silkworm-a3b.notion.site/Sistema-AI-3d0f48dfcec280e78d6ef16ffb623ea4

**Video demo**
https://drive.google.com/file/d/14iUa4K3GMzOddmEJ5DfGpizCu6lf0LtI/view?usp=sharing

---

## Dónde está cada criterio

| Criterio | Ubicación |
| --- | --- |
| Mapa de arquitectura del sistema | PDF, sección 2 |
| Manual operativo de estructuras de datos | PDF, sección 3 |
| Estrategia de optimización de costos | PDF, sección 4 |
| Malla de seguridad, privacidad y resiliencia | PDF, sección 5 |
| Dashboard de control ejecutivo | Enlace de arriba, descrito en la sección 6 del PDF |

---

## Stack

**Orquestador** — n8n (self-hosted)
**Base de datos** — Notion, dos tablas vinculadas: AI Requests y Clientes
**Procesamiento IA** — Anthropic, Claude Haiku 4.5
**Canal de salida** — Slack, canal `#todo-n8n`

---

## Cómo funciona

El disparador es un Notion Trigger en modo `pageAddedToDatabase`, que solo reacciona a filas nuevas.

Recorrido principal:

```
Notion Trigger → Get Request → Validate Request → Set Processing Status
→ Generate AI Response → Save AI Response → Notify Reviewer (Slack)
→ espera de aprobación → Check Approval → Save Final Settings
→ Send Approved Review (Slack, dentro del hilo)
```

**Cuatro rutas de contingencia**

- `Handle Validation Error` — consulta sin texto o en estado incorrecto
- `Handle AI ERROR` — fallo de la API de Anthropic
- `Handle Rejected` — el revisor descarta la respuesta
- `Handle Timeout` — nadie revisa dentro de la ventana de aprobación

**Validación humana**

El flujo se detiene después de generar el borrador y consulta el estado cada 30 segundos, hasta 5 veces. Ninguna respuesta llega al cliente sin aprobación.

**Protección contra bucles infinitos**

Triple: el trigger ignora las actualizaciones que hace el propio flujo, `Validate Request` exige `Status = Pending`, y el loop de revisión está acotado por contador mediante `$runIndex`.

---

## Evidencias

La carpeta `screenshots/` documenta el sistema en funcionamiento: el flujo completo en n8n, el listado de ejecuciones, las cuatro rutas de error activándose, el camino exitoso de punta a punta, la estructura de la base en Notion con su relación entre tablas, el dashboard publicado y el hilo de Slack con el pedido de aprobación y la respuesta final anidada.

---

## Seguridad

El JSON exportado no contiene claves ni tokens: n8n almacena las credenciales por referencia de identificador, de modo que el archivo es seguro de publicar.

El prompt de la IA recibe únicamente el texto de la consulta. Los datos de contacto viven en una tabla que el flujo no lee, así que ningún dato personal identificable viaja a la API.
