# Dossier Técnico: Ecosistema de Automatización IA Autónomo
**Proyecto Final de Arquitectura de Flujos de IA**

---

## 1. Mapa de Arquitectura del Sistema y Caso de Uso
* **Caso de Uso Seleccionado:** Triage y Resolución Automatizada de Tickets de Soporte Técnico con Escalado VIP y Validación Humana (Human-in-the-Loop).
* **Stack Tecnológico:**
  * **Orquestador:** Make (Flujo principal automatizado).
  * **Base de Datos / Memoria:** Airtable (Tablas relacionales de clientes y tickets).
  * **Motor de IA:** OpenAI (GPT-4o-mini) con prompts estructurados para clasificación de sentimiento y generación de borradores técnicos.
  * **Canal de Salida y Notificación:** Gmail (Comunicación final al cliente) y Slack (Canal interno para aprobación humana).

**Descripción del Flujo:**
1. **Trigger (Disparador):** El flujo se activa mediante un Webhook que recibe la información del ticket entrante.
2. **Cerebro (Airtable):** Consulta la base de datos para registrar el ticket con estado `Pendiente`.
3. **Procesamiento IA:** GPT-4o-mini analiza el mensaje, determina la urgencia, extrae la categoría del problema y redacta una respuesta sugerida.
4. **Validación Humana (HITL):** El sistema se detiene y envía una notificación interactiva a un canal de Slack con el resumen del ticket y el borrador de la IA, esperando la revisión del operador.
5. **Salida Multicanal:** Una vez aprobado, el sistema envía la respuesta final por Gmail al cliente utilizando el hilo de conversación correspondiente y actualiza el estado en Airtable a `Aprobado por Humano`.

---

## 2. Manual Operativo de Estructuras de Datos (Airtable)
La infraestructura relacional se compone de dos tablas interconectadas para evitar datos aislados:

* **Tabla 1: Clientes (`Clients`)**
  * `ClientID` (Primary Key / Identificador único)
  * `NombreEmpresa` (Texto)
  * `Correo` (Email de contacto)
  * `NivelVIP` (Single Select: `Estándar`, `Premium`, `Enterprise`)
* **Tabla 2: Tickets de Soporte (`Support_Tickets`)**
  * `TicketID` (Primary Key)
  * `Cliente` (Link to `Clients`)
  * `Asunto` (Texto)
  * `MensajeOriginal` (Long Text)
  * `Estado` (Single Select: `Pendiente`, `Procesado por IA`, `Aprobado por Humano`, `Cerrado`)
  * `RespuestaSugeridaIA` (Long Text)
  * `ErrorLog` (Long Text - Campo reservado para el registro de fallos)

---

## 3. Estrategia de Optimización de Costos y Recursos
Para garantizar la viabilidad financiera del proyecto, se estructuró la siguiente matriz de asignación de modelos:

| Tipo de Tarea | Herramienta / Modelo Asignado | Justificación de Costo y Eficiencia |
| :--- | :--- | :--- |
| **Tareas Mecánicas** (Clasificación de urgencia, extracción de campos, filtros de datos) | **GPT-4o-mini** | Modelo ultrarrápido y de bajo costo. Ideal para procesar altos volúmenes repetitivos de tickets sin impactar el presupuesto. |
| **Tareas de Lectura Densa** (Análisis técnico avanzado de manuales o errores complejos de software) | **Claude 3.5 Sonnet** | Seleccionado exclusivamente para casos críticos de nivel Enterprise donde se requiere razonamiento profundo y comprensión de contexto extenso. |
| **Procesos Masivos / Reportes** (Auditorías semanales y respaldos) | **OpenAI Batch API** | Utilizado para procesar lotes de datos fuera de horario, aprovechando el descuento del 50% en costos de inferencia. |

---

## 4. Malla de Seguridad, Privacidad y Resiliencia
* **Minimización de Datos:** Los flujos solo leen y procesan el correo y nombre del cliente necesarios para la resolución del ticket, omitiendo cualquier dato financiero o personal sensible de acuerdo con normativas de privacidad.
* **Resiliencia y Manejo de Errores:** Se implementaron nodos de manejo de errores conectados a rutas de contingencia (`Error Handlers`). Si la API de OpenAI falla o hay un dato faltante, el sistema captura el error, lo registra automáticamente en el campo `ErrorLog` de Airtable y envía una alerta de fallo al equipo técnico por Slack.
* **Human-in-the-Loop (Semáforo Crítico):** Se configuró una pausa obligatoria previa a la salida de correos. Esto evita alucinaciones de la IA o el "Efecto Metralleta" (envíos masivos automáticos erróneos), asegurando que ningún mensaje crítico salga al exterior sin la autorización explícita de un humano.

---

## 5. Dashboard de Control Ejecutivo
* **Enlace Web de Control:** https://airtable.com/invite/l?inviteId=inv4NG9M5vXJD0SRP&inviteToken=8c89460280a43c19ccfac2ea333bdae8cf2cc3a616bfe966366d47bf82dbf3b1&utm_medium=email&utm_source=product_team&utm_content=transactional-alerts
* Permite al dueño del negocio monitorear en tiempo real los KPIs operativos:
  * Volumen total de tickets procesados.
  * Tasa de éxito vs. tasa de errores del flujo.
  * Estado actual de los tickets en la cola de validación humana.
