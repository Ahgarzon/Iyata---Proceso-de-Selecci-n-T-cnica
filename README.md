# ChatVersa — Proceso de Selección Técnica  
Soy Angel Garzón Sarzosa, Ing en Automática 
## PARTE A

---

# Sistema Inteligente de Cotización Automatizada (n8n)

Automatización end-to-end basada en **WhatsApp + Google Sheets (datasets) + Firestore + Motor de precios determinista + IA redactora + Trello + Human-in-the-Loop (HITL)**.

> **Idea clave:**  
> No es un “workflow aislado”. Es un **sistema modular de automatización**, compuesto por **4 workflows interconectados**, que cubren el ciclo completo de atención comercial:  
> **capturar → validar → calcular → comunicar → escalar a humano → operar → mantener trazabilidad**.

---

## 1) Objetivo del proyecto

Este sistema resuelve un problema real y frecuente en negocios de impresión y servicios personalizados:  
**cotizar de forma rápida, consistente y sin errores**, manteniendo control operativo y evitando decisiones automáticas riesgosas.

El enfoque principal no es “automatizar todo”, sino **automatizar con criterio**, usando IA solo donde aporta valor y reglas deterministas donde se requiere exactitud.

---

## 2) ¿Qué hace el sistema?

- Recibe solicitudes desde **WhatsApp** (extensible a formularios o email).
- Identifica si el cliente es **nuevo o recurrente**.
- Captura y **normaliza información técnica** del requerimiento.
- Valida que estén todos los datos necesarios antes de cotizar.
- Calcula precios usando un **motor 100% determinista** (sin IA).
- Redacta la cotización con IA **sin alterar los valores numéricos**.
- Activa **Human-in-the-Loop (HITL)** cuando el caso lo requiere.
- Actualiza **Trello** para seguimiento operativo.
- Guarda historial completo para trazabilidad y continuidad.

---

## 3) ¿Qué demuestra este proyecto sobre mí?

Este sistema demuestra:

- **Arquitectura modular y escalable** (cada flujo tiene una responsabilidad clara).
- Uso consciente de **IA como asistente, no como juez**.
- Separación estricta entre:
  - Captura de datos
  - Validación
  - Cálculo
  - Comunicación
- Buenas prácticas en **automatización conversacional**.
- Diseño orientado a **operación real**, no solo a demo.
- Capacidad de manejar **edge cases y ambigüedad** con HITL.
- Enfoque “ops-friendly”: reglas y precios se editan en Google Sheets, no en código.

---

## 4) Diagrama general del sistema

<img width="1536" height="1024" alt="Arquitectura Sisteama inteligente de cotizacion automatizada" src="https://github.com/user-attachments/assets/b2a29520-f0ae-4abf-b894-1e5dc550c66c" />



Este diagrama representa el flujo lógico completo, desde la entrada del cliente hasta la operación interna y el cierre del ciclo.


---

## 5) Arquitectura por agentes (visión conceptual)

El sistema está diseñado como una **cadena de agentes especializados**, donde cada uno cumple un rol específico y controlado.

---

### 🟦 Agente 1 — Intake (Captura y registro inicial)

**Rol:** Primer punto de contacto con el cliente.

Funciones principales:
- Recibe mensajes desde WhatsApp.
- Identifica si el cliente ya existe o es nuevo.
- Normaliza datos (ej. tamaños, unidades, expresiones ambiguas).
- Genera y mantiene un **estado conversacional estructurado** (`intake_state_v1`).
- Centraliza toda la información antes de continuar.

**Valor:**  
Evita que datos “sucios” o ambiguos lleguen a las etapas críticas del sistema.

---

### 🟦 Agente 2 — Validador (Completitud técnica)

**Rol:** Asegurar que el sistema tenga todo lo necesario para cotizar.

Funciones:
- Consulta la matriz de productos (Sheets) para saber **qué atributos requiere cada producto**.
- Detecta campos faltantes (medidas, material, cantidad, etc.).
- Pregunta **solo lo estrictamente necesario**, una pregunta por turno.
- Marca el caso como **listo para cotización** cuando está completo.

**Valor:**  
Evita cotizaciones incompletas, suposiciones y reprocesos.

---

### 🟦 Agente 3 — Motor de Precios (Determinista)

**Rol:** Cálculo numérico exacto.

Características:
- No usa IA.
- Usa tablas, reglas y fórmulas.
- Calcula:
  - Precios unitarios
  - Rangos por cantidad
  - Packs especiales
  - IVA
  - Condiciones por tipo de cliente (ej. Colono)
- Devuelve solo números y metadatos claros.

**Valor:**  
Cero alucinaciones. Cero interpretaciones. Máxima confiabilidad.

---

### 🟦 Agente 4 — IA Redactor (Comunicación)

**Rol:** Traducir números a lenguaje humano.

Funciones:
- Recibe el resultado del motor de precios.
- Redacta la cotización de forma clara y profesional.
- No altera valores ni “opina”.
- Si detecta ambigüedad o inconsistencia → solicita HITL.

**Valor:**  
Comunicación profesional sin riesgo financiero.

---

### 🟦 Agente 5 — Memoria / Sync (Trazabilidad)

**Rol:** Persistencia y continuidad del sistema.

Funciones:
- Guarda historial de conversaciones y cotizaciones.
- Mantiene contexto entre interacciones.
- Actualiza Trello según el estado del caso.
- Permite retomar conversaciones con información previa.

**Valor:**  
El sistema “recuerda”, no empieza de cero cada vez.

---

## 6) Buenas prácticas en uso de LLM (diferencial clave)

Este proyecto **no usa IA de forma ingenua**. Aplica prácticas conscientes:

- **Contexto estructurado**, no solo memoria libre.
- El LLM recibe:
  - Estado del caso
  - Últimas interacciones relevantes
  - Rol claro (intake, redactor, etc.)
- Salidas del LLM siempre en **formato controlado (JSON)**.
- Validación posterior antes de usar la salida.
- La IA **nunca decide precios**.
- La IA **nunca completa datos críticos por su cuenta**.

La arquitectura protege el negocio de errores típicos de IA, debido a que los LLM no son optimos para calculos.

---

## 7) Human-in-the-Loop (HITL)

### ¿Cuándo entra un humano?

- Producto fuera de catálogo.
- Combinaciones no existentes en tablas.
- Requerimientos especiales (instalación, acabados no estándar).
- Ambigüedad persistente.
- Riesgo de error comercial.

### ¿Qué sucede?

1. El sistema genera un resumen limpio del caso.
2. Notifica al canal interno (ej. Telegram).
3. Registra el evento HITL en Firestore.
4. El cliente recibe un mensaje de espera controlado.

---

## 8) Workflows incluidos (n8n)

> Cada workflow es simple por separado, pero **juntos forman el sistema completo**.

---

### WF1 — `DV_Refresh`
**Actualización de precios y reglas desde Google Sheets**

<img width="1592" height="708" alt="Captura de pantalla 2026-01-01 203200" src="https://github.com/user-attachments/assets/d686730a-6260-4fd3-9d06-425662f54578" />


- Convierte hojas de cálculo en datasets normalizados.
- Permite actualizar precios sin tocar código.
- Reduce riesgo operativo.

---

### WF2 — `DV-Intake-Agent`
**Conversación, estado y decisión**

<img width="1837" height="514" alt="Captura de pantalla 2026-01-01 203329" src="https://github.com/user-attachments/assets/190df61e-9c28-470c-bf27-4a4aaf4e852b" />


- Entrada principal desde WhatsApp.
- Manejo de estado, validación y decisión.
- Controla el flujo hacia pricing o HITL.

---

### WF3 — `DV-Pricing-Engine`
**Motor determinista de precios**

<img width="1733" height="436" alt="Captura de pantalla 2026-01-01 203352" src="https://github.com/user-attachments/assets/784819ff-ab48-4b48-a557-3735fa92f60b" />


- Aplica reglas, rangos y fórmulas.
- Devuelve resultados numéricos exactos.
- Totalmente desacoplado de IA.

---

### WF4 — `DV_Trello_Sync`
**Seguimiento operativo**

<img width="1710" height="638" alt="Captura de pantalla 2026-01-01 203421" src="https://github.com/user-attachments/assets/0129bee7-5a77-4577-a36b-148da146a666" />


- Actualiza Trello según el estado del caso.
- Facilita la operación diaria del negocio.

---

## 9) Requisitos / dependencias

- n8n (self-host o cloud)
- WhatsApp Cloud API
- Google Sheets
- Google Firestore
- Trello
- LLM (Google Gemini)

---

## 10) Configuración de credenciales

Esta sección describe **cómo configurar todas las credenciales necesarias** para ejecutar el sistema una vez importados los **4 workflows** en n8n.

> ⚠️ **Importante**  
> Todas las credenciales deben configurarse **antes de activar los workflows**.  
> Si alguna credencial no está correctamente configurada, los nodos asociados fallarán en tiempo de ejecución.

---

### 10.1 Google Firestore (estado, clientes y logs)

**Uso dentro del sistema**
- Persistencia del estado conversacional (`clientes`)
- Historial de interacciones por cliente
- Registro de decisiones (pricing, HITL, validaciones)

**Workflows que lo utilizan**
- `DV-Intake-Agent`
- `DV-Pricing-Engine`
- `Trello Sync`

#### Requisitos previos
- Proyecto activo en **Google Cloud Platform**
- Firestore habilitado en modo **Native**
- Cuenta de servicio (Service Account)

#### Pasos de configuración

1. En **Google Cloud Console**:
   - Ir a **IAM & Admin → Service Accounts**
   - Crear una nueva Service Account (ej. `n8n-firestore-sa`)

2. Asignar permisos mínimos recomendados:
   - `Cloud Datastore User`
   - (Opcional) `Cloud Datastore Owner` para ambientes de prueba

3. Generar una **clave JSON** para la Service Account.
4. Descargar el archivo `.json`.

5. En **n8n**:
   - Ir a **Credentials → New**
   - Seleccionar **Google Service Account / Google Firebase**
   - Cargar el archivo JSON completo
   - Verificar que el `project_id` coincida con el usado en los nodos  
     (ejemplo: `dv-system-6aade`)

#### Validación
- Ejecutar manualmente un nodo `Get` o `Upsert`
- Confirmar que el documento aparece en Firestore

---

### 10.2 Google Sheets (datasets de precios y productos)

**Uso dentro del sistema**
- Fuente única de verdad para:
  - Precios
  - Reglas de productos
  - Variantes y atributos técnicos

**Workflow que lo utiliza**
- `DV_Refresh`

#### Métodos de autenticación soportados
- **OAuth** (más simple para pruebas)
- **Service Account** (recomendado para VPS y producción)

#### Configuración recomendada (Service Account)

1. Usar la misma Service Account de Firestore o crear una nueva.
2. Compartir la Google Sheet con el email de la Service Account  
   (permiso mínimo: *Viewer*).

3. En **n8n**:
   - **Credentials → New → Google Sheets**
   - Seleccionar **Service Account**
   - Cargar el JSON correspondiente

4. En los nodos `Google Sheets`:
   - Usar el **Spreadsheet ID** o la URL completa
   - Seleccionar correctamente cada hoja (Volantes, Afiches, Talonarios, etc.)

#### Validación
- Ejecutar el workflow `DV_Refresh`
- Confirmar que se generan/actualizan los archivos JSON de datasets

---

### 10.3 WhatsApp Cloud API (canal de entrada y salida)

**Uso dentro del sistema**
- Recepción de mensajes del cliente
- Envío de preguntas, validaciones y cotizaciones

**Workflow que lo utiliza**
- `DV-Intake-Agent`

#### Datos requeridos
- `Phone Number ID`
- `Access Token` (preferiblemente permanente)

#### Pasos de configuración

1. En **Meta Developers**:
   - Crear una App tipo **Business**
   - Habilitar **WhatsApp Cloud API**
   - Asociar un número (sandbox o productivo)

2. Generar un **Access Token**
3. Copiar:
   - `phone_number_id`
   - `access_token`

4. En **n8n**:
   - **Credentials → New → WhatsApp Cloud API**
   - Configurar el token y el Phone Number ID

5. Configurar el **Webhook** de recepción de mensajes:
   - URL: `https://<tu-n8n>/webhook/<ruta>`
   - Verificar correctamente desde Meta

#### Validación
- Enviar un mensaje de prueba desde WhatsApp
- Confirmar que el workflow se activa

---

### 10.4 Trello (gestión operativa y seguimiento)

**Uso dentro del sistema**
- Creación de tarjetas por solicitud
- Movimiento de tarjetas según estado:
  - Pendiente
  - Validando
  - HITL
  - Cotizado
  - Entregado

**Workflow que lo utiliza**
- `Trello Sync`

#### Datos requeridos
- `API Key`
- `API Token`
- `Board ID`
- `List IDs` (una por estado)

#### Pasos de configuración

1. Obtener credenciales:
   - API Key: https://trello.com/app-key
   - Generar API Token desde la misma página

2. En **n8n**:
   - **Credentials → New → Trello**
   - Ingresar API Key + Token

3. Identificar los IDs de listas del board
4. Configurar el mapeo estado → lista en el workflow

#### Validación
- Ejecutar un caso de prueba
- Confirmar que la tarjeta se crea y se mueve correctamente

---

### 10.5 LLM – Google Gemini (IA conversacional y redacción)

**Uso dentro del sistema**
- Agente de Intake (captura y validación de datos)
- Agente Redactor (construcción del mensaje final)

**Workflows que lo utilizan**
- `DV-Intake-Agent`
- `DV-Pricing-Engine` (redacción)

#### Configuración recomendada

1. Habilitar Gemini en Google AI Studio o GCP
2. Generar una **API Key**
3. En **n8n**:
   - **Credentials → New → Google Gemini / PaLM**
   - Ingresar la API Key

4. Parámetros recomendados:
   - **Temperatura:** baja (0.1 – 0.3)
   - **Prompts con formato estricto**
   - **Salida estructurada (JSON)** cuando aplique

> 🔒 La IA **no calcula precios** ni modifica valores numéricos.  
> Solo interactúa, valida información y redacta mensajes basados en resultados deterministas.

---

### 10.6 Nota importante sobre el motor de precios y confidencialidad

> ⚠️ **Advertencia técnica y legal**

Aunque las credenciales estén correctamente configuradas, **este repositorio por sí solo puede no calcular precios reales**.

Esto se debe a que:

- El **motor de precios determinista completo** se encuentra desplegado en un **VPS privado**.
- Los **datasets finales de precios** son **confidenciales** y pertenecen a la empresa para la cual se desarrolló esta automatización.
- En este repositorio se incluye:
  - La arquitectura
  - La lógica
  - La integración
  - Los flujos de decisión
- Pero **no se incluyen valores comerciales reales**.

Para una ejecución completa en producción se requiere:
- Acceso autorizado al VPS
- Carga de los datasets reales de precios
- Aprobación de la empresa propietaria de los datos

Este repositorio demuestra **capacidad técnica, arquitectura y buenas prácticas**, no expone información sensible.

---

### 10.7 Resumen de credenciales

| Servicio | Tipo de credencial | Uso principal |
|--------|-------------------|---------------|
| Firestore | Google Service Account | Estado, clientes, logs |
| Google Sheets | OAuth / Service Account | Precios y reglas |
| WhatsApp | Cloud API Token | Mensajería |
| Trello | API Key + Token | Seguimiento |
| LLM (Gemini) | API Key | Conversación y redacción |

---


## 11) Ejecución y pruebas

- Prueba demo sin WhatsApp usando ejecución manual.
- Prueba real enviando mensaje por WhatsApp.
- Prueba HITL enviando un caso fuera de tablas.
- Prueba actualización de precios editando Sheets.

---

## 12) Conclusión

Este proyecto no es solo una automatización, es un **sistema de decisión asistida**, diseñado para operar en producción real, con control, trazabilidad y escalabilidad.

Demuestra capacidad para:
- Diseñar arquitecturas complejas.
- Integrar IA de forma responsable.
- Pensar en negocio, operación y riesgo.
- Construir soluciones mantenibles y extensibles.

## Funcionamiento del Flujo en producción
https://drive.google.com/file/d/1c5KcIrLdr4wun08g1hr4-qOW8JgckwFy/view?usp=sharing

## Documentación SOP
https://docs.google.com/document/d/15K3r7WmGM5nYs95dH2XhCrfu6oAOletGCrEaagonr7A/edit?usp=sharing

## explicacion de uno de mis flujos de automatización y un aporte que puedo dar a su empresa (con este video pueden conocer mis capacidades)
https://drive.google.com/file/d/1QKP3oVwtbQKQuZiyCC_JcFyihN924Mhm/view?usp=sharing


## PARTE B
https://docs.google.com/spreadsheets/d/1h-zzIgw2HS-5YR9io5MiZ_kEKqGqorjh7LhEGRNWiDs/edit?usp=sharing

Revisar las Hojas Procedimiento (donde estan las formulas) y Procesado (donde estan los resultados)
