# 🤖 Buenas Prácticas: N8N + Telegram Bot

Este documento contiene las mejores prácticas y soluciones a problemas comunes al desarrollar bots de Telegram con N8N.

---

## 📌 Tabla de Contenidos

1. [Envío de Mensajes con Inline Keyboards](#envío-de-mensajes-con-inline-keyboards)
2. [Estructura de Datos para Botones](#estructura-de-datos-para-botones)
3. [Manejo de Webhooks](#manejo-de-webhooks)
4. [Debugging y Troubleshooting](#debugging-y-troubleshooting)

---

## 🎯 Envío de Mensajes con Inline Keyboards

### ❌ Problema Común

El nodo nativo de **Telegram** en N8N tiene **limitaciones** al enviar `inline_keyboard` con arrays anidados. Aunque el nodo tiene campos para "Reply Markup" e "Inline Keyboard", estos no funcionan correctamente cuando se pasan expresiones dinámicas con arrays de botones.

**Síntomas:**
- Error: `"The value [Array:...] is not supported!"`
- Error: `"can't parse reply keyboard markup JSON object"`
- Los botones no aparecen en Telegram
- El `reply_markup` se envía como `[object Object]` (string) en lugar de JSON

### ✅ Solución Recomendada: Usar HTTP Request

En lugar del nodo de Telegram, usa el nodo **HTTP Request** para llamar directamente a la API de Telegram.

#### Configuración del Nodo HTTP Request

**Parámetros Básicos:**
- **Method:** `POST`
- **URL:** `https://api.telegram.org/bot<TU_BOT_TOKEN>/sendMessage`
- **Authentication:** `None` (el token va en la URL)
- **Send Body:** `ON`
- **Body Content Type:** `JSON`
- **Specify Body:** `Using JSON`

**Campo JSON:**
```javascript
{{ {
  "chat_id": $('telegramTrigger').item.json.message.chat.id || $('telegramTrigger').item.json.callback_query.message.chat.id,
  "text": $('nodoDeLaLogica').item.json.text,
  "reply_markup": {
    "inline_keyboard": $('nodoDeLaLogica').item.json.buttons
  }
} }}
```

**Notas Importantes:**
- ✅ Usa `{{ }}` (sin `=` al inicio) cuando uses "Using JSON"
- ✅ La ruta correcta del chat_id es `message.chat.id`, NO `message.chat_id`
- ✅ Para callback queries, usa `callback_query.message.chat.id`
- ❌ NO uses `JSON.stringify()` - N8N serializa automáticamente
- ❌ NO uses "Using Fields Below" para objetos anidados complejos

---

## 📦 Estructura de Datos para Botones

### Formato Correcto de `inline_keyboard`

Los botones inline deben estructurarse como un **array de arrays**, donde cada sub-array representa una **fila** de botones.

**Para botones verticales (uno por línea):**
```javascript
const OPTIONS = {
  TIPO_EVENTO: [
    [{ text: "🎊 Eventos Sociales", callback_data: "Eventos sociales" }],
    [{ text: "🏢 Corporativo", callback_data: "Conferencias y eventos corporativos" }],
    [{ text: "🎮 E-Sports", callback_data: "E-Sport y Gaming" }]
  ]
};
```

**Para botones horizontales (varios en una línea):**
```javascript
const OPTIONS = {
  CONFIRMACION: [
    [
      { text: "✅ Confirmar", callback_data: "confirmar" },
      { text: "❌ Cancelar", callback_data: "cancelar" }
    ]
  ]
};
```

### Estructura de Respuesta desde el Código JavaScript

Tu código JavaScript debe devolver un objeto con esta estructura:

```javascript
return {
  text: "Mensaje para el usuario",
  buttons: [
    [{ text: "Botón 1", callback_data: "opcion1" }],
    [{ text: "Botón 2", callback_data: "opcion2" }]
  ],
  next_step: "siguiente_paso",
  action: "reply"
};
```

**NO uses:**
```javascript
// ❌ Incorrecto
return {
  text: "Mensaje",
  reply_markup: { inline_keyboard: [...] }  // N8N no lo maneja bien
};
```

---

## 🔗 Manejo de Webhooks

### Obtener el Bot Token

1. Abre Telegram y busca **@BotFather**
2. Envía `/mybots`
3. Selecciona tu bot
4. Haz clic en **"API Token"**
5. Copia el token (formato: `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`)

### Verificar que el Token Funciona

Abre esta URL en tu navegador (reemplaza `<TOKEN>`):
```
https://api.telegram.org/bot<TOKEN>/getMe
```

Si el token es correcto, verás un JSON con información de tu bot.

### Configurar el Webhook en N8N

1. El nodo **Telegram Trigger** debe estar configurado con:
   - **Updates:** `message` y `callback_query`
   - **Credential:** Tu credencial de Telegram con el token

2. N8N automáticamente configura el webhook cuando activas el workflow

---

## 🐛 Debugging y Troubleshooting

### Error: "404 Not Found"

**Causa:** El token del bot es incorrecto o la URL está mal formada.

**Solución:**
1. Verifica el token con `/getMe` (ver arriba)
2. Asegúrate de que la URL sea exactamente: `https://api.telegram.org/bot<TOKEN>/sendMessage`
3. No debe haber espacios ni caracteres extra

### Error: "400 Bad Request: can't parse reply keyboard markup JSON object"

**Causa:** El `reply_markup` se está enviando como string en lugar de objeto JSON.

**Solución:**
1. Usa "Using JSON" en el nodo HTTP Request
2. NO uses `JSON.stringify()` en la expresión
3. Verifica que la sintaxis sea `{{ { ... } }}` (sin `=`)

### Error: "400 Bad Request: chat_id is empty"

**Causa:** La ruta del `chat_id` es incorrecta.

**Solución:**
Usa la ruta correcta:
```javascript
// ✅ Correcto
$('telegramTrigger').item.json.message.chat.id

// ❌ Incorrecto
$('telegramTrigger').item.json.message.chat_id
```

### Los Botones No Aparecen (sin error)

**Causa:** El array de botones está vacío o mal estructurado.

**Solución:**
1. Verifica el OUTPUT del nodo de lógica antes del HTTP Request
2. Asegúrate de que `buttons` sea un array de arrays
3. Cada botón debe tener `text` y `callback_data`

---

## 📚 Recursos Adicionales

- [Telegram Bot API - sendMessage](https://core.telegram.org/bots/api#sendmessage)
- [Telegram Bot API - InlineKeyboardMarkup](https://core.telegram.org/bots/api#inlinekeyboardmarkup)
- [N8N HTTP Request Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)

---

## 🎓 Lecciones Aprendidas

1. **Simplicidad sobre abstracción:** A veces es mejor usar HTTP Request directo que depender de nodos especializados con limitaciones.

2. **Verifica la estructura de datos:** Siempre revisa el OUTPUT de cada nodo para confirmar que los datos tienen el formato esperado.

3. **Consulta la API oficial:** Cuando un nodo de N8N no funciona como esperas, consulta la documentación oficial de la API para entender el formato exacto que necesitas.

4. **Debugging incremental:** Prueba primero con la API directamente (usando curl o el navegador) antes de implementar en N8N.

---

## 🤖 Arquitectura de Bot Conversacional (State Machine)

### Patrón de Máquina de Estados

Un bot conversacional debe manejar el estado de cada usuario. Estructura recomendada:

```javascript
const STEPS = {
  START: 'start',
  TIPO_EVENTO: 'tipo_evento',
  FECHA: 'fecha',
  CIUDAD: 'ciudad',
  // ... más pasos
  CONFIRMACION: 'confirmacion',
  VALIDACION_IA: 'validacion_ia',
  COMPLETADO: 'completado'
};
```

### Switch por Paso Actual

```javascript
switch (currentStep) {
  case STEPS.START:
    response.text = 'Bienvenido! ¿Qué tipo de evento?';
    response.buttons = OPTIONS.TIPO_EVENTO;
    response.next_step = STEPS.FECHA;
    break;
    
  case STEPS.FECHA:
    // Validar input anterior, guardar, pedir siguiente
    if (validarFecha(input)) {
      response.update_data.fecha = input;
      response.next_step = STEPS.CIUDAD;
    } else {
      response.text = 'Fecha inválida, intenta de nuevo';
    }
    break;
  // ... más casos
}
```

### Estructura de Respuesta Estándar

```javascript
return {
  text: 'Mensaje al usuario',
  buttons: null,  // o array de botones
  next_step: 'siguiente_paso',
  update_data: { ...datosActualizados },
  action: 'reply',  // 'reply', 'validate_with_ai', 'cancel_session'
  new_intentos: 0,
  tipoValidacion: 'BOT'  // 'BOT' o 'IA'
};
```

---

## 💾 Persistencia de Sesión con Google Sheets

### Estructura de la Hoja

| chat_id | paso_actual | datos_json | intentos_fallidos | tipoValidacion | ultima_interaccion |
|---------|-------------|------------|-------------------|----------------|-------------------|
| 123456  | ciudad      | {"tipo":"boda"} | 0 | BOT | 2025-12-09T... |

### Flujo de Sesión

```
telegramTrigger → buscarSesion (Google Sheets lookup por chat_id)
  → esNuevoUsuario?
    → TRUE: crearSesion → logicaBot
    → FALSE: logicaBot (con datos existentes)
```

### Actualizar Sesión

```javascript
// En nodo actualizarSesion (Google Sheets appendOrUpdate)
chat_id: $('telegramTrigger').item.json.message?.chat?.id
paso_actual: $('logicaBot').item.json.next_step
datos_json: JSON.stringify($('logicaBot').item.json.update_data)
intentos_fallidos: $('logicaBot').item.json.new_intentos
```

### Campos Temporales en datos_json

Para control del flujo de IA, guarda campos con prefijo `_`:

```javascript
{
  "tipo_evento": "boda",
  "fecha_evento": "25/12/2025",
  "_campo_pendiente": "ubicacion_evento",  // Temporal
  "_errores_ia": ["ciudad inválida"],       // Temporal
  "tipoValidacion": "IA"
}
```

---

## 🧠 Integración con IA (Validación en Dos Capas)

### Capa 1: Validación del Bot (Sin Costo)

```javascript
const Validators = {
  fecha: (text) => {
    const regex = /^(\d{2})\/(\d{2})\/(\d{4})$/;
    if (!regex.test(text)) return { valid: false, error: 'Formato DD/MM/YYYY' };
    return { valid: true, value: text };
  },
  
  ciudad: (text) => {
    // Detectar caracteres sospechosos
    const invalidos = /[\.\/\+\&\%\@\#\$\!\?]/;
    if (invalidos.test(text)) {
      return { valid: false, error: 'Caracteres no válidos' };
    }
    // Detectar comandos
    if (text.startsWith('/')) {
      return { valid: false, error: 'Eso parece un comando' };
    }
    return { valid: true, value: text };
  }
};
```

### Capa 2: Validación con IA (Solo al Final)

La IA se invoca solo cuando TODOS los campos están completos:

```
logicaBot (confirmar) → switchAccion (validate_with_ai)
  → capaValidadorIA (AI Agent)
  → ValidadorIA (procesa respuesta)
  → switchValidacionIA
    → send_to_central: enviar a workflow central
    → ask_field: preguntar campo faltante
    → send_to_error_support: escalar a humano
```

### Prompt de Validación para IA

```
Eres un validador de datos. Valida estos campos:
- ubicacion_evento: debe ser ciudad real o ficticia, NO comandos
- nombre_cliente: caracteres válidos

Responde SOLO en JSON:
{
  "valido": boolean,
  "campo_faltante": "nombre_campo" | null,
  "pregunta_usuario": "Pregunta amigable" | null,
  "errores": ["lista de errores"]
}
```

---

## ✏️ Flujo de Corrección de Datos

### Agregar Opción de Corregir en Confirmación

```javascript
OPTIONS.CONFIRMACION = [
  [{ text: '✅ Confirmar', callback_data: 'confirmar' }],
  [{ text: '✏️ Corregir un dato', callback_data: 'corregir' }],
  [{ text: '❌ Cancelar', callback_data: 'cancelar' }]
];

OPTIONS.MENU_CORRECCION = [
  [{ text: '📅 Fecha', callback_data: 'edit_fecha_evento' }],
  [{ text: '📍 Ciudad', callback_data: 'edit_ubicacion_evento' }],
  [{ text: '⬅️ Volver', callback_data: 'volver_resumen' }]
];
```

### Estados para Corrección

```javascript
case STEPS.MENU_CORRECCION:
  if (incomingCallback.startsWith('edit_')) {
    const campo = incomingCallback.replace('edit_', '');
    response.update_data._campo_editando = campo;
    response.next_step = STEPS.CORRIGIENDO_CAMPO;
  }
  break;

case STEPS.CORRIGIENDO_CAMPO:
  const campo = currentData._campo_editando;
  const validacion = Validators[campo]?.(input);
  if (validacion?.valid) {
    response.update_data[campo] = validacion.value;
    delete response.update_data._campo_editando;
    response.next_step = STEPS.COMPLETADO; // Volver a resumen
  }
  break;
```

---

## 🔄 Bypass de Bot para Validación IA

Cuando la IA pregunta por un campo faltante, el usuario responde directamente a la IA sin pasar por logicaBot:

```
buscarSesion → esValidacionIA (tipoValidacion === 'IA')?
  → TRUE: prepararDatosIA → capaValidadorIA
  → FALSE: logicaBot → switchAccion
```

### Nodo prepararDatosIA

```javascript
const datos = JSON.parse(session.datos_json || '{}');
const campoPendiente = datos._campo_pendiente;

if (campoPendiente && userInput) {
  datos[campoPendiente] = userInput;
}

return {
  update_data: datos,
  contexto_validacion: JSON.stringify(datos)
};
```

---

## ⚠️ Errores Comunes y Soluciones

### Error: "Node hasn't been executed"

**Problema:** Un nodo recibe de múltiples rutas pero intenta leer de una que no se ejecutó.

**Solución:** Usar `$if()` con `isExecuted`:
```javascript
={{ $if($('ValidadorIA').isExecuted, 
     $('ValidadorIA').item.json.data, 
     $('switchAccion').item.json.data) }}
```

### Error: Sesión no se actualiza correctamente

**Problema:** `datos_json` no se guarda con `_campo_pendiente`.

**Solución:** Verificar que `actualizarSesion` lea del nodo correcto:
```javascript
datos_json: $if($('ValidadorIA').isExecuted,
  JSON.stringify($('ValidadorIA').item.json.update_data),
  JSON.stringify($('switchAccion').item.json.update_data))
```

### Error: Callback buttons no funcionan

**Problema:** `chat_id` no se obtiene correctamente.

**Solución:** Manejar ambos casos:
```javascript
$('telegramTrigger').item.json.message?.chat?.id || 
$('telegramTrigger').item.json.callback_query?.message?.chat?.id
```

---

## 📋 Checklist de Implementación

- [ ] Máquina de estados con todos los pasos definidos
- [ ] Validadores para cada campo de texto
- [ ] Google Sheets con estructura correcta
- [ ] Nodo buscarSesion configurado
- [ ] Switch para rutas (reply, validate_with_ai, cancel)
- [ ] AI Agent con prompt estructurado
- [ ] Nodo post-IA que procesa respuesta
- [ ] Switch para acciones de IA (send_to_central, ask_field, error)
- [ ] Bypass para respuestas directas a IA
- [ ] Flujo de corrección de datos
- [ ] Manejo de `$if(isExecuted)` en nodos multi-ruta
- [ ] Fallback para errores de IA
- [ ] Nodo adaptador para integración con flujo web

---

## 🔌 Integración Multi-Canal (Modelo de Datos Canónico)

### El Problema
Cada canal (Web, Telegram, WhatsApp) envía datos con estructuras diferentes. Mantener la lógica de negocio compatible con todos es insostenible.

### La Solución: Universal Data Object (UDO)
Define un **único formato JSON estándar** para tu organización. Todos los canales deben adaptar sus datos a este formato ANTES de entrar al flujo principal.

**Estructura del UDO:**
```javascript
{
  "cliente": { "nombre": "...", "email": "..." },
  "evento": { "tipo": "...", "fecha": "YYYY-MM-DD" },
  "venta": { "paquete": "...", "presupuesto": 100 },
  "metadata": { "origen": "telegram", "timestamp": "..." }
}
```

### Arquitectura de Adaptadores

```mermaid
graph TD
    A[Telegram Bot] -->|Raw JSON| B(Adaptador Telegram)
    C[Web Form] -->|Raw Body| D(Adaptador Web)
    E[WhatsApp] -->|Raw Msg| F(Adaptador WhatsApp)
    
    B --> G{MERGE}
    D --> G
    F --> G
    
    G -->|JSON Canónico (UDO)| H[Core Business Logic]
    H --> I[Calcular Días]
    H --> J[Clasificar Urgencia]
```

### Implementación en Nodos Core
Tus nodos de lógica (`calcularDias`, `clasificarUrgencia`) deben ser agnósticos del origen. Prográmalos para leer del UDO, pero mantén retrocompatibilidad si es necesario:

```javascript
// Ejemplo de lectura robusta
const fecha = input.evento?.fecha || input.body?.fecha_evento || input.fecha_evento;
```

---

## 🏗️ Arquitectura Completa del Bot

### Diagrama del Flujo Final

```
┌──────────────────────────────────────────────────────────────────────┐
│                        TELEGRAM BOT FLOW                              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  telegramTrigger                                                      │
│       │                                                               │
│       ▼                                                               │
│  buscarSesion (Google Sheets lookup)                                  │
│       │                                                               │
│       ▼                                                               │
│  esNuevoUsuario? ─────TRUE────► crearSesion                          │
│       │                              │                                │
│       │FALSE                         │                                │
│       ▼                              │                                │
│  esValidacionIA? ◄───────────────────┘                               │
│       │                                                               │
│  TRUE │                    FALSE                                      │
│       ▼                      │                                        │
│  prepararDatosIA             ▼                                        │
│       │               logicaBot (State Machine)                       │
│       │                      │                                        │
│       ▼                      ▼                                        │
│  capaValidadorIA ◄─── switchAccion ───► HTTP_Request (reply)         │
│       │                      │                     │                  │
│       ▼                      │                     ▼                  │
│  ValidadorIA                 │              actualizarSesion          │
│       │                      │                                        │
│       ▼                      │                                        │
│  validarDatosIA              │                                        │
│       │                      │                                        │
│       ▼                      │                                        │
│  switchValidacionIA          │                                        │
│       │                      │                                        │
│  ┌────┼────────┐             │                                        │
│  │    │        │             │                                        │
│  ▼    ▼        ▼             ▼                                        │
│ ask  send   error     Workflow Central                                │
│ field central support      │                                          │
│  │    │        │           ▼                                          │
│  │    ▼        │    correoErrorSoporte                               │
│  │ adaptar     │                                                      │
│  │ Datos       │                                                      │
│  │    │        │                                                      │
│  │    ▼        │                                                      │
│  │  Merge ─────┴──────► calcularDias → clasificar → validar ...      │
│  │                                                                    │
│  ▼                                                                    │
│ HTTP_Request (pregunta al usuario)                                   │
│  │                                                                    │
│  ▼                                                                    │
│ actualizarSesion (guarda _campo_pendiente)                           │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🎓 Retrospectiva del Proyecto

### Lo Que Funcionó Bien

1. **Validación en 2 Capas**: Reducir llamadas a IA en 80%
2. **Máquina de Estados**: Flujo predecible y debuggeable
3. **Persistencia en Sheets**: Simple y efectivo para MVPs
4. **Nomenclatura Clara**: Debugging más rápido

### Desafíos y Soluciones

| Desafío | Solución |
|---------|----------|
| Nodos multi-ruta | `$if(isExecuted)` |
| Formatos de fecha diferentes | Nodo adaptador |
| Datos anidados (`update_data`) | Leer del nivel correcto |
| Session no persistía `_campo_pendiente` | Leer desde `ValidadorIA` |

### Lecciones Clave

1. **Siempre verifica la estructura de datos** en cada nodo
2. **Usa `$if(isExecuted)`** cuando un nodo recibe de múltiples rutas
3. **HTTP Request > Nodo Nativo** para inline keyboards en Telegram
4. **Campos temporales con `_prefijo`** para datos de control
5. **Nodos adaptadores** para integrar múltiples canales
6. **Documentar en las buenas prácticas** mientras aprendes

### Próximos Pasos Sugeridos

- [ ] Agregar más canales (WhatsApp, Instagram)
- [ ] Dashboard de métricas (tasa de conversión, abandono)
- [ ] Tests automatizados del flujo
- [ ] Rate limiting para evitar abuso
- [ ] Backup automático de sesiones

---

## 📚 Archivos de Referencia del Proyecto

| Archivo | Propósito |
|---------|-----------|
| `telegramLogic.js` | Máquina de estados del bot |
| `prepararDatosIA.js` | Prepara contexto para validación IA |
| `validadorIA.js` | Procesa respuesta de Gemini |
| `validarDatosIA.js` | Validación básica post-IA |
| `adaptarDatosTelegram.js` | Convierte formato Telegram → Web |
| `calcularDias.js` | Calcula días hasta el evento |
| `clasificarUrgencia.js` | Clasifica por urgencia |

---

**Última actualización:** 2025-12-09


