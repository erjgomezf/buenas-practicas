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

**Última actualización:** 2025-12-04
