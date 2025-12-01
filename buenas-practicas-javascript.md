# 📘 Buenas Prácticas de JavaScript

Guía de buenas prácticas para escribir JavaScript limpio, mantenible y eficiente, con énfasis especial en el uso dentro de N8N.

---

## 1. Estilo y Convenciones

### 1.1. Nomenclatura

- **Variables y funciones:** `camelCase`
  ```javascript
  const nombreCliente = "Juan";
  function calcularDias() { }
  ```

- **Constantes:** `UPPER_SNAKE_CASE` (solo para valores verdaderamente constantes)
  ```javascript
  const MAX_INTENTOS = 3;
  const API_URL = "https://api.example.com";
  ```

- **Clases:** `PascalCase`
  ```javascript
  class EmailGenerator { }
  ```

### 1.2. Declaración de Variables

- **Usa `const` por defecto**, `let` si necesitas reasignar
- **NUNCA uses `var`**

```javascript
// ✅ Bien
const nombre = "Juan";
let contador = 0;

// ❌ Mal
var nombre = "Juan";
```

### 1.3. Template Literals

Usa template literals en lugar de concatenación:

```javascript
// ✅ Bien
const mensaje = `Hola ${nombre}, tu evento es el ${fecha}`;

// ❌ Mal
const mensaje = "Hola " + nombre + ", tu evento es el " + fecha;
```

---

## 2. Manejo de Datos

### 2.1. Destructuring

```javascript
// Objetos
const { nombre, email, telefono } = cliente;

// Arrays
const [primero, segundo] = lista;

// Con valores por defecto
const { paquete = "Básico" } = solicitud;
```

### 2.2. Spread Operator

```javascript
// Copiar objetos
const nuevoObjeto = { ...objetoOriginal, campo_nuevo: "valor" };

// Combinar arrays
const todosLosItems = [...items1, ...items2];
```

### 2.3. Array Methods

Usa métodos funcionales en lugar de loops:

```javascript
// map - transformar
const nombres = clientes.map(c => c.nombre);

// filter - filtrar
const urgentes = solicitudes.filter(s => s.dias < 7);

// reduce - acumular
const total = precios.reduce((sum, p) => sum + p, 0);

// find - buscar
const cliente = clientes.find(c => c.id === 123);
```

---

## 3. Funciones

### 3.1. Arrow Functions

```javascript
// ✅ Bien - conciso
const sumar = (a, b) => a + b;

// ✅ Bien - con cuerpo
const validar = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};

// ❌ Evitar - usar function solo si necesitas 'this'
function sumar(a, b) {
  return a + b;
}
```

### 3.2. Parámetros por Defecto

```javascript
function enviarEmail(destinatario, asunto = "Sin asunto") {
  // ...
}
```

### 3.3. Return Temprano

```javascript
// ✅ Bien
function validarCliente(cliente) {
  if (!cliente.nombre) return { valido: false, error: "Falta nombre" };
  if (!cliente.email) return { valido: false, error: "Falta email" };
  
  return { valido: true };
}

// ❌ Mal - anidación profunda
function validarCliente(cliente) {
  if (cliente.nombre) {
    if (cliente.email) {
      return { valido: true };
    } else {
      return { valido: false, error: "Falta email" };
    }
  } else {
    return { valido: false, error: "Falta nombre" };
  }
}
```

---

## 4. Manejo de Errores

### 4.1. Try/Catch

```javascript
try {
  const resultado = operacionRiesgosa();
  return resultado;
} catch (error) {
  console.error("Error en operación:", error.message);
  return null; // o lanzar error personalizado
}
```

### 4.2. Validación Defensiva

```javascript
function procesarDatos(datos) {
  // Validar entrada
  if (!datos || typeof datos !== 'object') {
    throw new Error("Datos inválidos");
  }
  
  // Procesar con seguridad
  const nombre = datos.nombre?.trim() || "Sin nombre";
  return nombre;
}
```

---

## 5. Buenas Prácticas en N8N

### 5.1. Acceso a Datos

```javascript
// Obtener datos del nodo actual
const input = $input.item.json;

// Obtener datos de un nodo específico
const datosFormulario = $('formularioValido').item.json;

// Acceso seguro con optional chaining
const email = input.cliente?.email || "no-email@example.com";
```

### 5.2. Retornar Datos

```javascript
// ✅ Bien - retornar objeto con spread
return {
  ...input,
  campo_nuevo: "valor",
  timestamp: new Date().toISOString()
};

// ❌ Mal - mutar el input
input.campo_nuevo = "valor";
return input;
```

### 5.3. Validación de Datos

```javascript
const input = $input.item.json;
const errores = [];

// Validar campos requeridos
if (!input.nombre || input.nombre.length < 3) {
  errores.push("Nombre inválido");
}

if (!input.email || !input.email.includes('@')) {
  errores.push("Email inválido");
}

return {
  ...input,
  datos_validos: errores.length === 0,
  lista_errores: errores
};
```

### 5.4. Generación de HTML

```javascript
// Usa template literals para HTML
const nombre = input.nombre_cliente;
const fecha = new Date(input.fecha_evento).toLocaleDateString('es-ES');

const html = `
<!DOCTYPE html>
<html>
<body>
  <h1>Hola ${nombre}</h1>
  <p>Tu evento es el ${fecha}</p>
</body>
</html>
`;

return { html };
```

---

## 6. Patrones Comunes

### 6.1. Formateo de Fechas

```javascript
// Fecha legible en español
const fechaFormateada = new Date(input.fecha_evento)
  .toLocaleDateString('es-ES', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });

// ISO para APIs
const timestamp = new Date().toISOString();
```

### 6.2. Limpieza de Strings

```javascript
const nombre = input.nombre.trim();
const telefono = input.telefono.replace(/\D/g, ''); // Solo dígitos
const email = input.email.toLowerCase().trim();
```

### 6.3. Condicionales Complejas

```javascript
// ✅ Bien - lógica clara
const esUrgente = dias < 7 || paquete === "Enterprise" || 
                  (tipo === "Corporativo" && dias < 14);

// ✅ Mejor - extraer a función
function calcularUrgencia(dias, paquete, tipo) {
  if (dias < 7) return "ALTA";
  if (paquete === "Enterprise") return "ALTA";
  if (tipo === "Corporativo" && dias < 14) return "ALTA";
  if (dias < 30) return "MEDIA";
  return "NORMAL";
}
```

---

## 7. Antipatrones a Evitar

### ❌ No uses `==` (usa `===`)

```javascript
// ❌ Mal
if (valor == "5") { }

// ✅ Bien
if (valor === "5") { }
```

### ❌ No modifiques parámetros

```javascript
// ❌ Mal
function procesarCliente(cliente) {
  cliente.procesado = true;
  return cliente;
}

// ✅ Bien
function procesarCliente(cliente) {
  return { ...cliente, procesado: true };
}
```

### ❌ No uses callbacks anidados (callback hell)

```javascript
// ❌ Mal
getData(function(a) {
  getMoreData(a, function(b) {
    getMoreData(b, function(c) {
      // ...
    });
  });
});

// ✅ Bien - usa async/await
async function procesarDatos() {
  const a = await getData();
  const b = await getMoreData(a);
  const c = await getMoreData(b);
  return c;
}
```

---

## 8. Checklist de Calidad

Antes de finalizar tu código JavaScript en N8N:

- [ ] ¿Usaste `const`/`let` en lugar de `var`?
- [ ] ¿Los nombres de variables son descriptivos?
- [ ] ¿Validaste los datos de entrada?
- [ ] ¿Usaste template literals para strings complejos?
- [ ] ¿Retornaste un nuevo objeto en lugar de mutar?
- [ ] ¿Manejaste posibles errores?
- [ ] ¿El código es legible sin comentarios excesivos?
- [ ] ¿Usaste métodos de array en lugar de loops cuando es posible?

---

**Última Actualización**: 2025-12-01
