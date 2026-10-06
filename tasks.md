# Implementation Plan: Serviteca ADSO Landing Page

## Overview

Implementación de la SPA estática en HTML5, CSS3 y JavaScript vanilla siguiendo la arquitectura MVC ligera definida en el diseño. Los archivos se crean en `c:\Users\mafer\OneDrive\Documentos\serviteca\` con la estructura: `index.html`, `styles/main.css`, `scripts/store.js`, `scripts/validators.js`, `scripts/ui.js`, `scripts/main.js`. La persistencia usa `localStorage`. Las fases siguen el orden: infraestructura → lógica de negocio → UI → integración.

---

## Tasks

- [ ] 1. Crear estructura de archivos y esqueleto HTML base
  - [ ] 1.1 Crear `index.html` con el esqueleto HTML5 completo
    - Declarar `<!DOCTYPE html>`, `<html lang="es">`, `<head>` con `meta charset`, `meta viewport`, título "Serviteca ADSO" y links a `styles/main.css` y los cuatro scripts (`defer`)
    - Definir el árbol semántico: `<header id="encabezado">` con el SVG del logo y el `<nav id="nav-principal">`, `<main>` con `<section id="clientes">`, `<section id="agenda">`, `<section id="historial">`, y `<footer id="pie">`
    - Agregar los `id` de anclaje y los atributos ARIA mínimos (`role="img"` en SVG, `aria-label`)
    - _Requirements: 1.1, 2.2, 2.3, 9.1, 9.2_

  - [ ] 1.2 Crear `styles/main.css` con el sistema de diseño visual (variables, reset y tipografía)
    - Declarar todas las variables CSS (`--color-bg`, `--color-surface`, `--color-accent`, `--color-accent-dark`, `--color-text`, `--color-text-muted`, `--color-error`, `--color-success`) en `:root`
    - Aplicar reset básico y `box-sizing: border-box`; establecer `font-size: 16px`, `line-height: 1.5` y `font-family: system-ui, sans-serif` en el `<body>`
    - Definir estilos de `h1` y `h2` con `clamp()` y `font-weight: 700`; color neón en `h2`
    - Garantizar contraste ≥ 4.5:1 para todos los pares texto/fondo
    - _Requirements: 2.1, 2.3, 2.4, 2.5_

  - [ ] 1.3 Crear `scripts/store.js` con la estructura de módulo vacío (namespace y stubs)
    - Declarar el objeto `store` global con todos los métodos de la API pública como stubs que lanzan `"Not implemented"`
    - Definir las constantes de claves: `KEYS = { clientes: 'serviteca_clientes', vehiculos: 'serviteca_vehiculos', citas: 'serviteca_citas' }`
    - _Requirements: 8.1, 8.2, 8.3_

  - [ ] 1.4 Crear `scripts/validators.js`, `scripts/ui.js` y `scripts/main.js` como módulos vacíos
    - Cada archivo declara su objeto global (`validators`, `ui`, `main`) con stubs de todos los métodos de la API pública del diseño
    - `main.js` exporta `initApp()` como stub
    - _Requirements: 1.1, 3.1, 4.1, 5.1_

- [ ] 2. Implementar el módulo `store.js` (modelo / persistencia)
  - [ ] 2.1 Implementar helpers privados de lectura/escritura en `localStorage`
    - Escribir `_read(key)`: intenta `JSON.parse(localStorage.getItem(key))`; retorna `[]` si nulo; lanza `StorageError` si falla
    - Escribir `_write(key, data)`: intenta `localStorage.setItem(key, JSON.stringify(data))`; lanza `StorageError` si falla (cuota excedida, modo privado)
    - _Requirements: 8.2, 8.3, 8.4_

  - [ ] 2.2 Implementar la API de Clientes en `store.js`
    - `getClientes()` → `_read(KEYS.clientes)`
    - `saveCliente(cliente)` → lee array, hace push, escribe con `_write`
    - `clienteExiste(id)` → `getClientes().some(c => c.id === id)`
    - `lookupClientePorDocumento(doc)` → `getClientes().find(c => c.id === doc) ?? null`
    - _Requirements: 3.2, 3.4, 5.2, 5.3, 8.1, 8.2_

  - [ ]* 2.3 Escribir property test para round-trip de Cliente (Property 1)
    - **Property 1: Round-trip de persistencia de Cliente**
    - Para cualquier objeto `Cliente` válido, `saveCliente` + `getClientes()` debe retornar un array que incluya ese objeto con los mismos valores
    - **Validates: Requirements 3.2, 8.1, 8.2**

  - [ ]* 2.4 Escribir property test para unicidad de identificación de Cliente (Property 3)
    - **Property 3: Unicidad de identificación de Cliente**
    - Dado un cliente ya guardado con id `X`, intentar guardar un segundo cliente con id `X` debe resultar en rechazo; `getClientes().filter(c => c.id === X)` retorna exactamente un elemento
    - **Validates: Requirements 3.4**

  - [ ]* 2.5 Escribir property test para lookup de cliente por documento (Property 10)
    - **Property 10: Lookup de cliente por documento**
    - Para cualquier `D` registrado, `lookupClientePorDocumento(D)` retorna el cliente; para `D` no registrado, retorna `null`
    - **Validates: Requirements 5.2, 5.3**

  - [ ] 2.6 Implementar la API de Vehículos en `store.js`
    - `getVehiculos()`, `saveVehiculo(v)`, `placaExiste(placa)`, `getVehiculosPorTipo(tipo)`, `lookupVehiculoPorTelefono(tel)`
    - `lookupVehiculoPorTelefono(tel)`: busca en `getClientes()` el cliente cuyo `telefono === tel`; si lo encuentra, retorna el primer vehículo con `clienteId === cliente.id`; de lo contrario `null`
    - _Requirements: 4.2, 4.4, 4.5, 5.4, 5.5, 8.1_

  - [ ]* 2.7 Escribir property test para round-trip de Vehículo (Property 5)
    - **Property 5: Round-trip de persistencia de Vehículo**
    - Para cualquier objeto `Vehiculo` válido, `saveVehiculo` + `getVehiculos()` contiene el objeto con los mismos valores
    - **Validates: Requirements 4.2, 8.1**

  - [ ]* 2.8 Escribir property test para unicidad de placa (Property 7)
    - **Property 7: Unicidad de placa de Vehículo**
    - Para cualquier placa `P` ya almacenada, registrar otro vehículo con la misma placa debe ser rechazado; `getVehiculos().filter(v => v.placa === P)` retorna exactamente un elemento
    - **Validates: Requirements 4.4**

  - [ ]* 2.9 Escribir property test para partición de vehículos por tipo (Property 8)
    - **Property 8: Partición de vehículos por tipo**
    - `getVehiculosPorTipo("Automóvil")` no contiene motocicletas; `getVehiculosPorTipo("Motocicleta")` no contiene automóviles; la unión de ambas particiones iguala el total
    - **Validates: Requirements 4.5**

  - [ ]* 2.10 Escribir property test para lookup de vehículo por teléfono (Property 11)
    - **Property 11: Lookup de vehículo por teléfono**
    - Para teléfono `T` con vehículo asociado, `lookupVehiculoPorTelefono(T)` retorna el vehículo; para `T` sin vehículo, retorna `null`
    - **Validates: Requirements 5.4, 5.5**

  - [ ] 2.11 Implementar la API de Citas en `store.js`
    - `getCitas()`, `saveCita(cita)`, `updateEstadoCita(citaId, estado)`
    - `getCitasPendientes()`: filtra `estado === "pendiente"`, ordena ASC por `fechaRevision`
    - `getCitasAtendidas()`: filtra `estado === "atendido"`, ordena DESC por `fechaRevision`
    - `saveCita` genera `citaId` con `crypto.randomUUID()` si no viene provisto; inicializa `estado = "pendiente"`
    - _Requirements: 5.6, 5.9, 6.2, 7.1, 8.2, 8.3_

  - [ ]* 2.12 Escribir property test para round-trip de Cita con estado inicial "pendiente" (Property 12)
    - **Property 12: Round-trip de persistencia de Cita con estado inicial "pendiente"**
    - Para cualquier `Cita` válida, `saveCita` + `getCitas()` contiene el objeto con los mismos valores y `estado === "pendiente"`
    - **Validates: Requirements 5.6, 8.2**

  - [ ]* 2.13 Escribir property test para orden ascendente de citas pendientes (Property 15)
    - **Property 15: Orden ascendente de citas pendientes**
    - Para cualquier colección de citas pendientes, `getCitasPendientes()[i].fechaRevision <= getCitasPendientes()[i+1].fechaRevision` para todo `i`
    - **Validates: Requirements 5.9**

  - [ ]* 2.14 Escribir property test para transición de estado pendiente → atendido (Property 16)
    - **Property 16: Transición de estado "pendiente" → "atendido" y reclasificación**
    - Tras `updateEstadoCita(id, "atendido")`: (a) no aparece en `getCitasPendientes()`; (b) aparece en `getCitasAtendidas()` con `estado === "atendido"`; (c) persiste en `localStorage`
    - **Validates: Requirements 6.2, 6.4, 6.5, 8.3**

  - [ ]* 2.15 Escribir property test para orden descendente del historial (Property 18)
    - **Property 18: Orden descendente del historial**
    - Para cualquier colección de citas atendidas, `getCitasAtendidas()[i].fechaRevision >= getCitasAtendidas()[i+1].fechaRevision` para todo `i`
    - **Validates: Requirements 7.1**

- [ ] 3. Checkpoint — verificar módulo store
  - Ejecutar todos los property tests del paso 2 y confirmar que pasan antes de continuar.
  - Asegurar que todas las operaciones CRUD leen y escriben correctamente en `localStorage`.

- [ ] 4. Implementar el módulo `validators.js`
  - [ ] 4.1 Implementar `validateEmail(email)` y `validatePlaca(placa)`
    - `validateEmail`: retorna `true` si la cadena contiene `@` seguido de un dominio con al menos un punto
    - `validatePlaca`: retorna `true` si la cadena tiene entre 1 y 7 caracteres alfanuméricos (`/^[A-Za-z0-9]{1,7}$/`)
    - _Requirements: 3.5, 4.6_

  - [ ]* 4.2 Escribir property test para validación de formato de email (Property 4)
    - **Property 4: Validación de formato de email**
    - Para cualquier cadena sin `@` o sin dominio con punto, `validateEmail` retorna `false`; para cadenas con el patrón `<chars>@<chars>.<chars>`, retorna `true`
    - **Validates: Requirements 3.5**

  - [ ]* 4.3 Escribir property test para validación de formato de placa (Property 9)
    - **Property 9: Validación de formato de placa**
    - Para cualquier cadena con más de 7 caracteres o con caracteres no alfanuméricos, `validatePlaca` retorna `false`; para alfanuméricas de 1-7 chars, retorna `true`
    - **Validates: Requirements 4.6**

  - [ ] 4.4 Implementar `validateFechas(fechaRevision, fechaDevolucion)`
    - Retorna `true` solo si `new Date(fechaDevolucion) > new Date(fechaRevision)`
    - _Requirements: 5.8_

  - [ ]* 4.5 Escribir property test para validación de rango de fechas (Property 14)
    - **Property 14: Validación de rango de fechas de Cita**
    - Para cualquier par donde `fechaDevolucion <= fechaRevision`, `validateFechas` retorna `false`; para `fechaDevolucion > fechaRevision`, retorna `true`
    - **Validates: Requirements 5.8**

  - [ ] 4.6 Implementar `validateCliente(data)`
    - Valida presencia y longitudes de todos los campos obligatorios (`id` ≤ 20, `telefono` ≤ 15, `email` ≤ 100, `nombres` ≤ 100, `apellidos` ≤ 100)
    - Valida `email` con `validateEmail`
    - Retorna `{ valid: boolean, errors: [{ campo, mensaje }] }`
    - _Requirements: 3.1, 3.3, 3.5_

  - [ ]* 4.7 Escribir property test para rechazo de formulario con campos vacíos — Cliente (Property 2)
    - **Property 2: Rechazo de formulario con campos vacíos (Cliente)**
    - Para cualquier objeto de datos de cliente con al menos un campo obligatorio vacío o solo espacios, `validateCliente` retorna `{ valid: false }`
    - **Validates: Requirements 3.3**

  - [ ] 4.8 Implementar `validateVehiculo(data)` y `validateCita(data)`
    - `validateVehiculo`: valida `placa` con `validatePlaca`, presencia de `marca` ≤ 50, `modelo` ≤ 50, `tipo` en `["Automóvil","Motocicleta"]`
    - `validateCita`: valida todos los campos obligatorios, que `servicios` sea un array no vacío y que las fechas sean válidas con `validateFechas`
    - _Requirements: 4.1, 4.3, 4.6, 5.1, 5.7, 5.8_

  - [ ]* 4.9 Escribir property test para rechazo de formulario con campos vacíos — Vehículo (Property 6)
    - **Property 6: Rechazo de formulario con campos vacíos (Vehículo)**
    - Para cualquier objeto de datos de vehículo con al menos un campo obligatorio vacío, `validateVehiculo` retorna `{ valid: false }`
    - **Validates: Requirements 4.3**

  - [ ]* 4.10 Escribir property test para validación de campos obligatorios de Cita (Property 13)
    - **Property 13: Validación de campos obligatorios y servicios de Cita**
    - Para cualquier objeto con al menos un campo obligatorio vacío o `servicios` vacío, `validateCita` retorna `{ valid: false }` con al menos un error reportado
    - **Validates: Requirements 5.7**

- [ ] 5. Checkpoint — verificar módulo validators
  - Ejecutar todos los property tests del paso 4 y confirmar que pasan.
  - Verificar manualmente casos borde: email sin punto en dominio, placa con 8 caracteres, fechas iguales.

- [ ] 6. Implementar el módulo `ui.js` (renderizado y mensajes)
  - [ ] 6.1 Implementar `ui.showError(campo, mensaje)`, `ui.clearErrors(formId)` y `ui.showBanner(tipo, mensaje)`
    - `showError`: inserta un `<span class="field-error">` debajo del `<input>` o `<select>` con el selector `[name="${campo}"]` dentro del formulario; aplica `border-color: var(--color-error)` al campo
    - `clearErrors`: elimina todos los `.field-error` del formulario y restaura estilos de campos
    - `showBanner`: inserta un banner `role="alert"` con clase `banner--error` o `banner--success` en la sección indicada; se auto-elimina a los 5 segundos
    - _Requirements: 3.3, 3.4, 3.5, 4.3, 4.4, 4.6, 5.7, 5.8, 6.3, 8.4_

  - [ ] 6.2 Implementar `ui.renderClientes(clientes, vehiculos)`
    - Genera la lista de clientes registrados en `#lista-clientes`
    - Para cada cliente, muestra: id, nombres, apellidos, teléfono, email
    - Dentro de `#lista-vehiculos`, genera dos subsecciones `<div class="subseccion">` con `<h3>Automóviles</h3>` y `<h3>Motocicletas</h3>`; filtra `vehiculos` por `tipo` para cada subsección
    - Si no hay vehículos en una categoría, muestra "No hay vehículos registrados en esta categoría"
    - _Requirements: 4.5, 4.7_

  - [ ] 6.3 Implementar `ui.renderCitas(citas)`
    - Renderiza en `#lista-citas` las citas recibidas (ya ordenadas ASC) con todos los campos del requisito 5.9
    - Cada ítem incluye un `<button data-cita-id="${cita.citaId}">Marcar como atendido</button>` para citas con `estado === "pendiente"`
    - _Requirements: 5.9, 6.1_

  - [ ]* 6.4 Escribir property test para render del botón "Marcar como atendido" (Property 17)
    - **Property 17: Render de botón "Marcar como atendido" solo en pendientes**
    - Para cualquier cita con `estado === "pendiente"`, `renderCitas([cita])` produce HTML con el botón; para `estado === "atendido"`, el botón no aparece
    - **Validates: Requirements 6.1**

  - [ ] 6.5 Implementar `ui.renderHistorial(citas)` y `ui.renderHistorialItem(cita)`
    - `renderHistorialItem(cita)`: retorna una cadena HTML con placa, marca, modelo, tipo de vehículo, tipo de reparación, todos los servicios, fecha de revisión y fecha de devolución
    - `renderHistorial(citas)`: renderiza el listado en `#lista-historial`; actualiza el contador `#contador-historial` con `citas.length`; si `citas.length === 0` muestra "No hay vehículos atendidos aún"
    - _Requirements: 7.1, 7.2, 7.3, 7.4_

  - [ ]* 6.6 Escribir property test para completitud del render de ítem de historial (Property 19)
    - **Property 19: Completitud del render de ítem de historial**
    - Para cualquier `Cita` con `estado === "atendido"`, `renderHistorialItem(cita)` contiene placa, marca, modelo, tipo de vehículo, tipo de reparación, todos los servicios, fecha de revisión y fecha de devolución
    - **Validates: Requirements 7.2**

  - [ ]* 6.7 Escribir property test para exactitud del contador del historial (Property 20)
    - **Property 20: Exactitud del contador del historial**
    - Para cualquier lista de citas atendidas de tamaño `n` (incluyendo `n = 0`), `renderHistorial(citas)` actualiza el contador con el valor exacto `n`
    - **Validates: Requirements 7.4**

  - [ ] 6.8 Implementar `ui.setNavActivo(sectionId)` y `ui.showLookupResult(campo, texto)`
    - `setNavActivo`: añade clase `nav--activo` al enlace cuyo `href === "#${sectionId}"`; quita la clase de todos los demás
    - `showLookupResult(campo, texto)`: inserta o actualiza un `<span class="lookup-result">` debajo del campo indicado; si `texto` es `null` elimina el span
    - _Requirements: 1.6, 5.2, 5.3, 5.4, 5.5_

  - [ ]* 6.9 Escribir property test para enlace activo del menú (Property 22)
    - **Property 22: Enlace activo del menú refleja la sección visible**
    - Para cualquier `sectionId` válido, `setNavActivo(sectionId)` hace que exactamente el enlace con `href="#sectionId"` tenga `nav--activo` y todos los demás no lo tengan
    - **Validates: Requirements 1.6**

- [ ] 7. Checkpoint — verificar módulo ui
  - Ejecutar los property tests de los pasos 6.4, 6.6, 6.7 y 6.9.
  - Abrir `index.html` en el navegador y verificar manualmente que las listas vacías muestran los mensajes correctos.

- [ ] 8. Implementar estilos CSS completos en `styles/main.css`
  - [ ] 8.1 Implementar layout del encabezado y navegación fija
    - Encabezado con `position: sticky; top: 0; z-index: 100`; `display: flex; align-items: center; gap: 1rem`
    - Logo SVG con tamaño mínimo 48 × 48 px; texto "Serviteca ADSO" `font-size: clamp(1.5rem, 4vw, 2.5rem); font-weight: 700`
    - Menú `<nav>` con `display: flex; gap: 1rem`; estilos para `.nav--activo` (subrayado o fondo neón)
    - Menú hamburger (`<button id="nav-toggle">`) visible solo en ≤ 599 px; `<ul>` colapsable con `display: none` / `display: flex` mediante clase `.nav--abierto`
    - _Requirements: 1.4, 1.6, 2.1, 2.2, 2.3_

  - [ ] 8.2 Implementar grid responsivo para secciones, formularios y listas
    - Secciones con `padding: 2rem max(1rem, 5vw)` y fondo alternado (`--color-bg` / `--color-surface`)
    - Formularios con `display: grid; gap: 1rem`; en ≥ 600 px `grid-template-columns: repeat(2, 1fr)`
    - Listas de clientes y vehículos con `display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`
    - Botones CTA con `background: var(--color-accent); color: #0a0a0a; font-weight: 700; border-radius: 4px; padding: .75rem 1.5rem`; hover a `--color-accent-dark`
    - _Requirements: 1.2, 2.1, 2.5_

  - [ ] 8.3 Implementar estilos de estados: error, éxito, lookup y campo inválido
    - `.field-error { color: var(--color-error); font-size: .875rem; margin-top: .25rem }`
    - `input.invalid, select.invalid { border-color: var(--color-error) }`
    - `.banner--error` y `.banner--success` con padding, border-radius y los colores correspondientes
    - `.lookup-result { font-size: .875rem; color: var(--color-text-muted); margin-top: .25rem }`
    - _Requirements: 3.3, 4.3, 5.7_

  - [ ] 8.4 Implementar estilos del pie de página
    - `<footer>` con `background: var(--color-surface); text-align: center; padding: 2rem`
    - Texto "Serviteca ADSO" con contraste ≥ 4.5:1 (usar `--color-text` sobre `--color-surface`)
    - _Requirements: 9.1, 9.2, 9.3_

- [ ] 9. Implementar el módulo `main.js` (controlador e integración)
  - [ ] 9.1 Implementar `initApp()`: hidratación desde `localStorage` al arrancar
    - Envolver toda la carga en `try/catch`; si falla, mostrar banner global de error y dejar secciones vacías
    - Llamar a `store.getClientes()`, `store.getVehiculos()`, `store.getCitasPendientes()`, `store.getCitasAtendidas()` y pasar los datos a los métodos `ui.render*` correspondientes
    - _Requirements: 8.1, 8.4_

  - [ ]* 9.2 Escribir property test para hidratación completa al inicializar (Property 21)
    - **Property 21: Hidratación completa al inicializar la aplicación**
    - Para cualquier estado previo en `localStorage`, tras `initApp()`, `store.getClientes()`, `store.getVehiculos()` y `store.getCitas()` retornan los mismos objetos que fueron persistidos y la UI los renderiza
    - **Validates: Requirements 8.1**

  - [ ] 9.3 Implementar el manejador `handleSubmitCliente(event)`
    - Leer valores del `#form-cliente`; llamar a `validators.validateCliente(data)`
    - Si inválido: `ui.showError` por cada error y abortar
    - Verificar `store.clienteExiste(id)`; si existe: mostrar mensaje de duplicado y abortar
    - Guardar con `store.saveCliente(cliente)` en `try/catch`; si falla, banner de error conservando valores del formulario
    - En éxito: `ui.renderClientes(store.getClientes(), store.getVehiculos())`, limpiar formulario
    - _Requirements: 3.2, 3.3, 3.4, 3.5, 3.6, 8.2_

  - [ ] 9.4 Implementar el manejador `handleSubmitVehiculo(event)`
    - Leer valores del `#form-vehiculo`; llamar a `validators.validateVehiculo(data)`
    - Verificar `store.placaExiste(placa)`; si existe: mostrar "La placa ya está registrada"
    - Guardar con `store.saveVehiculo(vehiculo)` en `try/catch`
    - En éxito: `ui.renderClientes(store.getClientes(), store.getVehiculos())`, limpiar formulario
    - _Requirements: 4.2, 4.3, 4.4, 4.6, 8.2_

  - [ ] 9.5 Implementar los lookups en tiempo real del `#form-cita` (input events)
    - Al evento `input` en `#cita-documento`: llamar a `store.lookupClientePorDocumento(val)`; mostrar nombre o "Cliente no encontrado" con `ui.showLookupResult`
    - Al evento `input` en `#cita-telefono`: llamar a `store.lookupVehiculoPorTelefono(val)`; mostrar marca + modelo + placa o "Vehículo no encontrado"
    - Ambas respuestas deben reflejarse en ≤ 1 segundo (operación síncrona)
    - _Requirements: 5.2, 5.3, 5.4, 5.5_

  - [ ] 9.6 Implementar el manejador `handleSubmitCita(event)`
    - Leer todos los campos del `#form-cita`, incluyendo los checkboxes de servicios
    - Llamar a `validators.validateCita(data)`; si inválido, mostrar errores por campo
    - Guardar con `store.saveCita(cita)` en `try/catch`
    - En éxito: `ui.renderCitas(store.getCitasPendientes())`, mostrar banner de confirmación, limpiar formulario
    - _Requirements: 5.6, 5.7, 5.8, 8.2_

  - [ ] 9.7 Implementar el manejador `handleMarcarAtendido(citaId)` (delegación de eventos en `#lista-citas`)
    - Escuchar `click` en `#lista-citas` con `event.target.closest('[data-cita-id]')`
    - Llamar `store.updateEstadoCita(citaId, "atendido")` en `try/catch`; si falla, mostrar error inline en la cita y mantener `estado = "pendiente"`
    - En éxito: `ui.renderCitas(store.getCitasPendientes())` y `ui.renderHistorial(store.getCitasAtendidas())`
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 8.3_

  - [ ] 9.8 Implementar el scroll suave y el `IntersectionObserver` para el menú activo
    - Registrar `click` en todos los `<a>` del `<nav>` para `scrollIntoView({ behavior: 'smooth' })`; asegurar que la transición complete en ≤ 1 segundo (nativo del navegador)
    - Crear `IntersectionObserver` con `threshold: 0.4` que llame a `ui.setNavActivo(entry.target.id)` cuando `entry.isIntersecting`
    - Implementar el toggle del menú hamburger para pantallas ≤ 599 px
    - _Requirements: 1.4, 1.5, 1.6_

- [ ] 10. Checkpoint — verificar integración completa
  - Ejecutar todos los property tests restantes (pasos 9.2 y cualquier pendiente).
  - Abrir `index.html` localmente; recorrer el flujo completo: registrar cliente → registrar vehículo → agendar cita → marcar como atendido → verificar historial.
  - Verificar que al recargar la página los datos persisten (localStorage).
  - Comprobar responsividad en ≥ 3 anchos de ventana (320 px, 768 px, 1280 px).

- [ ] 11. Crear estructura HTML completa de formularios e IDs en `index.html`
  - [ ] 11.1 Completar el HTML de `#clientes`: formularios con todos los campos del diseño
    - `#form-cliente` con los cinco campos (`id`, `telefono`, `email`, `nombres`, `apellidos`), atributos `maxlength`, `type`, `required` y `name` alineados con los parámetros que `main.js` leerá
    - `#form-vehiculo` con los cuatro campos (`placa`, `marca`, `modelo`, `tipo`), con el `<select>` de tipo de vehículo
    - Contenedores `#lista-clientes`, `#lista-vehiculos`
    - _Requirements: 3.1, 4.1, 4.5_

  - [ ] 11.2 Completar el HTML de `#agenda`: formulario de citas y lista
    - `#form-cita` con todos los campos del requisito 5.1: documento, teléfono, tipo de vehículo, tipo de reparación, `datetime-local` de revisión y devolución, cuatro checkboxes de servicios
    - Spans de lookup bajo los campos de documento y teléfono
    - Contenedor `#lista-citas`
    - _Requirements: 5.1_

  - [ ] 11.3 Completar el HTML de `#historial`: encabezado con contador y lista
    - `<h2>` con `<span id="contador-historial">0</span>` vehículos atendidos
    - Contenedor `#lista-historial`
    - _Requirements: 7.3, 7.4_

- [ ] 12. Checkpoint final — asegurar que todas las pruebas pasen
  - Ejecutar la suite completa de property tests; ninguno debe fallar.
  - Verificar en el navegador que no hay scroll horizontal en ningún ancho entre 320 px y 1920 px.
  - Confirmar contraste de todos los pares texto/fondo con una herramienta de accesibilidad.
  - Asegurar que todos los tests pasen; consultar al usuario si surge alguna duda.

---

## Notes

- Las tareas marcadas con `*` son opcionales y pueden omitirse para una entrega MVP más rápida
- Los property tests se pueden implementar con un framework minimalista en un archivo `scripts/tests.js` o inline en la consola del navegador; no requieren dependencias externas
- Cada tarea referencia los requisitos específicos para trazabilidad completa
- Los checkpoints garantizan validación incremental antes de avanzar a la siguiente fase
- Las propiedades 1–22 cubren todos los módulos: store (props 1, 3, 5, 7, 8, 10, 11, 12, 15, 16, 18, 21), validators (props 2, 4, 6, 9, 13, 14), ui (props 17, 19, 20, 22)

---

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2", "1.3", "1.4"] },
    { "id": 1, "tasks": ["2.1"] },
    { "id": 2, "tasks": ["2.2", "2.6", "2.11"] },
    { "id": 3, "tasks": ["2.3", "2.4", "2.5", "2.7", "2.8", "2.9", "2.10", "2.12", "2.13", "2.14", "2.15"] },
    { "id": 4, "tasks": ["4.1", "4.4"] },
    { "id": 5, "tasks": ["4.2", "4.3", "4.5", "4.6", "4.8"] },
    { "id": 6, "tasks": ["4.7", "4.9", "4.10"] },
    { "id": 7, "tasks": ["6.1", "6.2", "6.3", "6.5", "6.8"] },
    { "id": 8, "tasks": ["6.4", "6.6", "6.7", "6.9"] },
    { "id": 9, "tasks": ["8.1", "8.2", "8.3", "8.4"] },
    { "id": 10, "tasks": ["11.1", "11.2", "11.3"] },
    { "id": 11, "tasks": ["9.1", "9.3", "9.4", "9.5", "9.6", "9.7", "9.8"] },
    { "id": 12, "tasks": ["9.2"] }
  ]
}
```
