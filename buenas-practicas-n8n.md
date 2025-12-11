# 📘 Buenas Prácticas para Desarrollo de Workflows en N8N

Documento vivo para acumular principios, patrones y referencias técnicas para el desarrollo de workflows profesionales en N8N.

---

## 1. Principios Fundamentales de Diseño de Workflows

### 1.1. Claridad y Legibilidad

- **Nomenclatura Descriptiva:** Cada nodo debe tener un nombre que describa claramente su función. Evita nombres genéricos como "HTTP Request 1" o "Set".
  - ❌ Mal: `HTTP Request`, `Set`, `IF`
  - ✅ Bien: `Obtener Datos del Cliente`, `Preparar Payload para API`, `Validar Email`

- **Organización Visual:** Organiza los nodos de forma lógica, de izquierda a derecha siguiendo el flujo de datos. Usa espaciado consistente.

- **Agrupación Lógica:** Agrupa nodos relacionados visualmente. N8N permite mover múltiples nodos juntos para mantener la coherencia.

### 1.2. Principio de Responsabilidad Única (SRP)

- **Un Nodo, Una Tarea:** Cada nodo debe realizar una sola operación bien definida. Si un nodo "Set" está haciendo demasiadas transformaciones, considera dividirlo.
  
- **Workflows Modulares:** Para workflows complejos, considera dividirlos en sub-workflows reutilizables usando el nodo "Execute Workflow".

### 1.3. Manejo de Errores Robusto

- **Siempre Configura Error Workflows:** Para nodos críticos (APIs externas, operaciones de BD, envío de emails), configura un flujo alternativo en caso de error.
  
- **Logging de Errores:** Registra los errores en un sistema centralizado (Google Sheets, base de datos, servicio de logging) para poder auditarlos.

- **Notificaciones de Fallos:** Configura alertas (email, Slack, Discord) para errores críticos que requieran intervención manual.

---

## 2. Organización y Nomenclatura

### 2.1. Convenciones de Nombres

| Tipo de Nodo | Convención | Ejemplo |
|--------------|------------|---------|
| Webhook | Verbo + Contexto | `Recibir Solicitud de Contacto` |
| HTTP Request | Acción + Destino | `Obtener Usuario de API`, `Enviar a Slack` |
| Set/Code | Acción + Dato | `Preparar Datos de Email`, `Calcular Total` |
| IF/Switch | Condición Clara | `¿Email Válido?`, `Clasificar por Tipo` |
| Database | Operación + Tabla | `Insertar en Usuarios`, `Consultar Pedidos` |
| Email/Notificación | Acción + Destinatario | `Notificar a Admin`, `Enviar Confirmación` |

### 2.2. Estructura de Workflows

- **Inicio Claro:** El punto de entrada debe ser obvio (Webhook, Trigger, Schedule).
- **Validación Temprana:** Valida los datos de entrada lo antes posible en el flujo.
- **Transformación Centralizada:** Agrupa las transformaciones de datos en nodos "Set" bien nombrados.
- **Salidas Múltiples:** Si el workflow tiene múltiples finales (éxito, error, diferentes tipos), nómbralos claramente.

---

## 3. Validación y Calidad de Datos

### 3.1. Validación de Entrada

- **Validación en Múltiples Capas:**
  1. **Cliente (Frontend):** Validación básica de formato
  2. **Webhook/Trigger:** Validación de estructura y tipos
  3. **Lógica de Negocio:** Validación de reglas de negocio

- **Uso del Nodo IF para Validaciones:**
  ```
  Webhook → IF (¿Datos Válidos?) 
    → True: Continuar Procesamiento
    → False: Registrar Error + Notificar
  ```

### 3.2. Sanitización de Datos

- **Limpieza de Inputs:** Usa nodos "Code" o "Set" para limpiar datos antes de usarlos:
  - Trim de espacios en blanco
  - Normalización de formatos (teléfonos, emails)
  - Escape de caracteres especiales

---

## 4. Seguridad y Credenciales

### 4.1. Manejo de Credenciales

- **Nunca Hardcodear Secretos:** Usa el sistema de credenciales de N8N para API keys, tokens, contraseñas.
  
- **Credenciales por Entorno:** Si tienes múltiples entornos (dev, staging, prod), usa credenciales diferentes para cada uno.

- **Rotación de Credenciales:** Documenta cuándo y cómo rotar las credenciales de servicios externos.

### 4.2. Seguridad de Webhooks

- **Autenticación de Webhooks:** 
  - Usa "Header Auth" o "Query Auth" para webhooks públicos
  - Valida tokens o firmas (ej. HMAC) para webhooks de terceros

- **Rate Limiting:** Considera implementar límites de tasa para evitar abuso de webhooks públicos.

- **Validación de Origen:** Verifica la IP o headers de origen cuando sea posible.

---

## 5. Integración con APIs Externas

### 5.1. Buenas Prácticas de HTTP Requests

- **Manejo de Timeouts:** Configura timeouts apropiados para evitar que el workflow se cuelgue.

- **Reintentos Inteligentes:** Para APIs inestables, configura reintentos con backoff exponencial.

- **Códigos de Estado:** Maneja explícitamente los diferentes códigos HTTP:
  - 200-299: Éxito
  - 400-499: Error del cliente (validar datos)
  - 500-599: Error del servidor (reintentar o alertar)

### 5.2. Rate Limiting de APIs

- **Respetar Límites:** Si una API tiene límites de tasa, usa el nodo "Wait" o "Split In Batches" para espaciar las peticiones.

- **Caché de Respuestas:** Para datos que no cambian frecuentemente, considera cachear respuestas.

---

## 6. Uso de IA en Workflows

### 6.1. Integración con LLMs (Google Gemini, OpenAI, etc.)

- **Prompts Estructurados:** Usa prompts claros y específicos. Define el formato de salida esperado (JSON, texto plano, etc.).

- **Validación de Respuestas:** Siempre valida la respuesta de la IA antes de usarla en lógica crítica.

- **Fallbacks:** Ten un plan B si la IA falla o devuelve resultados inesperados.

### 6.2. Casos de Uso Comunes

- **Clasificación de Mensajes:** Categorizar emails, tickets de soporte, formularios.
- **Análisis de Sentimiento:** Detectar tono positivo/negativo en feedback de clientes.
- **Generación de Contenido:** Crear respuestas personalizadas, resúmenes, traducciones.
- **Extracción de Datos:** Parsear texto no estructurado para extraer información clave.

---

## 7. Almacenamiento y Persistencia

### 7.1. Google Sheets como Base de Datos

- **Estructura Clara:** Define headers claros en la primera fila. Usa nombres descriptivos.

- **Validación de Datos:** Antes de insertar en Sheets, valida que los datos cumplan con el esquema esperado.

- **Manejo de Duplicados:** Implementa lógica para evitar duplicados (buscar antes de insertar).

- **Limitaciones:** Google Sheets no es una base de datos real. Para volúmenes altos o consultas complejas, considera una BD real (PostgreSQL, MySQL, MongoDB).

### 7.2. Bases de Datos Reales

- **Transacciones:** Para operaciones críticas, usa transacciones para asegurar consistencia.

- **Índices:** Crea índices en columnas que se consultan frecuentemente.

- **Migraciones:** Documenta cambios en el esquema de la base de datos.

---

## 8. Testing y Debugging

### 8.1. Estrategias de Testing

- **Modo Manual:** Usa el botón "Execute Workflow" para probar con datos de ejemplo.

- **Datos de Prueba:** Crea un conjunto de datos de prueba que cubra casos normales y casos borde.

- **Logs Estructurados:** Usa nodos "Set" para crear logs intermedios que te ayuden a debuggear.

### 8.2. Debugging

- **Inspección de Datos:** Usa la vista de "Output" de cada nodo para ver exactamente qué datos están fluyendo.

- **Nodos de Debug:** Inserta nodos temporales (ej. "Set" que solo loguea) para inspeccionar el estado en puntos específicos.

- **Ejecuciones Pasadas:** Revisa el historial de ejecuciones para identificar patrones de error.

---

## 9. Documentación y Versionado

### 9.1. Documentación del Workflow

- **Notas en Nodos:** Usa el campo "Notes" de N8N para documentar lógica compleja o decisiones de diseño.

- **README del Workflow:** Mantén un documento externo (Markdown, Google Docs) que explique:
  - Propósito del workflow
  - Requisitos y dependencias
  - Variables de configuración
  - Casos de uso y ejemplos

### 9.2. Versionado

- **Exportar JSON Regularmente:** Exporta el workflow a JSON y guárdalo en control de versiones (Git).

- **Commits Descriptivos:** Usa mensajes de commit claros que expliquen qué cambió y por qué.

- **Branches para Cambios Grandes:** Si vas a hacer cambios significativos, considera duplicar el workflow o usar branches en Git.

---

## 10. Optimización y Performance

### 10.1. Eficiencia de Workflows

- **Evitar Loops Innecesarios:** Si puedes procesar datos en batch, hazlo en lugar de iterar uno por uno.

- **Procesamiento Paralelo:** Para tareas independientes, considera usar "Split In Batches" con procesamiento paralelo.

- **Caché de Datos:** Si consultas los mismos datos múltiples veces, guárdalos en una variable temporal.

### 10.2. Monitoreo

- **Tiempo de Ejecución:** Monitorea cuánto tiempo toma cada ejecución. Si aumenta, investiga.

- **Tasa de Errores:** Lleva un registro de la tasa de errores del workflow.

- **Alertas de Performance:** Configura alertas si un workflow tarda más de lo esperado.

---

## 11. Patrones Comunes

### 11.1. Patrón de Clasificación con IA

```
Webhook → Validar Datos → Clasificar con IA → Switch (por categoría)
  → Categoría A: Flujo A
  → Categoría B: Flujo B
  → Error: Registrar + Notificar
```

### 11.2. Patrón de Enriquecimiento de Datos

```
Trigger → Obtener Dato Base → Consultar API Externa → Combinar Datos → Guardar
```

### 11.3. Patrón de Notificación Multi-Canal

```
Evento → Preparar Mensaje → Split
  → Email
  → Slack
  → SMS
  → Webhook
```

---

## 12. Fallbacks y Resiliencia

### 12.1. Patrón Circuit Breaker para Servicios Externos

Cuando integres servicios externos (APIs de IA, servicios de pago, etc.), siempre implementa un fallback:

```
[Servicio Externo] (Continue On Fail) → [¿Exitoso?]
  → True: Usar respuesta del servicio
  → False: Usar fallback/alternativa
```

**Ejemplo práctico:**
```
[AI Agent] → [¿IA Exitosa?]
  → True: Email personalizado
  → False: Email genérico (template)
```

### 12.2. Configuración de "Continue On Fail"

Para nodos que llaman a servicios externos:

1. **Activa "Continue On Fail"** en Settings del nodo
2. **Valida la respuesta** con un nodo IF o Code
3. **Implementa fallback** para el camino de error

**Validación de respuesta de IA:**
```javascript
const iaExitosa = $json.output && 
                  $json.output.length > 10 && 
                  !$json.error;

return {
  ...input,
  ia_exitosa: iaExitosa,
  usar_fallback: !iaExitosa
};
```

### 12.3. Templates Genéricos como Fallback

Siempre ten un template genérico pero profesional como fallback:

- **Incluye datos esenciales** del formulario
- **Mantén el branding** consistente
- **Proporciona valor** al usuario (confirmación, próximos pasos)
- **No menciones** que es un fallback

### 12.4. Monitoreo de Fallbacks

Registra cuándo se usa el fallback para detectar problemas:

```javascript
return {
  ...input,
  tipo_email: "generico_fallback",
  timestamp_fallback: new Date().toISOString()
};
```

Luego puedes:
- Contar frecuencia de fallbacks en Google Sheets
- Alertar si tasa de fallback > 10%
- Investigar problemas con el servicio externo

---

## 13. Antipatrones a Evitar

### 13.1. Antipatrones Comunes

- ❌ **Nodos Sin Nombre:** Dificulta el debugging y mantenimiento.
- ❌ **Workflows Monolíticos:** Un solo workflow que hace demasiadas cosas.
- ❌ **Sin Manejo de Errores:** Asumir que todo siempre funcionará.
- ❌ **Hardcodear Valores:** URLs, IDs, credenciales en nodos en lugar de usar variables o credenciales.
- ❌ **Validación Insuficiente:** Confiar ciegamente en datos de entrada.
- ❌ **Logs Excesivos:** Loguear datos sensibles o demasiada información.
- ❌ **Sin Fallbacks para IA:** Depender 100% de servicios de IA sin alternativa.
- ❌ **Validación Única:** Validar solo en frontend o solo en backend.

### 13.2. Lecciones Aprendidas (Proyecto Live Moments)

#### Importancia de Fallbacks
- **Problema:** APIs de IA pueden fallar o tener latencia alta
- **Solución:** Implementar fallback con template genérico
- **Resultado:** 100% de emails enviados, incluso si IA falla

#### Validación en Múltiples Capas
- **Problema:** Datos inválidos llegando al workflow
- **Solución:** Validar en frontend + backend + lógica de negocio
- **Resultado:** Reducción de 90% en errores de procesamiento

#### Nomenclatura Descriptiva
- **Problema:** Difícil identificar qué nodo falló en logs
- **Solución:** Nombres descriptivos como `¿IA Exitosa?` en lugar de `IF`
- **Resultado:** Debugging 3x más rápido

#### Separación de Lógica
- **Problema:** Nodos Code haciendo demasiadas cosas
- **Solución:** Un nodo por transformación (calcular, clasificar, validar)
- **Resultado:** Código más mantenible y reutilizable

---

## 14. Patrones Avanzados de N8N

### 14.1. Nodos con Múltiples Rutas de Entrada

Cuando un nodo recibe datos de **dos o más rutas diferentes**, se debe usar `$if()` con `isExecuted` para evitar errores:

#### ❌ Problema Común
```javascript
// Error: "Node 'ValidadorIA' hasn't been executed"
={{ $('ValidadorIA').item.json.data || $('switchAccion').item.json.data }}
```

#### ✅ Solución Correcta
```javascript
={{ $if($('ValidadorIA').isExecuted, $('ValidadorIA').item.json.data, $('switchAccion').item.json.data) }}
```

**Sintaxis:**
```javascript
$if(condición, valor_si_true, valor_si_false)
```

**Casos de uso comunes:**
- Nodo que recibe de `switchAccion` (flujo normal) O `switchValidacionIA` (flujo IA)
- Nodo Merge que combina resultados de diferentes branches
- Actualización de datos que puede venir de múltiples fuentes

### 14.2. Arquitectura de Validación con IA (Dos Capas)

Para minimizar costos de LLM y mejorar la UX, implementa validación en dos capas:

```
┌─────────────────────────────────────────────────────────┐
│                    CAPA 1: BOT                          │
│  (telegramLogic.js - Sin costo, respuesta inmediata)    │
├─────────────────────────────────────────────────────────┤
│  • Validación de formato (regex)                        │
│  • Validación de tipos (números, emails)                │
│  • Detección de comandos maliciosos (/ + & %)           │
│  • Fallback con intentos (2 intentos antes de aceptar)  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                    CAPA 2: IA                           │
│  (geminiValidador.js - Solo cuando todo está completo)  │
├─────────────────────────────────────────────────────────┤
│  • Validación semántica (¿es una ciudad real?)          │
│  • Coherencia de datos (¿fecha razonable?)              │
│  • Preguntas específicas para campos faltantes          │
│  • Límite de intentos antes de escalar a humano         │
└─────────────────────────────────────────────────────────┘
```

**Beneficios:**
- 80% de errores capturados en Capa 1 (sin costo LLM)
- IA solo se usa para validación final
- Respuesta más rápida para errores obvios

### 14.3. Nodos Code Pre y Post IA

#### Nodo Pre-IA (`prepararDatosIA`)
Prepara el contexto para el LLM:

```javascript
// Parsear datos de sesión
const datos = JSON.parse(session.datos_json || '{}');

// Obtener campo pendiente
const campoPendiente = datos._campo_pendiente;

// Actualizar con respuesta del usuario
if (campoPendiente && userInput) {
  datos[campoPendiente] = userInput;
}

return {
  update_data: datos,
  contexto_validacion: JSON.stringify(datos)
};
```

#### Nodo Post-IA (`geminiValidador`)
Procesa la respuesta y decide acción:

```javascript
// Parsear respuesta (puede ser string o JSON)
const validacion = typeof respuesta === 'string'
  ? JSON.parse(respuesta.replace(/```json|```/g, '')
  : respuesta;

// Determinar acción
if (validacion.valido) {
  return { action: 'send_to_central', ... };
} else if (validacion.campo_faltante) {
  return { 
    action: 'ask_field',
    update_data: { _campo_pendiente: validacion.campo_faltante, ... }
  };
} else {
  return { action: 'send_to_error_support', ... };
}
```

### 14.4. Bypass de Nodos con `tipoValidacion`

Usa un campo de control para rutear el flujo:

```
telegramTrigger → buscarSesion → esValidacionIA?
  → TRUE (tipoValidacion === 'IA'): prepararDatosIA → AI Agent
  → FALSE: logicaBot → switchAccion
```

**En Google Sheets, guarda:**
```
| chat_id | tipoValidacion | _campo_pendiente |
|---------|----------------|------------------|
| 12345   | IA             | ubicacion_evento |
```

### 14.5. Sanitización Pre-Validación

Antes de enviar a la IA, sanitiza campos sospechosos:

```javascript
const caracteresInvalidos = /[\.\.\/\+\&\%\@\#\$\!\?\*\<\>\|]/;

for (const campo of ['ubicacion_evento', 'nombre_cliente']) {
  if (datos[campo] && caracteresInvalidos.test(datos[campo])) {
    datos[`_original_${campo}`] = datos[campo]; // Debug
    datos[campo] = null; // IA lo detectará como faltante
  }
}
```

---

## Apéndice A: Checklist Pre-Deployment

Antes de activar un workflow en producción, verifica:

- [ ] Todos los nodos tienen nombres descriptivos
- [ ] Manejo de errores configurado en nodos críticos
- [ ] Credenciales configuradas correctamente (no hardcodeadas)
- [ ] Validación de datos de entrada implementada
- [ ] Logs y monitoreo configurados
- [ ] Workflow probado con datos reales y casos borde
- [ ] Documentación actualizada
- [ ] Workflow exportado y versionado en Git
- [ ] Alertas de error configuradas
- [ ] Rate limiting considerado (si aplica)

---

## Apéndice B: Recursos y Referencias

### Documentación Oficial
- [N8N Documentation](https://docs.n8n.io/)
- [N8N Community Forum](https://community.n8n.io/)
- [N8N Workflow Templates](https://n8n.io/workflows/)

### Integraciones Comunes
- Google Workspace (Sheets, Gmail, Drive)
- Slack, Discord, Telegram
- OpenAI, Google Gemini
- Bases de datos (PostgreSQL, MySQL, MongoDB)
- CRMs (HubSpot, Salesforce)

- Webhooks (Stripe, GitHub, etc.)

---

**Última Actualización**: 2025-12-09
