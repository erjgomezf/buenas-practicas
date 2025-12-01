# 📘 Buenas Prácticas de Ingeniería de Software y Referencia General

Documento vivo para acumular principios, patrones, comandos y referencias técnicas multilinguaje para el desarrollo de software.

---

## 1. Principios Fundamentales de Diseño

### 1.1. Principios SOLID

Estos principios son la base para construir software robusto, mantenible y escalable, independientemente del lenguaje.

- **S - Responsabilidad Única (SRP):** Cada componente (clase, función, módulo) debe tener una sola razón para cambiar.
- **O - Abierto/Cerrado (OCP):** El software debe estar abierto a la extensión, pero cerrado a la modificación.
- **L - Sustitución de Liskov (LSP):** Las subclases deben ser sustituibles por sus superclases sin alterar la lógica del programa.
- **I - Segregación de Interfaces (ISP):** Es preferible tener muchas interfaces pequeñas y específicas que una grande y monolítica.
- **D - Inversión de Dependencias (DIP):** Los módulos de alto nivel no deben depender de los de bajo nivel. Ambos deben depender de abstracciones.

### 1.2. Otros Principios Clave

- **DRY (Don't Repeat Yourself):** No te repitas. Abstrae y reutiliza la lógica común para evitar la duplicación de código.
- **KISS (Keep It Simple, Stupid):** Mantén las cosas simples. Evita la complejidad innecesaria.
- **YAGNI (You Ain't Gonna Need It):** No lo vas a necesitar. No implementes funcionalidades que no sean estrictamente necesarias en el momento.

---

## 2. Ciclo de Vida y Calidad del Software

### 2.1. Ciclo de Desarrollo Recomendado

1.  **Análisis:** Entender el problema y definir criterios de aceptación claros.
2.  **Diseño Ligero:** Definir modelos de datos y contratos (interfaces) antes de escribir la lógica compleja.
3.  **Implementación Guiada por Pruebas (TDD/BDD):** Escribir un test que falle -> implementar el código mínimo para que pase -> refactorizar.
4.  **Revisión Técnica:** Evaluar legibilidad, acoplamiento, duplicación y nombrado del código.
5.  **Definition of Done:** El código funciona, los tests pasan, la documentación está actualizada y no se ha introducido deuda técnica evidente.

### 2.2. Checklist Pre-PR (Pull Request)

1.  ¿El nombre de la rama describe la intención? (ej. `feature/user-authentication`)
2.  ¿El código nuevo tiene tests que cubren el "happy path" y los casos de error?
3.  ¿Se ha ejecutado el formateador de código y el linter del proyecto?
4.  ¿El `diff` muestra solo los cambios relacionados con la tarea?
5.  ¿Se ha actualizado la documentación si es necesario?
6.  ¿Los commits son atómicos y tienen mensajes claros y descriptivos?

---

## 3. Arquitectura de Software

Patrones para organizar la estructura de una aplicación a gran escala.

- **Arquitectura en Capas:** Separa la aplicación en capas horizontales (Presentación, Lógica de Negocio, Acceso a Datos). Es un buen punto de partida para muchas aplicaciones.
- **Arquitectura Hexagonal (Puertos y Adaptadores):** Desacopla el núcleo de la aplicación (dominio) de la infraestructura (frameworks, BDs, APIs externas). El dominio define "puertos" (interfaces) y la infraestructura provee "adaptadores" (implementaciones).
- **Clean Architecture:** Un enfoque más estricto que la Hexagonal, con capas concéntricas (Entidades -> Casos de Uso -> Adaptadores de Interfaz -> Frameworks). La regla principal es que las dependencias solo apuntan hacia adentro.
- **Microservicios:** Descompone una aplicación grande en un conjunto de servicios pequeños, independientes y débilmente acoplados que se comunican a través de APIs. Ideal para sistemas muy grandes y equipos distribuidos.
- **Domain-Driven Design (DDD):** Un enfoque para software complejo que se centra en el dominio del negocio, usando un lenguaje ubicuo compartido entre desarrolladores y expertos del dominio.
- **CQRS (Command Query Responsibility Segregation):** Separa los modelos para leer datos (Queries) de los modelos para escribir datos (Commands). Útil para sistemas con altos volúmenes de lectura o escritura que requieren optimizaciones diferentes.

---

## 4. Diseño de APIs y Servicios Backend

### 4.1. Métodos HTTP (Referencia para APIs REST)

| Método  | Seguro (Safe) | Idempotente                                                  | Body habitual  | Semántica principal           |
| ------- | ------------- | ------------------------------------------------------------ | -------------- | ----------------------------- |
| GET     | Sí            | Sí                                                           | No (se ignora) | Lectura / listado             |
| POST    | No            | No                                                           | Sí             | Crear / acción no idempotente |
| PUT     | No            | Sí                                                           | Sí             | Reemplazo completo            |
| PATCH   | No            | No (a veces tratado como idempotente si parche determinista) | Sí             | Actualización parcial         |
| DELETE  | No            | Sí                                                           | No             | Eliminación                   |
| HEAD    | Sí            | Sí                                                           | No             | Metadatos (GET sin cuerpo)    |
| OPTIONS | Sí            | Sí                                                           | No             | Capacidades del servidor      |

- **Idempotencia:** Una operación es idempotente si realizarla una o N veces produce el mismo resultado (ej. `DELETE /users/1`).
- **Seguridad (Safe):** Un método es seguro si no modifica el estado del servidor (ej. `GET`, `HEAD`).

### 4.2. Códigos de Estado HTTP Comunes en APIs

| Código | Significado       | Cuándo usarlo                                                     |
| ------ | ----------------- | ----------------------------------------------------------------- |
| 200    | OK                | Peticiones `GET` exitosas.                                        |
| 201    | Created           | Creación de un recurso exitosa (`POST`).                          |
| 204    | No Content        | Petición exitosa sin contenido que devolver (`DELETE`).           |
| 400    | Bad Request       | Error del cliente (ej. JSON malformado, datos inválidos).         |
| 401    | Unauthorized      | El cliente no está autenticado.                                   |
| 403    | Forbidden         | El cliente está autenticado pero no tiene permisos.               |
| 404    | Not Found         | El recurso solicitado no existe.                                  |
| 409    | Conflict          | Conflicto con el estado actual del recurso (ej. email duplicado). |
| 429    | Too Many Requests | El cliente ha excedido el límite de peticiones (throttling).      |
| 500    | Internal Server Error | Error inesperado en el servidor. No filtrar detalles sensibles al cliente. |

### 4.3. Observabilidad

- **Logging Estructurado:** Usa logs en formato JSON. Registra `INFO` para flujos de negocio, `WARNING` para situaciones anómalas pero controladas, y `ERROR` para fallos inesperados.
- **Métricas:** Mide el rendimiento de la aplicación (latencia, tasa de errores, uso de recursos).
- **Tracing:** Sigue el flujo de una petición a través de múltiples servicios para depurar cuellos de botella.

---

## 5. Principios de Frontend

- **Separación de Responsabilidades:**
  - **HTML:** Para la estructura y el contenido semántico.
  - **CSS:** Para el estilo visual y la presentación.
  - **JavaScript:** Para la interactividad y el comportamiento dinámico.
- **Diseño Adaptable (Responsive Design):** Asegura que la aplicación se vea y funcione bien en todos los dispositivos.
- **Accesibilidad (A11y):** Construye aplicaciones que puedan ser utilizadas por el mayor número de personas posible, siguiendo estándares como WCAG.
- **Componentización:** Piensa en la UI como un conjunto de bloques reutilizables e independientes.

---

## 6. Catálogo de Patrones de Diseño

Soluciones probadas a problemas comunes de diseño de software.

### 6.1. Creacionales

| Patrón               | Propósito Principal                                                |
| :------------------- | :----------------------------------------------------------------- |
| **Singleton**        | Garantizar una única instancia de una clase.                       |
| **Factory Method**   | Delegar la creación de objetos a subclases.                        |
| **Abstract Factory** | Crear familias de objetos relacionados.                            |
| **Builder**          | Construir un objeto complejo paso a paso.                          |
| **Prototype**        | Clonar un objeto pre-configurado para evitar una creación costosa. |

### 6.2. Estructurales

| Patrón        | Intención Principal                                |
| :------------ | :------------------------------------------------- |
| **Adapter**   | Convertir una interfaz en otra.                    |
| **Bridge**    | Desacoplar abstracción de implementación.          |
| **Composite** | Tratar a un grupo de objetos como a uno solo.      |
| **Decorator** | Añadir comportamiento a un objeto dinámicamente.   |
| **Facade**    | Simplificar la interfaz de un subsistema complejo. |
| **Flyweight** | Ahorrar memoria compartiendo estado.               |
| **Proxy**     | Controlar el acceso a un objeto.                   |

### 6.3. Comportamiento

| Patrón                      | Intención Principal                                     |
| :-------------------------- | :------------------------------------------------------ |
| **Strategy**                | Encapsular algoritmos intercambiables.                  |
| **State**                   | Cambiar el comportamiento de un objeto según su estado. |
| **Mediator**                | Centralizar la comunicación entre objetos.              |
| **Command**                 | Encapsular una acción en un objeto.                     |
| **Observer**                | Notificar a múltiples objetos sobre un cambio.          |
| **Chain of Responsibility** | Pasar una solicitud por una cadena de manejadores.      |

---

## Apéndice A: Chuleta de Comandos

### Control de Versiones (Git)

- **Inicializar repositorio:** `git init`
- **Ver estado:** `git status`
- **Añadir todos los cambios:** `git add .`
- **Confirmar cambios:** `git commit -m "feat: Mensaje descriptivo"` (ver Convencional Commits)
- **Ver log compacto:** `git log --oneline --graph --decorate --all`
- **Crear y cambiar a una nueva rama:** `git checkout -b <nombre-rama>`
- **Cambiar de rama:** `git checkout <nombre-rama>`
- **Fusionar rama actual con otra:** `git merge <otra-rama>`

### Contenedores (Docker & Docker Compose)

- **Construir imágenes:** `docker-compose build`
- **Iniciar servicios (en segundo plano):** `docker-compose up -d`
- **Detener servicios:** `docker-compose stop`
- **Detener y eliminar contenedores:** `docker-compose down`
- **Ver logs de un servicio:** `docker-compose logs -f <nombre-servicio>`
- **Entrar a un contenedor:** `docker-compose exec <nombre-servicio> bash`
- **Listar contenedores en ejecución:** `docker ps`

### Herramientas de API (cURL)

- **Petición GET:** `curl https://api.example.com/resource`
- **Petición POST con JSON:** `curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com/resource`
- **Con Header de Autorización:** `curl -H "Authorization: Bearer <token>" https://api.example.com/resource`