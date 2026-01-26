# 📘 Buenas Prácticas para Desarrollo de Workflows en N8N

Documento vivo para acumular principios, patrones y referencias técnicas para el desarrollo de workflows profesionales en N8N.

---

## 19. Patrones Avanzados (Lecciones de Migración 2026)

### 19.1. Normalización de Datos con Nodos Dedicados

**Problema:** Expresiones complejas repetidas en múltiples nodos para extraer el mismo dato.

**Solución:** Crear un nodo Set dedicado al inicio del flujo para normalizar datos críticos.

**Ejemplo - Nodo `chatID`:**
```javascript
{
  "chat_id": {{ $('telegramTrigger').item?.json?.message?.from?.id || $('telegramTrigger').item?.json?.callback_query?.from?.id }}
}
```

**Beneficios:**
- ✅ DRY: Un solo lugar para la lógica de extracción
- ✅ Mantenibilidad: Cambios futuros solo requieren editar un nodo
- ✅ Legibilidad: Nodos posteriores usan `$('chatID').item.json.chat_id`
- ✅ Debugging: Fácil verificar el valor extraído

---

### 19.2. Configuración Correcta de HTTP Requests

**Problema Común:** Pegar código JavaScript en campos de parámetros individuales.

**❌ Incorrecto:**
```
Body Parameters:
  Name: // Los datos ya vienen preparados...
       const inputData = $input.first().json;
       ...
```

**✅ Correcto:**
```
Specify Body: Using JSON
JSON: {{ $json }}
```

**Regla de Oro:**
- Si el nodo anterior ya preparó el JSON completo → Usa `{{ $json }}`
- Si necesitas construir el JSON desde cero → Usa "Using JSON" con expresiones
- Nunca pegues código JavaScript en campos de parámetros individuales

---

### 19.3. Máquinas de Estado: Patrón Fast-Forward

**Problema:** Estado inicial que muestra opciones pero no procesa la respuesta del usuario.

**Escenario:**
```javascript
case STEPS.START:
  response.text = 'Elige una opción:';
  response.buttons = OPTIONS.TIPO_EVENTO;
  response.next_step = STEPS.FECHA;  // Avanza el paso
  break;

case STEPS.FECHA:
  // Espera que el usuario haya seleccionado el tipo de evento
  if (incomingCallback) {
    response.update_data.tipo_evento = incomingCallback;
    ...
  }
```

**Problema:** Si `paso_actual` es `START` y el usuario selecciona, el bot vuelve a mostrar el menú.

**Solución - Fast-Forward:**
```javascript
// ANTES del switch
if (currentStep === STEPS.START && (incomingCallback || incomingText)) {
  console.log('⏩ Transición rápida: START -> FECHA');
  currentStep = STEPS.FECHA;
}

switch (currentStep) {
  case STEPS.START:
    // Solo se ejecuta si NO hay input
    ...
  case STEPS.FECHA:
    // Ahora SÍ procesa el input del usuario
    ...
}
```

**Cuándo Usar:**
- Estados que muestran opciones pero no esperan respuesta inmediata
- Transiciones donde el "mostrar menú" y "procesar respuesta" están en pasos diferentes

---

### 19.4. Fallbacks Robustos para Datos Críticos

**Problema:** Un nodo falla porque un dato crítico (ej: `chat_id`) no está en el input esperado.

**Solución - Cascada de Fallbacks:**
```javascript
// 1. Intentar leer del input directo
let chatId = inputData.chat_id;

// 2. Fallback a nodo normalizado
if (!chatId) {
  try {
    const chatNodeData = $('chatID').first().json;
    chatId = chatNodeData.chat_id;
    console.log('⚠️ chat_id recuperado del nodo chatID (fallback)');
  } catch (e) {
    console.log('❌ Falló recuperación del chat_id');
  }
}

// 3. Validación final
if (!chatId) {
  throw new Error('❌ No se pudo obtener chat_id del flujo');
}
```

**Principios:**
- Siempre tener al menos 2 fuentes posibles para datos críticos
- Loggear cuándo se usa un fallback (ayuda en debugging)
- Fallar explícitamente si todos los fallbacks fallan

---

### 19.5. Evitar JSON.parse() en Datos de APIs

**Problema:** El API devuelve objetos JavaScript, pero el código intenta parsearlos como strings.

**Error Común:**
```javascript
const datosJson = sesionData.datos_json ? JSON.parse(sesionData.datos_json) : {};
// ❌ Error: "[object Object]" is not valid JSON
```

**Solución:**
```javascript
const datosJson = sesionData.datos_json || {};
// ✅ Si ya es un objeto, úsalo directamente
```

**Regla:**
- **Google Sheets / CSV:** Los datos vienen como strings → Usar `JSON.parse()`
- **APIs REST (FastAPI, Express, etc.):** Los datos vienen como objetos → NO parsear
- **Verificar siempre:** `console.log(typeof sesionData.datos_json)` antes de parsear

---

### 19.6. Debugging con Logs Estratégicos

**Patrón para Nodos Críticos:**
```javascript
// INPUT
console.log('🔍 INPUT nombreNodo:', JSON.stringify(inputData, null, 2));

// LÓGICA
const resultado = procesarDatos(inputData);

// OUTPUT
console.log('📤 OUTPUT nombreNodo:', JSON.stringify(resultado, null, 2));

return resultado;
```

**Beneficios:**
- Trazabilidad completa del flujo de datos
- Fácil identificar dónde se pierde/corrompe un valor
- Los emojis ayudan a escanear logs rápidamente

**Cuándo Usar:**
- Nodos que transforman datos complejos
- Nodos que fallan intermitentemente
- Durante desarrollo/debugging (remover en producción si afecta performance)

---

### 19.7. Validación de Configuración de Nodos

**Checklist Pre-Deployment:**

**HTTP Request Nodes:**
- [ ] Method correcto (GET/POST/PUT/DELETE)
- [ ] URL con expresiones válidas (probar con datos reales)
- [ ] Body configurado correctamente (Using JSON vs Fields)
- [ ] Headers necesarios (Content-Type, Authorization)
- [ ] Timeout apropiado para la operación

**Code Nodes:**
- [ ] Código actualizado desde archivos refactorizados
- [ ] Sin código comentado/debug innecesario
- [ ] Variables críticas validadas (throw Error si faltan)
- [ ] Return statement correcto (`return {...}` o `return [{json: {...}}]`)

**Set Nodes:**
- [ ] Expresiones probadas con datos reales
- [ ] Fallbacks para valores opcionales
- [ ] Nombres de campos consistentes con el resto del workflow

---

## Resumen de Lecciones Clave

1. **Normaliza Temprano:** Extrae datos complejos al inicio del flujo
2. **Configura Correctamente:** HTTP Requests con `{{ $json }}` cuando sea posible
3. **Piensa en Estados:** Usa Fast-Forward para transiciones complejas
4. **Siempre Fallback:** Datos críticos deben tener múltiples fuentes
5. **No Parsees Objetos:** Verifica el tipo antes de `JSON.parse()`
6. **Loggea Estratégicamente:** Input/Output de nodos críticos
7. **Valida Antes de Deployar:** Checklist de configuración

---

**Última Actualización**: 2026-01-26
