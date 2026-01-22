# 🗄️ Buenas Prácticas: SQLite3

## 📋 Índice

1. [Comandos Básicos](#comandos-básicos)
2. [Tablas del Proyecto](#tablas-del-proyecto)
3. [Queries Comunes](#queries-comunes)
4. [Buenas Prácticas](#buenas-prácticas)
5. [Integración con N8N](#integración-con-n8n)

---

## 🎯 Comandos Básicos

### Entrar y Salir de SQLite

```bash
# Entrar a la BD
sqlite3 data/livemoments.db

# Salir
.exit
# O presiona Ctrl+D
```

### Comandos de Configuración (Dentro de SQLite)

```sql
-- Formato de tabla bonito
.mode column

-- Mostrar nombres de columnas
.headers on

-- Ajustar ancho de columnas
.width 20 15 10

-- Ver todas las tablas
.tables

-- Ver estructura de una tabla
.schema paquetes

-- Ver TODO el schema
.schema

-- Ver información de la BD
.database
```

### Ejecutar SQL desde Terminal (Sin entrar a SQLite)

```bash
# Consulta directa
sqlite3 data/livemoments.db "SELECT COUNT(*) FROM paquetes;"

# Ver tablas
sqlite3 data/livemoments.db ".tables"

# Ejecutar archivo SQL
sqlite3 data/livemoments.db < script.sql
```

---

## 📊 Tablas del Proyecto

### 1. `paquetes` - Catálogo de Paquetes

**Propósito:** Almacena los paquetes de servicios (Básico, Premium, etc.)

**Schema:**
```sql
CREATE TABLE paquetes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL UNIQUE,
    descripcion TEXT,
    detalle TEXT,  -- JSON array: ["feature 1", "feature 2"]
    precio REAL NOT NULL DEFAULT 0,
    icono TEXT,
    activo BOOLEAN DEFAULT 1,
    ultima_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Índices:**
- `idx_paquetes_nombre` - Búsqueda por nombre
- `idx_paquetes_activo` - Filtrar activos

**Queries Comunes:**
```sql
-- Listar todos los paquetes activos
SELECT nombre, precio, icono FROM paquetes WHERE activo = 1;

-- Buscar paquete específico
SELECT * FROM paquetes WHERE nombre = 'Premium';

-- Ordenar por precio
SELECT nombre, precio FROM paquetes WHERE activo = 1 ORDER BY precio ASC;
```

**Datos de Ejemplo:**
```sql
nombre: "Premium"
descripcion: "🥈 Premium - 2 cámaras HD"
detalle: '["2 cámaras HD", "2 micrófonos", "Streaming a 2 plataformas"]'
precio: 250.0
icono: "🥈"
activo: 1
```

---

### 2. `addons` - Servicios Adicionales

**Propósito:** Almacena servicios adicionales que se pueden agregar a paquetes

**Schema:**
```sql
CREATE TABLE addons (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL UNIQUE,
    precio REAL NOT NULL DEFAULT 0,
    icono TEXT,
    activo BOOLEAN DEFAULT 1,
    ultima_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Índices:**
- `idx_addons_nombre` - Búsqueda por nombre
- `idx_addons_activo` - Filtrar activos

**Queries Comunes:**
```sql
-- Listar todos los addons activos
SELECT nombre, precio, icono FROM addons WHERE activo = 1;

-- Ordenar por precio (más caros primero)
SELECT nombre, precio FROM addons WHERE activo = 1 ORDER BY precio DESC;

-- Buscar addon por nombre parcial
SELECT * FROM addons WHERE nombre LIKE '%Cámara%';
```

**Datos de Ejemplo:**
```sql
nombre: "📹 Cámaras + Micrófonos adicionales"
precio: 30.0
icono: "📹"
activo: 1
```

---

### 3. `sesiones_telegram` - Estados de Conversación

**Propósito:** Almacena el estado actual de cada usuario del bot de Telegram

**Schema:**
```sql
CREATE TABLE sesiones_telegram (
    chat_id TEXT PRIMARY KEY,
    paso_actual TEXT NOT NULL DEFAULT 'start',
    datos_json TEXT,  -- JSON: {"tipo_evento": "Boda", "nombre": "Juan"}
    tipo_validacion TEXT DEFAULT 'BOT',  -- 'BOT' o 'IA'
    intentos_fallidos INTEGER DEFAULT 0,
    ultima_interaccion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Índices:**
- `idx_sesiones_paso` - Búsqueda por paso actual
- `idx_sesiones_tipo` - Filtrar por tipo de validación
- `idx_sesiones_ultima_interaccion` - Ordenar por fecha

**Queries Comunes:**
```sql
-- Buscar sesión de un usuario específico
SELECT * FROM sesiones_telegram WHERE chat_id = '123456789';

-- Crear nueva sesión
INSERT INTO sesiones_telegram (chat_id, paso_actual, datos_json)
VALUES ('123456789', 'start', '{}');

-- Actualizar sesión
UPDATE sesiones_telegram 
SET paso_actual = 'paquete',
    datos_json = '{"tipo_evento": "Boda"}',
    ultima_interaccion = CURRENT_TIMESTAMP
WHERE chat_id = '123456789';

-- Borrar sesión (cuando cancela)
DELETE FROM sesiones_telegram WHERE chat_id = '123456789';

-- Listar sesiones activas (últimas 24h)
SELECT chat_id, paso_actual, ultima_interaccion
FROM sesiones_telegram
WHERE datetime(ultima_interaccion) > datetime('now', '-1 day');
```

**Datos de Ejemplo:**
```sql
chat_id: "123456789"
paso_actual: "paquete"
datos_json: '{"tipo_evento": "Boda", "fecha": "2026-06-15", "ciudad": "Bogotá"}'
tipo_validacion: "BOT"
intentos_fallidos: 0
ultima_interaccion: "2026-01-19 14:30:00"
```

---

### 4. `sync_history` - Historial de Sincronizaciones

**Propósito:** Auditoría de sincronizaciones desde Google Sheets

**Schema:**
```sql
CREATE TABLE sync_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    tipo TEXT NOT NULL,  -- 'catalog' o 'sessions'
    origen TEXT NOT NULL,  -- 'manual', 'webhook', 'cron'
    estado TEXT NOT NULL,  -- 'success', 'error', 'partial'
    registros_procesados INTEGER DEFAULT 0,
    mensaje_error TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Índices:**
- `idx_sync_timestamp` - Ordenar por fecha
- `idx_sync_tipo` - Filtrar por tipo

**Queries Comunes:**
```sql
-- Ver últimas 10 sincronizaciones
SELECT tipo, origen, estado, registros_procesados, timestamp
FROM sync_history
ORDER BY timestamp DESC
LIMIT 10;

-- Ver solo errores
SELECT * FROM sync_history WHERE estado = 'error';

-- Contar sincronizaciones exitosas hoy
SELECT COUNT(*) FROM sync_history
WHERE estado = 'success'
AND date(timestamp) = date('now');
```

---

## 🎯 Queries Comunes por Caso de Uso

### Para N8N: Obtener Catálogo

```sql
-- Paquetes para mostrar en Telegram
SELECT nombre, descripcion, detalle, precio, icono
FROM paquetes
WHERE activo = 1
ORDER BY precio ASC;

-- Addons para checkboxes
SELECT nombre, precio, icono
FROM addons
WHERE activo = 1
ORDER BY nombre ASC;
```

### Para N8N: Gestión de Sesiones

```sql
-- Buscar sesión existente
SELECT paso_actual, datos_json, tipo_validacion, intentos_fallidos
FROM sesiones_telegram
WHERE chat_id = ?;

-- Crear sesión nueva (UPSERT)
INSERT INTO sesiones_telegram (chat_id, paso_actual, datos_json)
VALUES (?, 'start', '{}')
ON CONFLICT(chat_id) DO UPDATE SET
    paso_actual = 'start',
    datos_json = '{}',
    ultima_interaccion = CURRENT_TIMESTAMP;

-- Actualizar sesión
UPDATE sesiones_telegram
SET paso_actual = ?,
    datos_json = ?,
    ultima_interaccion = CURRENT_TIMESTAMP
WHERE chat_id = ?;
```

### Para Desarrollo: Debugging

```sql
-- Ver estadísticas generales
SELECT 
    (SELECT COUNT(*) FROM paquetes WHERE activo = 1) as paquetes_activos,
    (SELECT COUNT(*) FROM addons WHERE activo = 1) as addons_activos,
    (SELECT COUNT(*) FROM sesiones_telegram) as sesiones_totales;

-- Ver sesiones por paso
SELECT paso_actual, COUNT(*) as cantidad
FROM sesiones_telegram
GROUP BY paso_actual;

-- Sesiones con problemas
SELECT chat_id, paso_actual, intentos_fallidos
FROM sesiones_telegram
WHERE intentos_fallidos > 3;
```

---

## ✅ Buenas Prácticas

### 1. Siempre Usar Prepared Statements (Evitar SQL Injection)

**❌ MAL:**
```javascript
const chat_id = "123456";
db.run(`SELECT * FROM sesiones_telegram WHERE chat_id = '${chat_id}'`);
// Vulnerable a SQL injection!
```

**✅ BIEN:**
```javascript
const chat_id = "123456";
db.get("SELECT * FROM sesiones_telegram WHERE chat_id = ?", [chat_id]);
// Seguro con parámetros
```

### 2. Usar Transacciones para Múltiples Operaciones

```sql
BEGIN TRANSACTION;

UPDATE sesiones_telegram SET paso_actual = 'paquete' WHERE chat_id = '123';
INSERT INTO sync_history (tipo, estado) VALUES ('catalog', 'success');

COMMIT;
-- O ROLLBACK si hay error
```

### 3. Índices para Columnas de Búsqueda Frecuente

Ya creados en el schema:
- ✅ `chat_id` (PRIMARY KEY automático)
- ✅ `nombre` en paquetes y addons
- ✅ `activo` para filtros
- ✅ `ultima_interaccion` para ordenamiento

### 4. Soft Delete (No Borrar, Desactivar)

```sql
-- En lugar de DELETE
UPDATE paquetes SET activo = 0 WHERE nombre = 'Antiguo';

-- Luego filtrar con WHERE activo = 1
```

### 5. Timestamps Automáticos con Triggers

Ya implementado en el schema:
- ✅ `ultima_actualizacion` se actualiza automáticamente
- ✅ `creado_en` se setea al insertar

### 6. Validación de Datos JSON

```javascript
// Antes de guardar
try {
    const datos = JSON.parse(datos_json);
    // Validar estructura
    if (!datos.tipo_evento) throw new Error("Falta tipo_evento");
} catch (e) {
    console.error("Error validando JSON:", e);
}
```

---

## 🔌 Integración con N8N

### Opción 1: Nodo SQLite Nativo

```javascript
// N8N tiene soporte nativo para SQLite
// Configuración:
// - Database Path: /home/programar/Documentos/N8N/data/livemoments.db
// - Operation: executeQuery
// - Query: SELECT * FROM paquetes WHERE activo = 1
```

### Opción 2: Nodo Code (JavaScript)

```javascript
const sqlite3 = require('sqlite3').verbose();
const db = new sqlite3.Database('/home/programar/Documentos/N8N/data/livemoments.db');

// SELECT
return new Promise((resolve, reject) => {
    db.all('SELECT * FROM paquetes WHERE activo = 1', [], (err, rows) => {
        if (err) reject(err);
        else resolve(rows);
    });
});

// INSERT/UPDATE
return new Promise((resolve, reject) => {
    const chat_id = $input.item.json.chat_id;
    const paso = $input.item.json.paso;
    const datos = JSON.stringify($input.item.json.datos);
    
    db.run(
        'UPDATE sesiones_telegram SET paso_actual = ?, datos_json = ? WHERE chat_id = ?',
        [paso, datos, chat_id],
        (err) => {
            if (err) reject(err);
            else resolve({ success: true });
        }
    );
});
```

### Queries para Reemplazar Google Sheets Nodes

**Nodo `obtenerPaquetes`:**
```sql
SELECT nombre, descripcion, detalle, precio, icono
FROM paquetes
WHERE activo = 1
ORDER BY precio ASC;
```

**Nodo `obtenerAddons`:**
```sql
SELECT nombre, precio, icono
FROM addons
WHERE activo = 1
ORDER BY nombre ASC;
```

**Nodo `buscarSesion`:**
```sql
SELECT paso_actual, datos_json, tipo_validacion, intentos_fallidos
FROM sesiones_telegram
WHERE chat_id = ?;
```

**Nodo `crearSesion`:**
```sql
INSERT INTO sesiones_telegram (chat_id, paso_actual, datos_json)
VALUES (?, 'start', '{}')
ON CONFLICT(chat_id) DO UPDATE SET
    paso_actual = 'start',
    ultima_interaccion = CURRENT_TIMESTAMP;
```

**Nodo `actualizarSesion`:**
```sql
UPDATE sesiones_telegram
SET paso_actual = ?,
    datos_json = ?,
    tipo_validacion = ?,
    intentos_fallidos = ?,
    ultima_interaccion = CURRENT_TIMESTAMP
WHERE chat_id = ?;
```

---

## 🛠️ Mantenimiento

### Backup

```bash
# Backup simple (copiar archivo)
cp data/livemoments.db data/livemoments_backup_$(date +%Y%m%d).db

# Backup SQL dump
sqlite3 data/livemoments.db .dump > backup_$(date +%Y%m%d).sql
```

### Restaurar

```bash
# Desde archivo
cp data/livemoments_backup_20260119.db data/livemoments.db

# Desde dump SQL
sqlite3 data/livemoments_restored.db < backup_20260119.sql
```

### Vacuum (Optimizar espacio)

```sql
-- Liberar espacio de registros borrados
VACUUM;
```

### Verificar Integridad

```sql
PRAGMA integrity_check;
-- Resultado esperado: ok
```

---

## 📚 Recursos

- [SQLite Official Docs](https://www.sqlite.org/docs.html)
- [SQL Tutorial](https://www.w3schools.com/sql/)
- Schema del proyecto: `database/schema.sql`
- Scripts Python: `scripts/sync-catalog.py`, `scripts/clean-sessions.py`

---

## 🎯 Próximos Pasos para N8N

1. **Modificar `obtenerPaquetes`**: Cambiar Google Sheets node → SQLite query
2. **Modificar `obtenerAddons`**: Cambiar Google Sheets node → SQLite query
3. **Modificar `buscarSesion`**: Cambiar Google Sheets node → SQLite query
4. **Modificar `actualizarSesion`**: Cambiar Google Sheets node → SQLite query
5. **Probar flujo completo** en Telegram

---

**Recuerda:** SQLite es **más rápido** (~5ms) que Google Sheets (~200ms) y **sin límites de API**. 🚀
