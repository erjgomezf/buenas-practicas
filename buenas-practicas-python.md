# 📘 Buenas Prácticas y Guía de Estudio de Python para Backend

Documento vivo para acumular principios, patrones, comandos y referencias técnicas para el desarrollo backend en el ecosistema Python, con foco en FastAPI, Django y Django REST Framework (DRF).

---

## 1. Filosofía y Principios de Backend (Python)

Nuestra base es el código limpio y el diseño sólido. Estos son nuestros pilares innegociables.

### 1.1. Principios SOLID en Python

- **S - Responsabilidad Única (SRP):** Cada componente (clase, función) tiene una sola razón para cambiar.
- **O - Abierto/Cerrado (OCP):** Abiertos a la extensión (usando herencia, composición o inyección de dependencias), pero cerrados a la modificación del código existente.
- **L - Sustitución de Liskov (LSP):** Las subclases deben ser sustituibles por sus superclases sin alterar la lógica. En Python, esto se logra respetando los contratos definidos (ej. signaturas de métodos).
- **I - Segregación de Interfaces (ISP):** Interfaces pequeñas y específicas. Usa `typing.Protocol` o `abc.ABC` para definir contratos que los clientes puedan implementar sin acoplarse a métodos que no necesitan.
- **D - Inversión de Dependencias (DIP):** Los módulos de alto nivel dependen de abstracciones (`Protocol`), no de módulos de bajo nivel (clases concretas).

### 1.2. Herramientas y Patrones Clave en Python

- **Inyección de Dependencias (DI):** Aplicación práctica del DIP para lograr un código desacoplado, flexible y testeable. En FastAPI se logra con el sistema `Depends`.
- **Abstracciones con `typing.Protocol` y `abc.ABC`:** Para definir contratos claros y aplicar DIP y LSP de forma pythónica. `Protocol` es preferido por ser más flexible (duck typing estático).
- **Modelado y Validación con Pydantic:** Para crear modelos de datos seguros, auto-documentados y con validación en tiempo de ejecución. Es el corazón de FastAPI.
- **Estilo y Legibilidad:** Adhesión estricta a **PEP 8** y uso intensivo de **Tipado Estático (`type hints`)** para claridad y detección temprana de errores.

---

## 2. Calidad, Estilo y Documentación

### 2.1. Formateo y Linting

- **Black:** Adopta `black` como formateador de código no negociable. Ejecuta `black .` antes de cada commit para asegurar un estilo uniforme.
- **Linter (Ruff, Flake8):** Utiliza un linter para detectar errores de lógica, código no utilizado y violaciones de buenas prácticas que el formateador no corrige.

### 2.2. Comentarios y Docstrings Estructurados

- **Propósito:** Escribir docstrings que no solo sean legibles para humanos, sino también interpretables por herramientas automáticas (como asistentes de código o generadores de documentación).
- **Formato recomendado (estilo reStructuredText/Sphinx simplificado):**

  - Usar comillas triples `"""Docstring aquí"""`.
  - Una descripción breve en la primera línea.
  - Usar `*` para definir secciones claras como `* Atributos:`, `* Métodos:`, `* Parámetros:`.
  - Usar `-` para listar los elementos dentro de cada sección.

- **Ejemplo práctico:**

  ```python
  class MiVista(APIView):
      """Vista para gestionar los perfiles de usuario."""
  ```

### 2.3. Convenciones de Naming en Python

| Tipo        | Convención                                  | Ejemplo                       |
| ----------- | ------------------------------------------- | ----------------------------- |
| Modelos     | Singular PascalCase                         | Curso, UsuarioPerfil          |
| Serializers | PascalCase + Sufijo Serializer              | CursoSerializer               |
| ViewSets    | PascalCase + Sufijo ViewSet                 | CursoViewSet                  |
| Campos bool | prefijo `es_` / `tiene_` / adjetivo directo | `publicado`, `es_activo`      |
| Rutas (URL) | plural-kebab-case                           | `cursos`, `cursos-publicados` |
| Funciones   | snake_case descriptivo                      | `generar_token_reset`         |
| Variables   | snake_case                                  | `nombre_de_usuario`           |
| Constantes  | UPPER_SNAKE_CASE                            | `TASA_DE_INTERES`             |
| Tests       | prefijo `test_` + verbo/condición           | `test_crea_curso_ok`          |

---

## 3. Estructura de Proyectos

### 3.1. Estructura de Proyectos con FastAPI

Para mantener un proyecto FastAPI organizado, escalable y fácil de mantener, se recomienda una estructura modular que separe las responsabilidades. El uso de `APIRouter` es clave para dividir la lógica de negocio en componentes más pequeños.

**Árbol de Archivos Recomendado:**

```txt
/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── dependencies.py
│   └── routers/
│       ├── __init__.py
│       ├── customer.py
│       ├── invoices.py
│       └── transactions.py
│   └── test/
│       ├── __init__.py
│       └── test_customer.py
├── models.py
├── db.py
└── requirements.txt
```

**Descripción de Componentes:**

- **`app/main.py`**: Punto de entrada de la aplicación. Aquí se crea la instancia principal de `FastAPI` y se incluyen los routers.

  ```python
  # app/main.py
  from fastapi import FastAPI
  from .routers import customer, invoices

  app = FastAPI()

  app.include_router(customer.router)
  app.include_router(invoices.router)

  @app.get("/")
  def read_root():
      return {"Hello": "World"}
  ```

- **`app/dependencies.py`**: Define dependencias comunes que se pueden inyectar (`Depends`), como la sesión de la base de datos.
- **`app/routers/`**: Directorio que agrupa los `APIRouter`. Cada archivo es un conjunto de endpoints para una entidad de negocio.

  ```python
  # app/routers/customer.py
  from fastapi import APIRouter

  router = APIRouter(
      prefix="/customers",
      tags=["customers"],
  )

  @router.get("/")
  def read_customers():
      return [{"customer_id": 1, "name": "John Doe"}]
  ```

- **`models.py`**: Define los modelos de datos (Pydantic, SQLModel, SQLAlchemy).
- **`db.py`**: Contiene la configuración y lógica para la conexión a la base de datos.

### 3.2. Estructura de Proyectos con Django y DRF

La estructura de un proyecto Django está diseñada para mantener una clara separación de responsabilidades.

**Árbol de Archivos Recomendado:**

```txt
/
├── manage.py
├── requirements.txt
├── project_name/         # Directorio de configuración del proyecto
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── app_name/               # App de Django para una funcionalidad específica
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── serializers.py
    ├── permissions.py
    ├── tests.py
    ├── urls.py
    └── views.py
```

**Descripción de Componentes:**

- **`project_name/`**: Contiene la configuración del proyecto (`settings.py`) y el enrutador principal (`urls.py`).
- **`app_name/`**: Una aplicación autocontenida para una funcionalidad (ej. `usuarios`, `productos`).
  - **`models.py`**: Define los modelos del ORM de Django.
  - **`serializers.py`**: Define los serializadores de DRF para convertir modelos a JSON. **Aquí debe vivir la mayor parte de la lógica de validación.**
  - **`views.py`**: Contiene la lógica de las vistas (APIViews, ViewSets) que manejan las peticiones HTTP.
  - **`urls.py`**: Define las rutas específicas de la aplicación.
  - **`permissions.py`**: Para lógicas de autorización complejas o reutilizables.

---

## 4. Django y Django REST Framework (DRF)

### 4.1. Estrategia de Validación y Permisos

- **Validaciones en Serializers:** Centralizar todas las validaciones de datos (nivel de campo y nivel de objeto) en los `serializers.py`. Esto asegura que ninguna data inválida llegue a los modelos o la lógica de negocio.
- **Permisos Personalizados en `permissions.py`:** Para lógicas de autorización complejas, crear clases de permisos que hereden de `BasePermission`. Esto mantiene las vistas limpias.

### 4.2. Ejemplo de CRUD con DRF (ViewSet)

```python
# models.py
from django.db import models

class Curso(models.Model):
  titulo = models.CharField(max_length=120)
  publicado = models.BooleanField(default=False)
  creado_en = models.DateTimeField(auto_now_add=True)

  def __str__(self) -> str:
    return self.titulo

# serializers.py
from rest_framework import serializers
from .models import Curso

class CursoSerializer(serializers.ModelSerializer):
  class Meta:
    model = Curso
    fields = ["id", "titulo", "publicado", "creado_en"]
    read_only_fields = ["id", "creado_en"]

# views.py
from rest_framework import viewsets
from django_filters.rest_framework import DjangoFilterBackend

class CursoViewSet(viewsets.ModelViewSet):
  queryset = Curso.objects.all().order_by("-creado_en")
  serializer_class = CursoSerializer
  filter_backends = [DjangoFilterBackend]
  filterset_fields = ["publicado"]

# urls.py (de la app)
from rest_framework.routers import DefaultRouter
from .views import CursoViewSet

router = DefaultRouter()
router.register(r"cursos", CursoViewSet, basename="curso")

urlpatterns = router.urls
```

### 4.3. Seguridad: Limitar Peticiones con Throttling

Protege tu API contra ataques de DoS y abuso.

**Configuración Global en `settings.py`:**

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',  # Para IPs anónimas
        'rest_framework.throttling.UserRateThrottle'   # Para usuarios autenticados
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```

Si un usuario excede el límite, recibirá una respuesta `HTTP 429 Too Many Requests`.

### 4.4. Documentación automática con `drf-spectacular`

Genera documentación OpenAPI 3.0 (Swagger UI, Redoc) a partir de tu código.

1. `pip install drf-spectacular`
2. Añadir `drf_spectacular` a `INSTALLED_APPS`.
3. En `settings.py`:

   ```python
   REST_FRAMEWORK = {
       "DEFAULT_SCHEMA_CLASS": "drf_spectacular.openapi.AutoSchema",
   }
   SPECTACULAR_SETTINGS = {
       "TITLE": "API del Curso DRF",
       "DESCRIPTION": "Documentación interactiva de la API",
       "VERSION": "1.0.0",
   }
   ```

4. En el `urls.py` del proyecto, añade las rutas para la UI:

   ```python
   from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView, SpectacularRedocView

   urlpatterns = [
       # ...
       path("api/schema/", SpectacularAPIView.as_view(), name="schema"),
       path("api/docs/", SpectacularSwaggerView.as_view(url_name="schema"), name="swagger-ui"),
       path("api/redoc/", SpectacularRedocView.as_view(url_name="schema"), name="redoc"),
   ]
   ```

---

## 5. FastAPI

### 5.1. Conexión a Base de Datos con SQLModel

```python
# db.py
from typing import Annotated
from fastapi import Depends, FastAPI
from sqlmodel import Session, create_engine, SQLModel

sqlite_name = "db.sqlite3"
sqlite_url = f"sqlite:///{sqlite_name}"

engine = create_engine(sqlite_url)

def create_all_tables():
    SQLModel.metadata.create_all(engine)

def get_session():
    with Session(engine) as session:
        yield session

SessionDep = Annotated[Session, Depends(get_session)]
```

---

### 5.2. Construcción de Consultas con SQLModel

Para construir consultas con múltiples condiciones, SQLModel (y SQLAlchemy) ofrece una sintaxis fluida y legible.

#### Encadenamiento de .where() (AND implícito)

Puedes encadenar múltiples llamadas al método .where(). SQLModel las combinará automáticamente usando un operador lógico AND. Esta es la forma más común y legible para condiciones AND simples.

```python
# Buscar un CustomerPlan que pertenezca a un customer_id y tenga el estado "activo"
query = select(CustomerPlan).where(CustomerPlan.customer_id == customer_id).where(CustomerPlan.status == "activo")
results = session.exec(query).all()

#Esto se traduce en una sentencia SQL similar a:
WHERE customerplan.customer_id = ? AND customerplan.status = ?
```

#### Condiciones Complejas con _and_ y _or_

Para consultas que requieren una lógica más compleja, como combinar condiciones AND y OR, puedes importar y usar _and_ y _or_ directamente desde sqlmodel.

```python
from sqlmodel import select, and_, or_
# Ejemplo con or_
# Ejemplo con OR
#Buscar planes que estén activos O pendientes
query_or = select(CustomerPlan).where(or_(CustomerPlan.status == "activo", CustomerPlan.status == "pendiente"))

#Ejemplo combinando and_ y or_
# Buscar planes de un cliente específico que estén activos O pendientes
customer_plan_db_complex = session.exec(
    select(CustomerPlan).where(and_(CustomerPlan.customer_id == customer_id,or_(
      CustomerPlan.status == "activo",
      CustomerPlan.status == "pendiente")))).all()
```

---

### 5.3. Seguridad: Queries Parametrizadas (Prevenir SQL Injection)

**Nunca concatenes valores directamente en queries SQL.** Esto abre la puerta a ataques de inyección SQL.

#### ❌ Incorrecto (Vulnerable)

```python
# PELIGROSO: Concatenación directa
days = 7
cursor.execute(f"""
    DELETE FROM sesiones WHERE fecha < datetime('now', '-{days} days')
""")

# PELIGROSO: Usando .format()
cursor.execute("""
    SELECT * FROM usuarios WHERE nombre = '{}'
""".format(nombre_usuario))
```

Un atacante podría ingresar: `Juan'; DROP TABLE usuarios;--` y borrar tu tabla.

#### ✅ Correcto (Seguro)

Usa **placeholders** (`?` en SQLite, `%s` en PostgreSQL/MySQL) y pasa los valores como tupla:

```python
from datetime import datetime, timedelta

# Calcular en Python, pasar como parámetro
cutoff_date = datetime.now() - timedelta(days=7)
cutoff_str = cutoff_date.strftime('%Y-%m-%d %H:%M:%S')

cursor.execute(
    "DELETE FROM sesiones WHERE fecha < ?",
    (cutoff_str,)  # ← Tupla con valores
)

# Múltiples parámetros
cursor.execute(
    "INSERT INTO usuarios (nombre, email) VALUES (?, ?)",
    (nombre, email)
)
```

#### 🔑 Reglas Clave

| Regla | Ejemplo |
|-------|---------|
| Un parámetro | `cursor.execute("... WHERE id = ?", (id,))` |
| Múltiples parámetros | `cursor.execute("... VALUES (?, ?, ?)", (a, b, c))` |
| Tupla de un elemento | Requiere coma: `(valor,)` no `(valor)` |

#### Con SQLModel/SQLAlchemy

SQLModel ya maneja esto automáticamente cuando usas sus métodos:

```python
# ✅ Seguro: SQLModel parametriza internamente
query = select(Usuario).where(Usuario.nombre == nombre_usuario)
```

---

## 6. Modelos Arquitectónicos en Python

### 6.1. Panorama y Adopción Progresiva

1. **Comenzar con MVT (Django) o una estructura simple en FastAPI.**
2. **Extraer a una Capa de Servicios:** Cuando las vistas/routers crecen, mueve la lógica de negocio a funciones o clases en un `services.py`. Esto mejora la reutilización y los tests.
3. **Introducir Arquitectura Hexagonal (Ports & Adapters):** Si el sistema depende de múltiples servicios externos (BDs, APIs, colas), define "puertos" (interfaces) para el dominio y crea "adaptadores" para las implementaciones concretas.
4. **Considerar DDD/Clean Architecture:** Para dominios de negocio muy complejos, donde las reglas y el lenguaje compartido son críticos.

| Aspecto           | MVT (Django)                      | MVC Clásico             | Hexagonal / Clean                                       | Service Layer Simple              |
| ----------------- | --------------------------------- | ----------------------- | ------------------------------------------------------- | --------------------------------- |
| Enfoque principal | Productividad + convención        | Separación presentación | Aislar dominio de infraestructura                       | Extraer lógica reusable           |
| Tests dominio     | Menos aislado si lógica en modelo | Medio                   | Muy aislado (puro)                                      | Fácil si servicios puros          |
| Curva complejidad | Baja                              | Media                   | Alta inicial                                            | Baja                              |
| Cuándo aplicar    | Inicio / CRUD educativo           | Apps web tradicionales  | Sistemas con reglas complejas / múltiples integraciones | Cuando modelos/serializers crecen |

---

## Apéndice A: Chuleta de Comandos

### Entorno Virtual y Dependencias

- **Crear entorno virtual:** `python -m venv .venv`
- **Activar (Linux/macOS):** `source .venv/bin/activate`
- **Activar (Windows):** `.venv\Scripts\activate`
- **Instalar dependencias:** `pip install -r requirements.txt`
- **Guardar dependencias:** `pip freeze > requirements.txt`
- **Ocultar info sensible:** `pip install python-dotenv`

### Django

- **Crear proyecto:** `django-admin startproject <proyecto>`
- **Iniciar servidor:** `python manage.py runserver`
- **Crear aplicación:** `python manage.py startapp <nombre_app>`
- **Crear migraciones:** `python manage.py makemigrations`
- **Aplicar migraciones:** `python manage.py migrate`
- **Crear superusuario:** `python manage.py createsuperuser`
- **Shell de Django:** `python manage.py shell`
- **Conector PostgreSQL:** `pip install psycopg2-binary`
- **Desplegar estáticos:** `python manage.py collectstatic` (Asegúrate de tener `STATIC_ROOT` definido en `settings.py`).

### FastAPI

- **Instalar:** `pip install "fastapi[standard]"`
- **Ejecutar (dev):** `uvicorn main:app --reload` (donde `main` es `main.py` y `app` es la instancia de FastAPI).
- **Ejecutar con CLI:** `fastapi dev main.py` / `fastapi dev app/main.py`

### Testing

- **Ejecutar pruebas con pytest:** `pytest`
- **Ejecutar pruebas con unittest:** `python -m unittest discover tests`

### Base de Datos (SQLite)

- **Entrar a la consola de SQLite:** `sqlite3 db.sqlite3`
- **listas las tablas:** `.tables`
- **chequear el esquema de una tabla:** `.schema <nombre_tabla>`
- **Salir de SQLite:** `.exit`

---
