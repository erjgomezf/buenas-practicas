# 🐳 Buenas Prácticas y Comandos Docker

Este documento recopila los comandos esenciales y buenas prácticas para gestionar el entorno de N8N y el API de Sesiones con Docker.

---

## 🚀 Comandos Esenciales (Ciclo de Vida)

### Iniciar Servicios
Levantar todo en segundo plano (detached mode).
```bash
docker-compose up -d
```

### Detener Servicios
Detener y eliminar contenedores (mantiene volúmenes/datos).
```bash
docker-compose down
```

### Reiniciar un Servicio Específico
Útil cuando el código de Python cambia (aunque tenemos reload activado, a veces es necesario).
```bash
docker-compose restart sessions-api
```

### Ver Logs (Tiempo Real)
Ver logs de todos los servicios.
```bash
docker-compose logs -f
```
Ver logs solo del API (útil para debuggear endpoints).
```bash
docker-compose logs -f sessions-api
```

---

## 🛠️ Desarrollo y Builds

### Reconstruir Imagen (IMPORTANTE)
Cuando modificas `requirements.txt` o la estructura de carpetas en `api/`, reiniciar no es suficiente. Debes reconstruir la imagen.
```bash
docker-compose up -d --build sessions-api
```
*Tip: `--build` fuerza a Docker a leer el Dockerfile y copiar los archivos de nuevo.*

### Limpiar Contenedor y Reconstruir
Si tienes problemas extraños (archivos viejos persistentes), detén el contenedor, bórralo y reconstrúyelo.
```bash
docker-compose stop sessions-api
docker-compose rm -f sessions-api
docker-compose up -d --build sessions-api
```

### Entrar al Contenedor (Shell)
Para inspeccionar archivos internos o ejecutar scripts manualmente (ej. `sqlite3`).
```bash
docker exec -it sessions_api /bin/bash
```

---

## 💾 Gestión de Datos (Volúmenes)

### Ubicación de Datos
- **N8N:** Los workflows y credenciales se guardan en el volumen de Docker `n8n_data`.
- **API (SQLite):** La base de datos está mapeada desde tu carpeta local `./data` hacia `/app/data` en el contenedor.

### Verificar Base de Datos desde el Host
Como está mapeada, puedes usar `sqlite3` directamente en tu máquina:
```bash
sqlite3 data/livemoments.db "SELECT * FROM sesiones_telegram;"
```

### Verificar Base de Datos desde el Contenedor
Si no tienes sqlite3 en tu host:
```bash
docker exec sessions_api sqlite3 /app/data/livemoments.db "SELECT count(*) FROM sesiones_telegram;"
```

---

## ⚠️ Troubleshooting Común

### 1. "Endpoint not found" o cambios no reflejados
**Causa:** Docker usa una versión cacheada del código.
**Solución:** Reconstruir la imagen.
```bash
docker-compose up -d --build sessions-api
```

### 2. "Port already in use"
**Causa:** Otro proceso está usando el puerto 8000 o 5678.
**Solución:** Buscar y matar el proceso, o cambiar el puerto en `docker-compose.yml`.
```bash
sudo lsof -i :8000
```

### 3. Error de Permisos en SQLite
**Causa:** El usuario dentro del contenedor no tiene permisos para escribir en `./data`.
**Solución:** Dar permisos de escritura a la carpeta de datos.
```bash
chmod -R 777 data/
```

### 4. N8N no conecta con el API
**Causa:** Estás usando `localhost` dentro de N8N.
**Solución:** Dentro de Docker, `localhost` es el propio contenedor. Para hablar con otro servicio, usa el nombre del servicio en `docker-compose.yml`.
- ❌ `http://localhost:8000` (Incorrecto desde N8N)
- ✅ `http://sessions-api:8000` (Correcto)

---

## 🛡️ Buenas Prácticas de Seguridad

1. **No exponer puertos innecesarios:** Solo exponer lo que necesitas acceder desde el host.
2. **Variables de Entorno:** Usar `.env` para credenciales (tokens, passwords). Nunca hardcodear en `docker-compose.yml`.
3. **Healthchecks:** Implementar healthchecks (como el que tiene `sessions-api`) para que Docker sepa si el servicio está vivo.
