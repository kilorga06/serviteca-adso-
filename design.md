# Design Document — Serviteca ADSO Landing Page

## Overview

Single-page application (SPA) estática desarrollada en **HTML5, CSS3 y JavaScript vanilla**, sin frameworks externos ni dependencias de terceros. Implementa una arquitectura MVC ligera distribuida en cuatro módulos (`store.js`, `validators.js`, `ui.js`, `main.js`) que se cargan desde el mismo directorio. Toda la persistencia se realiza mediante `localStorage`. El diseño aplica una paleta oscura/neón (fondo `#0a0a0a` / `#1a1a1a`, acento amarillo neón `#FFE600`) y es completamente responsivo entre 320 px y 1920 px.

---

## Architecture

### Patrón MVC Ligero

```
┌─────────────────────────────────────────────────────────────┐
│                         index.html                          │
│  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌────────┐  │
│  │  main.js │  │ validators.js│  │  store.js │  │  ui.js │  │
│  │(Controlador)│ │ (Validación) │  │  (Modelo) │  │ (Vista)│  │
│  └──────────┘  └──────────────┘  └──────────┘  └────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                     ┌────────▼────────┐
                     │   localStorage  │
                     │  (Persistencia) │
                     └─────────────────┘
```

| Módulo | Responsabilidad |
|---|---|
| `store.js` | CRUD sobre `localStorage`; serialización/deserialización JSON; manejo de errores de I/O |
| `validators.js` | Funciones puras de validación; retornan `{ valid: boolean, errors: string[] }` |
| `ui.js` | Renderizado de listas, formularios, mensajes de error/éxito y estado activo del menú |
| `main.js` | Punto de entrada; registra event listeners; coordina store ↔ validators ↔ ui |

### Flujo de datos (ejemplo: registrar cliente)

```
Usuario → Formulario_Cliente
         → main.js: handleSubmitCliente()
             → validators.js: validateCliente(data)   // puro
                 ← { valid, errors }
             → store.js: saveCliente(cliente)           // I/O
                 → localStorage.setItem(...)
             → ui.js: renderClientes(store.getClientes()) // vista
```

---

## Components and Interfaces

### Estructura HTML

```
<body>
  <header id="encabezado">          <!-- logo SVG + "Serviteca ADSO" + <nav> -->
    <nav id="nav-principal">        <!-- enlaces fijos con IntersectionObserver -->
  <main>
    <section id="clientes">         <!-- Registro de clientes y vehículos -->
    <section id="agenda">           <!-- Agendamiento de citas -->
    <section id="historial">        <!-- Historial de vehículos atendidos -->
  <footer id="pie">                 <!-- Identidad del negocio -->
```

### Componentes UI

#### `ui.renderClientes(clientes, vehiculos)`
Renderiza la lista de clientes registrados. Genera las subsecciones "Automóviles" y "Motocicletas" filtrando `vehiculos` por `tipo`.

#### `ui.renderCitas(citas)`
Renderiza la lista de citas con `estado === "pendiente"`, ordenadas ascendentemente por `fechaRevision`. Cada ítem incluye el botón "Marcar como atendido".

#### `ui.renderHistorial(citas)`
Renderiza las citas con `estado === "atendido"`, ordenadas descendentemente por `fechaRevision`. Actualiza el contador del encabezado de la sección.

#### `ui.renderHistorialItem(cita)`
Genera la cadena HTML para un ítem individual del historial, incluyendo: placa, marca, modelo, tipo de vehículo, tipo de reparación, servicios realizados, fecha y hora de revisión y fecha y hora de devolución.

#### `ui.showError(campo, mensaje)` / `ui.clearErrors(formId)`
Muestra u oculta mensajes de validación junto a cada campo del formulario. Los campos con error reciben borde en `--color-error`.

#### `ui.setNavActivo(sectionId)`
Añade la clase CSS `nav--activo` al enlace cuyo `href` apunta a `sectionId` y la quita de todos los demás enlaces del menú.

### API de `store.js`

```javascript
// Clientes
store.getClientes()                    // → Cliente[]
store.saveCliente(cliente)             // → void  (lanza StorageError si falla)
store.clienteExiste(id)                // → boolean

// Vehículos
store.getVehiculos()                   // → Vehiculo[]
store.saveVehiculo(vehiculo)           // → void
store.placaExiste(placa)               // → boolean
store.getVehiculosPorTipo(tipo)        // → Vehiculo[]

// Citas
store.getCitas()                       // → Cita[]
store.saveCita(cita)                   // → void
store.updateEstadoCita(citaId, estado) // → void
store.getCitasPendientes()             // → Cita[]  (ordenadas ASC por fechaRevision)
store.getCitasAtendidas()              // → Cita[]  (ordenadas DESC por fechaRevision)

// Lookups
store.lookupClientePorDocumento(doc)   // → Cliente | null
store.lookupVehiculoPorTelefono(tel)   // → Vehiculo | null
```

### API de `validators.js`

```javascript
validators.validateCliente(data)                      // → { valid: boolean, errors: { campo: string }[] }
validators.validateVehiculo(data)                     // → { valid: boolean, errors: { campo: string }[] }
validators.validateCita(data)                         // → { valid: boolean, errors: { campo: string }[] }
validators.validateEmail(email)                       // → boolean
validators.validatePlaca(placa)                       // → boolean  (≤7 chars alfanuméricos)
validators.validateFechas(revision, devolucion)       // → boolean  (devolucion > revision)
```

---

## Data Models

### Cliente

```javascript
/**
 * @typedef {Object} Cliente
 * @property {string} id        - Número de identificación (máx. 20 chars, único)
 * @property {string} telefono  - Número de teléfono (máx. 15 chars)
 * @property {string} email     - Correo electrónico (contiene "@" y dominio con punto)
 * @property {string} nombres   - Nombres (máx. 100 chars)
 * @property {string} apellidos - Apellidos (máx. 100 chars)
 */
```

### Vehículo

```javascript
/**
 * @typedef {Object} Vehiculo
 * @property {string} placa     - Número de placa (máx. 7 chars alfanuméricos, único)
 * @property {string} clienteId - FK → Cliente.id
 * @property {string} marca     - Marca (máx. 50 chars)
 * @property {string} modelo    - Modelo (máx. 50 chars)
 * @property {"Automóvil"|"Motocicleta"} tipo - Tipo de vehículo
 */
```

### Cita

```javascript
/**
 * @typedef {Object} Cita
 * @property {string}   citaId           - UUID generado en cliente (crypto.randomUUID)
 * @property {string}   clienteId        - FK → Cliente.id
 * @property {string}   clienteTelefono  - Teléfono del cliente (desnormalizado)
 * @property {string}   placa            - FK → Vehiculo.placa
 * @property {"Automóvil"|"Motocicleta"} tipoVehiculo
 * @property {string}   tipoReparacion   - Descripción (máx. 100 chars)
 * @property {string[]} servicios        - Subconjunto no vacío de
 *                                         ["Cambio de aceite","Alineación","Sincronización","Lavado"]
 * @property {string}   fechaRevision    - ISO 8601 datetime-local
 * @property {string}   fechaDevolucion  - ISO 8601 datetime-local (> fechaRevision)
 * @property {"pendiente"|"atendido"} estado
 */
```

### Claves de localStorage

| Clave | Tipo | Descripción |
|---|---|---|
| `serviteca_clientes` | `Cliente[]` | Array serializado en JSON |
| `serviteca_vehiculos` | `Vehiculo[]` | Array serializado en JSON |
| `serviteca_citas` | `Cita[]` | Array serializado en JSON |

---

## Responsive Design

### Breakpoints

| Rango | Layout |
|---|---|
| 320 px – 599 px | 1 columna, menú colapsable (hamburger) |
| 600 px – 1023 px | 2 columnas en formularios |
| 1024 px – 1920 px | Layout de 3 columnas en secciones con sidebar |

### Sistema de Grid/Flex

- `display: flex` para el encabezado y el menú de navegación.
- `display: grid` con `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))` para las listas de clientes y vehículos.
- Unidades relativas (`rem`, `%`, `clamp()`) para tipografía y espaciados.
- Sin `overflow-x: hidden` en el `<body>`; el layout nunca genera scroll horizontal.

---

## Navigation & IntersectionObserver

```javascript
// main.js — activación del enlace activo del menú
const secciones = document.querySelectorAll('section[id]');
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        ui.setNavActivo(entry.target.id);
      }
    });
  },
  { threshold: 0.4 }  // sección al 40 % visible activa su enlace
);
secciones.forEach(s => observer.observe(s));

// Scroll suave al hacer clic
navLinks.forEach(link => {
  link.addEventListener('click', e => {
    e.preventDefault();
    document.querySelector(link.getAttribute('href'))
      .scrollIntoView({ behavior: 'smooth' });
  });
});
```

---

## Visual Design System

### Paleta de colores

| Token | Valor | Uso |
|---|---|---|
| `--color-bg` | `#0a0a0a` | Fondo principal |
| `--color-surface` | `#1a1a1a` | Tarjetas, formularios |
| `--color-accent` | `#FFE600` | Encabezados, botones CTA, elementos destacados |
| `--color-accent-dark` | `#b3a100` | Hover de botones (ratio contraste ≥ 4.5:1 sobre `#0a0a0a`) |
| `--color-text` | `#f0f0f0` | Texto principal |
| `--color-text-muted` | `#a0a0a0` | Texto secundario |
| `--color-error` | `#ff4d4d` | Mensajes de error |
| `--color-success` | `#4dff91` | Mensajes de éxito |

> Contraste `#f0f0f0` sobre `#0a0a0a` ≈ 18.1:1 — supera WCAG 2.1 AA (4.5:1).  
> Contraste `#FFE600` sobre `#0a0a0a` ≈ 14.7:1 — supera WCAG 2.1 AA.

### Tipografía

```css
:root {
  font-size: 16px;               /* base mínima */
  line-height: 1.5;              /* interlineado mínimo */
  font-family: system-ui, sans-serif;
}
h1 { font-size: clamp(1.5rem, 4vw, 2.5rem); font-weight: 700; }
h2 { font-size: clamp(1.25rem, 3vw, 2rem);  font-weight: 700; color: var(--color-accent); }
```

### Logo SVG inline

```html
<!-- Automóvil de frente, tamaño mínimo 48 × 48 px -->
<svg width="64" height="64" viewBox="0 0 64 64" aria-label="Logo Serviteca ADSO" role="img"
     xmlns="http://www.w3.org/2000/svg">
  <!-- carrocería -->
  <rect x="8" y="28" width="48" height="18" rx="4" fill="#FFE600"/>
  <!-- cabina -->
  <path d="M16 28 L20 16 L44 16 L48 28 Z" fill="#FFE600"/>
  <!-- ventanas -->
  <path d="M21 17 L24 27 L40 27 L43 17 Z" fill="#0a0a0a" opacity=".6"/>
  <!-- ruedas -->
  <circle cx="18" cy="46" r="8" fill="#1a1a1a" stroke="#FFE600" stroke-width="2"/>
  <circle cx="46" cy="46" r="8" fill="#1a1a1a" stroke="#FFE600" stroke-width="2"/>
  <!-- faros -->
  <rect x="8"  y="32" width="6" height="4" rx="1" fill="#fff" opacity=".9"/>
  <rect x="50" y="32" width="6" height="4" rx="1" fill="#fff" opacity=".9"/>
</svg>
```

---

## Error Handling

| Escenario | Comportamiento |
|---|---|
| `localStorage` no disponible al guardar | Capturar con `try/catch`; mostrar banner de error; conservar datos del formulario en memoria |
| `localStorage` falla al cargar (init) | Mostrar banner de error global; inicializar arrays vacíos; permitir uso en sesión |
| `localStorage` falla al actualizar estado | Mostrar error inline en la cita; mantener `estado = "pendiente"` en memoria |
| Campo inválido en formulario | Resaltar campo con borde `--color-error`; mostrar mensaje debajo del campo |
| ID de cliente duplicado | Mensaje inline "El número de identificación ya está registrado" |
| Placa duplicada | Mensaje inline "La placa ya está registrada" |
| Fecha de devolución ≤ revisión | Mensaje inline en campo `fechaDevolucion` |
| Documento no encontrado en lookup | Texto inline "Cliente no encontrado" bajo el campo |
| Teléfono sin vehículo en lookup | Texto inline "Vehículo no encontrado" bajo el campo |

---

## Testing Strategy

### Enfoque general

El proyecto no usa frameworks externos, por lo que la estrategia de pruebas se articula en tres capas que se ejecutan directamente en el navegador (o con Node.js para las funciones puras):

| Capa | Herramienta | Alcance |
|---|---|---|
| Unitaria / Property-based | `scripts/tests.js` (inline, sin dependencias) | `store.js`, `validators.js` |
| Integración E2E | Script manual + `scripts/e2e.js` | Flujo completo multimodular |
| Visual / Accesibilidad | Checklist manual | Responsividad y WCAG |

---

### Capa 1 — Tests unitarios y property-based (`scripts/tests.js`)

Archivo único que se carga en `index.html` mediante `<script src="scripts/tests.js" type="module">` solo durante desarrollo (`?test=1`). No se distribuye en producción.

#### Estructura del runner minimalista

```javascript
// scripts/tests.js
const results = { passed: 0, failed: 0, errors: [] };

function assert(condition, label) {
  if (condition) {
    results.passed++;
  } else {
    results.failed++;
    results.errors.push(label);
    console.error(`✗ FAIL: ${label}`);
  }
}

// Generador de datos aleatorios
function randString(len = 8) {
  return Math.random().toString(36).substring(2, 2 + len);
}
function randCliente() {
  return {
    id:        String(Math.floor(Math.random() * 1e10)),
    telefono:  String(Math.floor(Math.random() * 1e9)),
    email:     `${randString()}@${randString()}.com`,
    nombres:   randString(10),
    apellidos: randString(10),
  };
}
function randVehiculo(clienteId) {
  const tipos = ["Automóvil", "Motocicleta"];
  return {
    placa:     randString(6).toUpperCase(),
    clienteId,
    marca:     randString(6),
    modelo:    randString(4),
    tipo:      tipos[Math.floor(Math.random() * 2)],
  };
}
function randCita(clienteId, placa, tipoVehiculo) {
  const servicios = ["Cambio de aceite","Alineación","Sincronización","Lavado"];
  const base = Date.now() + Math.floor(Math.random() * 1e8);
  return {
    citaId:          crypto.randomUUID(),
    clienteId,
    clienteTelefono: String(Math.floor(Math.random() * 1e9)),
    placa,
    tipoVehiculo,
    tipoReparacion:  randString(15),
    servicios:       [servicios[Math.floor(Math.random() * servicios.length)]],
    fechaRevision:   new Date(base).toISOString(),
    fechaDevolucion: new Date(base + 3600000).toISOString(),
    estado:          "pendiente",
  };
}
```

#### Tests de `validators.js` (Properties 2, 4, 6, 9, 13, 14)

```javascript
// Property 4 — Validación de email (100 iteraciones)
for (let i = 0; i < 100; i++) {
  const valid = `${randString()}@${randString()}.com`;
  assert(validators.validateEmail(valid),   `P4: email válido acepta  [${valid}]`);
  const invalid = randString(8); // sin '@'
  assert(!validators.validateEmail(invalid), `P4: email inválido rechaza [${invalid}]`);
}

// Property 9 — Validación de placa (100 iteraciones)
for (let i = 0; i < 100; i++) {
  const ok = randString(Math.ceil(Math.random() * 7)).toUpperCase();
  assert(validators.validatePlaca(ok),   `P9: placa válida acepta    [${ok}]`);
  const bad = randString(8).toUpperCase(); // 8 chars → excede límite
  assert(!validators.validatePlaca(bad), `P9: placa inválida rechaza [${bad}]`);
}

// Property 2 — Rechazo de cliente con campo vacío
for (let i = 0; i < 50; i++) {
  const data = randCliente();
  const campo = ['id','telefono','email','nombres','apellidos'][i % 5];
  data[campo] = '   '; // solo espacios
  const r = validators.validateCliente(data);
  assert(!r.valid, `P2: campo '${campo}' vacío → rechazo`);
}

// Property 14 — Validación de rango de fechas (100 iteraciones)
for (let i = 0; i < 100; i++) {
  const base = Date.now() + Math.floor(Math.random() * 1e9);
  const rev = new Date(base).toISOString();
  const devOk  = new Date(base + 3600000).toISOString();  // +1 h
  const devBad = new Date(base - 3600000).toISOString();  // -1 h
  assert( validators.validateFechas(rev, devOk),  `P14: devolucion > revision OK`);
  assert(!validators.validateFechas(rev, devBad), `P14: devolucion < revision → false`);
}
```

#### Tests de `store.js` (Properties 1, 3, 5, 7, 8, 10, 11, 12, 15, 16, 21)

```javascript
// Limpiar localStorage antes de cada suite
function resetStore() {
  localStorage.removeItem('serviteca_clientes');
  localStorage.removeItem('serviteca_vehiculos');
  localStorage.removeItem('serviteca_citas');
}

// Property 1 — Round-trip Cliente (50 iteraciones)
resetStore();
for (let i = 0; i < 50; i++) {
  const c = randCliente();
  store.saveCliente(c);
  const found = store.getClientes().find(x => x.id === c.id);
  assert(found && found.email === c.email, `P1: round-trip cliente ${c.id}`);
}

// Property 3 — Unicidad de id de Cliente
resetStore();
const dup = randCliente();
store.saveCliente(dup);
store.saveCliente({ ...dup, nombres: 'Otro' }); // mismo id
const count = store.getClientes().filter(c => c.id === dup.id).length;
assert(count === 1, `P3: id duplicado → exactamente 1 registro`);

// Property 5 — Round-trip Vehículo (50 iteraciones)
resetStore();
for (let i = 0; i < 50; i++) {
  const c = randCliente(); store.saveCliente(c);
  const v = randVehiculo(c.id);
  store.saveVehiculo(v);
  const found = store.getVehiculos().find(x => x.placa === v.placa);
  assert(found && found.marca === v.marca, `P5: round-trip vehiculo ${v.placa}`);
}

// Property 8 — Partición por tipo (30 iteraciones)
resetStore();
for (let i = 0; i < 30; i++) {
  const c = randCliente(); store.saveCliente(c);
  const v = randVehiculo(c.id); store.saveVehiculo(v);
}
const autos  = store.getVehiculosPorTipo("Automóvil");
const motos  = store.getVehiculosPorTipo("Motocicleta");
const todos  = store.getVehiculos();
assert(autos.every(v => v.tipo === "Automóvil"),    `P8: partición autos sin motos`);
assert(motos.every(v => v.tipo === "Motocicleta"),  `P8: partición motos sin autos`);
assert(autos.length + motos.length === todos.length,`P8: unión = total`);

// Property 15 — Orden ascendente de pendientes
resetStore();
for (let i = 0; i < 20; i++) {
  const c = randCliente(); store.saveCliente(c);
  const v = randVehiculo(c.id); store.saveVehiculo(v);
  store.saveCita(randCita(c.id, v.placa, v.tipo));
}
const pendientes = store.getCitasPendientes();
for (let i = 0; i < pendientes.length - 1; i++) {
  assert(
    pendientes[i].fechaRevision <= pendientes[i+1].fechaRevision,
    `P15: pendientes orden ASC (índice ${i})`
  );
}

// Property 16 — Transición pendiente → atendido
resetStore();
const cTest = randCliente(); store.saveCliente(cTest);
const vTest = randVehiculo(cTest.id); store.saveVehiculo(vTest);
const citTest = randCita(cTest.id, vTest.placa, vTest.tipo);
store.saveCita(citTest);
store.updateEstadoCita(citTest.citaId, "atendido");
assert(!store.getCitasPendientes().some(c => c.citaId === citTest.citaId), `P16a: sale de pendientes`);
assert( store.getCitasAtendidas().some(c => c.citaId === citTest.citaId),  `P16b: entra en atendidas`);
const raw = JSON.parse(localStorage.getItem('serviteca_citas'));
assert(raw.find(c => c.citaId === citTest.citaId)?.estado === "atendido",  `P16c: persiste en localStorage`);
```

#### Reporte final

```javascript
console.info(`\nResultados: ${results.passed} ✓  ${results.failed} ✗`);
if (results.failed > 0) console.table(results.errors);
```

---

### Capa 2 — Tests de integración E2E (`scripts/e2e.js`)

Script manual que simula el flujo completo desde el navegador usando las funciones públicas de los módulos. Se ejecuta una sola vez por release en la consola de DevTools o como módulo cargado en `index.html?e2e=1`.

#### Flujo cubierto

```
1. Registro de cliente
   └─ validateCliente(data) → saveCliente(data)
   └─ Verificar: getClientes().length === 1
   └─ Verificar: UI renderiza el cliente en #clientes

2. Registro de vehículo asociado al cliente
   └─ lookupClientePorDocumento(id) → saveVehiculo(data)
   └─ Verificar: getVehiculos().length === 1
   └─ Verificar: UI renderiza el vehículo bajo "Automóviles" o "Motocicletas"

3. Agendamiento de cita
   └─ lookupVehiculoPorTelefono(tel) → validateCita(data) → saveCita(data)
   └─ Verificar: getCitasPendientes().length === 1
   └─ Verificar: UI renderiza la cita en #agenda con botón "Marcar como atendido"

4. Marcar cita como atendida
   └─ updateEstadoCita(citaId, "atendido")
   └─ Verificar: getCitasPendientes().length === 0
   └─ Verificar: getCitasAtendidas().length === 1

5. Historial
   └─ Verificar: UI renderiza el ítem en #historial
   └─ Verificar: contador de historial = 1
   └─ Verificar: HTML del ítem contiene placa, marca, modelo, tipo, servicios, fechas

6. Persistencia entre recargas
   └─ Simular recarga: llamar initApp() con el localStorage actual
   └─ Verificar: UI refleja estado idéntico al final del paso 5
```

---

### Capa 3 — Verificación manual de responsividad

Usar DevTools (Chrome / Firefox) con las siguientes vistas y comprobar cada punto:

| Viewport | Puntos a verificar |
|---|---|
| **320 px** | Menú hamburger visible y funcional; formularios en 1 columna; sin scroll horizontal; texto legible (≥ 14 px) |
| **600 px** | Formularios en 2 columnas; menú aún colapsable o expandido según diseño |
| **1024 px** | Layout de 3 columnas activado; menú horizontal; logo y nav en misma línea |
| **1280 px** | Sin desbordamientos; grid auto-fit rellena el espacio disponible |
| **1920 px** | Márgenes centrados; contenido nunca supera 1400 px de ancho si hay `max-width` definido |

Checklist por breakpoint:
- [ ] Sin `overflow-x` en `<html>` ni `<body>`
- [ ] Imágenes/SVG escalan sin distorsión
- [ ] Botones y campos táctiles ≥ 44 × 44 px (WCAG 2.5.5)
- [ ] Texto del menú y secciones no se superpone ni recorta

---

### Capa 4 — Verificación de accesibilidad

#### Contraste de color (automático con DevTools)

| Combinación | Ratio calculado | Requisito WCAG AA |
|---|---|---|
| `#f0f0f0` sobre `#0a0a0a` | ≈ 18.1:1 | ≥ 4.5:1 ✓ |
| `#FFE600` sobre `#0a0a0a` | ≈ 14.7:1 | ≥ 4.5:1 ✓ |
| `#a0a0a0` sobre `#0a0a0a` | ≈ 4.6:1  | ≥ 4.5:1 ✓ |
| `#ff4d4d` sobre `#1a1a1a` | ≈ 5.9:1  | ≥ 4.5:1 ✓ |
| `#4dff91` sobre `#1a1a1a` | ≈ 9.2:1  | ≥ 4.5:1 ✓ |

> La validación definitiva requiere revisión manual con tecnologías asistivas (lectores de pantalla).

#### Navegación por teclado (checklist manual)

- [ ] Todos los campos de formulario son alcanzables con `Tab`
- [ ] El orden del foco sigue el orden visual de arriba a abajo
- [ ] Los botones se activan con `Enter` y `Space`
- [ ] El menú de navegación es operable con `Tab` + `Enter`
- [ ] Los mensajes de error son anunciados por lectores de pantalla (`aria-live` o `role="alert"`)
- [ ] El logo SVG tiene `aria-label` y `role="img"` definidos
- [ ] No hay trampas de foco (el usuario puede salir de cualquier sección con `Tab`)

---



## Correctness Properties

*Una propiedad es una característica que debe sostenerse en todas las ejecuciones válidas del sistema — una declaración formal sobre lo que el software debe hacer. Las propiedades sirven como puente entre la especificación legible por humanos y las garantías de corrección verificables automáticamente.*

### Property 1: Round-trip de persistencia de Cliente

*Para cualquier* objeto `Cliente` con campos válidos (id, telefono, email, nombres, apellidos), guardarlo en el store y luego recuperarlo mediante `store.getClientes()` debe producir un array que contenga un objeto con exactamente los mismos valores de campo.

**Validates: Requirements 3.2, 8.1, 8.2**

---

### Property 2: Rechazo de formulario con campos vacíos (Cliente)

*Para cualquier* objeto de datos de cliente que tenga al menos un campo obligatorio vacío o con solo espacios en blanco, `validators.validateCliente(data)` debe retornar `{ valid: false }` y `store.saveCliente` no debe ser invocado.

**Validates: Requirements 3.3**

---

### Property 3: Unicidad de identificación de Cliente

*Para cualquier* cliente ya almacenado con identificación `X`, intentar guardar un segundo cliente con identificación `X` debe resultar en rechazo (sin modificar el registro existente ni añadir uno duplicado), y `store.getClientes().filter(c => c.id === X)` debe retornar exactamente un elemento.

**Validates: Requirements 3.4**

---

### Property 4: Validación de formato de email

*Para cualquier* cadena de texto que no contenga el patrón `<chars>@<chars>.<chars>` (es decir, sin `@` o sin dominio con al menos un punto), `validators.validateEmail(str)` debe retornar `false`. *Para cualquier* cadena que sí cumpla ese patrón, debe retornar `true`.

**Validates: Requirements 3.5**

---

### Property 5: Round-trip de persistencia de Vehículo

*Para cualquier* objeto `Vehiculo` con campos válidos, guardarlo en el store y luego recuperarlo mediante `store.getVehiculos()` debe producir un array que contenga un objeto con exactamente los mismos valores de campo.

**Validates: Requirements 4.2, 8.1**

---

### Property 6: Rechazo de formulario con campos vacíos (Vehículo)

*Para cualquier* objeto de datos de vehículo con al menos un campo obligatorio vacío, `validators.validateVehiculo(data)` debe retornar `{ valid: false }` y el registro no debe guardarse en el store.

**Validates: Requirements 4.3**

---

### Property 7: Unicidad de placa de Vehículo

*Para cualquier* vehículo ya almacenado con placa `P`, intentar registrar otro vehículo con la misma placa `P` debe ser rechazado, y `store.getVehiculos().filter(v => v.placa === P)` debe retornar exactamente un elemento.

**Validates: Requirements 4.4**

---

### Property 8: Partición de vehículos por tipo

*Para cualquier* colección de vehículos almacenados, `store.getVehiculosPorTipo("Automóvil")` no debe contener ningún vehículo con `tipo === "Motocicleta"`, y `store.getVehiculosPorTipo("Motocicleta")` no debe contener ninguno con `tipo === "Automóvil"`. La unión de ambas particiones debe ser igual al conjunto total de vehículos.

**Validates: Requirements 4.5**

---

### Property 9: Validación de formato de placa

*Para cualquier* cadena con más de 7 caracteres o con caracteres no alfanuméricos, `validators.validatePlaca(str)` debe retornar `false`. *Para cualquier* cadena alfanumérica de 1 a 7 caracteres, debe retornar `true`.

**Validates: Requirements 4.6**

---

### Property 10: Lookup de cliente por documento

*Para cualquier* cliente registrado con documento `D`, `store.lookupClientePorDocumento(D)` debe retornar el objeto `Cliente` correspondiente. *Para cualquier* cadena `D` que no corresponda a ningún cliente registrado, debe retornar `null`.

**Validates: Requirements 5.2, 5.3**

---

### Property 11: Lookup de vehículo por teléfono

*Para cualquier* cliente registrado con teléfono `T` que tenga al menos un vehículo asociado, `store.lookupVehiculoPorTelefono(T)` debe retornar el vehículo correspondiente. *Para cualquier* teléfono `T` sin vehículo asociado, debe retornar `null`.

**Validates: Requirements 5.4, 5.5**

---

### Property 12: Round-trip de persistencia de Cita con estado inicial "pendiente"

*Para cualquier* objeto `Cita` con campos válidos, guardarlo en el store y luego recuperarlo mediante `store.getCitas()` debe producir un array que contenga un objeto con los mismos valores de campo **y** con `estado === "pendiente"`.

**Validates: Requirements 5.6, 8.2**

---

### Property 13: Validación de campos obligatorios y servicios de Cita

*Para cualquier* objeto de datos de cita que tenga al menos un campo obligatorio vacío o cuyo array `servicios` esté vacío, `validators.validateCita(data)` debe retornar `{ valid: false }` con al menos un error reportado.

**Validates: Requirements 5.7**

---

### Property 14: Validación de rango de fechas de Cita

*Para cualquier* par `(fechaRevision, fechaDevolucion)` tal que `fechaDevolucion ≤ fechaRevision`, `validators.validateFechas(fechaRevision, fechaDevolucion)` debe retornar `false`. *Para cualquier* par donde `fechaDevolucion > fechaRevision`, debe retornar `true`.

**Validates: Requirements 5.8**

---

### Property 15: Orden ascendente de citas pendientes

*Para cualquier* colección de citas con `estado === "pendiente"`, el resultado de `store.getCitasPendientes()` debe estar ordenado de modo que para todo índice `i`, `citas[i].fechaRevision ≤ citas[i+1].fechaRevision`.

**Validates: Requirements 5.9**

---

### Property 16: Transición de estado "pendiente" → "atendido" y reclasificación

*Para cualquier* cita con `estado === "pendiente"`, tras invocar `store.updateEstadoCita(citaId, "atendido")`, la cita debe: (a) no aparecer en `store.getCitasPendientes()`, (b) aparecer en `store.getCitasAtendidas()` con `estado === "atendido"`, y (c) persistir el nuevo estado en `localStorage` de forma que al releer la clave se obtenga `estado === "atendido"`.

**Validates: Requirements 6.2, 6.4, 6.5, 8.3**

---

### Property 17: Render de botón "Marcar como atendido" solo en pendientes

*Para cualquier* cita con `estado === "pendiente"`, `ui.renderCitas([cita])` debe producir HTML que contenga un elemento con el atributo `data-cita-id` igual a `citaId` y un botón con texto "Marcar como atendido". *Para cualquier* cita con `estado === "atendido"`, dicho botón no debe aparecer.

**Validates: Requirements 6.1**

---

### Property 18: Orden descendente del historial

*Para cualquier* colección de citas con `estado === "atendido"`, el resultado de `store.getCitasAtendidas()` debe estar ordenado de modo que para todo índice `i`, `citas[i].fechaRevision ≥ citas[i+1].fechaRevision`.

**Validates: Requirements 7.1**

---

### Property 19: Completitud del render de ítem de historial

*Para cualquier* objeto `Cita` con `estado === "atendido"`, `ui.renderHistorialItem(cita)` debe producir una cadena HTML que contenga la placa, marca, modelo, tipo de vehículo, tipo de reparación, todos los servicios, la fecha de revisión y la fecha de devolución de esa cita.

**Validates: Requirements 7.2**

---

### Property 20: Exactitud del contador del historial

*Para cualquier* lista de citas atendidas de tamaño `n` (incluyendo `n = 0`), `ui.renderHistorial(citas)` debe actualizar el contador visible de la sección con el valor exacto `n`.

**Validates: Requirements 7.4**

---

### Property 21: Hidratación completa al inicializar la aplicación

*Para cualquier* estado previo almacenado en `localStorage` (conjuntos arbitrarios de clientes, vehículos y citas), tras invocar `initApp()`, `store.getClientes()`, `store.getVehiculos()` y `store.getCitas()` deben retornar arrays con exactamente los mismos objetos que fueron persistidos, y la UI debe renderizarlos en sus secciones correspondientes.

**Validates: Requirements 8.1**

---

### Property 22: Enlace activo del menú refleja la sección visible

*Para cualquier* sección que cruce el umbral de visibilidad del `IntersectionObserver`, el enlace de navegación cuyo `href` apunte a esa sección debe tener la clase `nav--activo`, y todos los demás enlaces del menú no deben tenerla.

**Validates: Requirements 1.6**
