# Requirements Document

## Introduction

La Serviteca ADSO necesita una landing page de una sola página (single-page application) desarrollada en HTML, CSS y JavaScript puro, sin frameworks externos. La página permitirá registrar clientes y vehículos, agendar citas de servicio, consultar el historial de vehículos atendidos, y presentar la identidad visual del negocio. El diseño será responsivo, de aspecto profesional, con color neón amarillo como color principal y un logo de automóvil representado mediante SVG o emoji.

---

## Glossary

- **Landing_Page**: Página web de una sola vista que agrupa todas las secciones funcionales de la Serviteca ADSO.
- **Cliente**: Persona registrada en el sistema con número de identificación, teléfono, correo electrónico, nombres y apellidos.
- **Vehículo**: Automóvil o motocicleta registrado con número de placa, marca, modelo y tipo de vehículo.
- **Cita**: Registro de una solicitud de servicio agendado para un cliente y vehículo, con fecha y hora de revisión, fecha y hora de devolución, tipo de reparación y servicios solicitados.
- **Servicio**: Tarea específica realizada al vehículo; puede ser cambio de aceite, alineación, sincronización o lavado.
- **Estado_de_Servicio**: Indicador del estado actual de una cita; puede ser "pendiente" o "atendido".
- **Historial**: Listado de citas con Estado_de_Servicio igual a "atendido".
- **Formulario_Cliente**: Componente UI que captura los datos del Cliente.
- **Formulario_Cita**: Componente UI que captura los datos de una Cita.
- **Sección_Clientes**: Área de la Landing_Page que gestiona el registro de Clientes y sus Vehículos.
- **Sección_Agenda**: Área de la Landing_Page que gestiona el agendamiento de Citas.
- **Sección_Historial**: Área de la Landing_Page que muestra el Historial de vehículos atendidos.
- **Pie_de_Página**: Componente inferior de la Landing_Page que identifica a la Serviteca ADSO.
- **Almacenamiento_Local**: Mecanismo de persistencia usando `localStorage` del navegador.
- **Número_de_Placa**: Identificador único alfanumérico del Vehículo.

---

## Requirements

### Requisito 1: Estructura general de la página

**User Story:** Como visitante, quiero una página única bien estructurada, para navegar fácilmente entre las secciones sin recargar el sitio.

#### Criterios de Aceptación

1. THE Landing_Page SHALL contener, en orden: encabezado con logo y nombre, Sección_Clientes, Sección_Agenda, Sección_Historial y Pie_de_Página.
2. THE Landing_Page SHALL ser responsiva y adaptarse a anchos de pantalla desde 320px hasta 1920px sin que ningún elemento de texto o imagen quede recortado ni aparezca barra de desplazamiento horizontal.
3. THE Landing_Page SHALL implementarse en un único archivo HTML con estilos CSS y lógica JavaScript embebidos o en archivos del mismo directorio, sin dependencias de frameworks externos.
4. WHILE el usuario desplaza la página, THE Landing_Page SHALL mantener el menú de navegación visible y fijo en la parte superior de la ventana con enlaces a cada sección mediante anclas.
5. WHEN el usuario hace clic en un enlace del menú de navegación, THE Landing_Page SHALL desplazarse hasta la sección correspondiente mediante una transición animada, sin salto instantáneo, completada en no más de 1 segundo.
6. WHEN una sección es visible en el área de visualización del usuario, THE Landing_Page SHALL resaltar visualmente el enlace correspondiente en el menú de navegación, distinguiéndolo de los demás enlaces.

---

### Requisito 2: Identidad visual y diseño

**User Story:** Como dueño del negocio, quiero que la página refleje la identidad de la Serviteca ADSO, para transmitir profesionalismo a los clientes.

#### Criterios de Aceptación

1. THE Landing_Page SHALL utilizar el color neón amarillo como color principal de énfasis, aplicado en todos los encabezados de sección, botones de llamada a la acción y elementos destacados de la página.
2. THE Landing_Page SHALL mostrar en el encabezado un logo de automóvil de frente representado mediante SVG inline o un emoji de automóvil (`🚗`) de tamaño mínimo 48px.
3. THE Landing_Page SHALL mostrar el nombre "Serviteca ADSO" en el encabezado con tipografía en negrita de tamaño mínimo 24px.
4. THE Landing_Page SHALL usar un esquema de color único y consistente en todas las secciones que garantice una relación de contraste mínima de 4.5:1 entre el texto y el fondo, conforme a WCAG 2.1 nivel AA.
5. THE Landing_Page SHALL mostrar un tamaño de fuente base mínimo de 16px, un interlineado mínimo de 1.5 y márgenes laterales mínimos de 16px en todas las secciones.

---

### Requisito 3: Registro de clientes

**User Story:** Como empleado de la serviteca, quiero registrar clientes con sus datos de contacto, para mantener un directorio actualizado.

#### Criterios de Aceptación

1. THE Sección_Clientes SHALL mostrar el Formulario_Cliente con los campos: número de identificación (texto, máximo 20 caracteres, obligatorio), número de teléfono (texto, máximo 15 caracteres, obligatorio), correo electrónico (email, máximo 100 caracteres, obligatorio), nombres (texto, máximo 100 caracteres, obligatorio) y apellidos (texto, máximo 100 caracteres, obligatorio).
2. WHEN el empleado envía el Formulario_Cliente con todos los campos válidos, THE Sección_Clientes SHALL guardar el registro del Cliente en el Almacenamiento_Local y mostrar el cliente en la lista de clientes registrados en menos de 1 segundo.
3. IF el empleado envía el Formulario_Cliente con uno o más campos vacíos, THEN THE Formulario_Cliente SHALL resaltar visualmente cada campo vacío obligatorio y mostrar un mensaje de error indicando los campos requeridos faltantes sin guardar el registro.
4. IF el empleado envía el Formulario_Cliente con un número de identificación ya registrado en el Almacenamiento_Local, THEN THE Formulario_Cliente SHALL mostrar el mensaje "El número de identificación ya está registrado" sin guardar el registro duplicado ni modificar el registro existente.
5. IF el empleado envía el Formulario_Cliente con un valor de correo electrónico que no contenga "@" seguido de un dominio con al menos un punto, THEN THE Formulario_Cliente SHALL mostrar un mensaje de error indicando formato de correo inválido sin guardar el registro.
6. IF el Almacenamiento_Local no está disponible al intentar guardar el Formulario_Cliente, THEN THE Sección_Clientes SHALL mostrar un mensaje de error indicando que no fue posible guardar el registro sin perder los datos ingresados en el formulario.

---

### Requisito 4: Registro de vehículos

**User Story:** Como empleado de la serviteca, quiero registrar los vehículos asociados a un cliente, para gestionar los servicios por vehículo.

#### Criterios de Aceptación

1. THE Sección_Clientes SHALL mostrar un formulario de registro de Vehículo con los campos: número de placa (texto, máximo 7 caracteres alfanuméricos, obligatorio), marca (texto, máximo 50 caracteres, obligatorio), modelo (texto, máximo 50 caracteres, obligatorio) y tipo de vehículo (selección: "Automóvil" o "Motocicleta", obligatorio).
2. WHEN el empleado envía el formulario de Vehículo con todos los campos válidos, THE Sección_Clientes SHALL guardar el Vehículo en el Almacenamiento_Local y mostrarlo en la subsección correspondiente al tipo de vehículo registrado dentro de la lista de vehículos registrados.
3. IF el empleado envía el formulario de Vehículo con uno o más campos vacíos, THEN THE Sección_Clientes SHALL mostrar un mensaje de error indicando cuáles campos obligatorios están vacíos, no guardar el registro y mantener los valores ingresados en los campos restantes del formulario.
4. IF el empleado envía el formulario de Vehículo con un Número_de_Placa ya registrado en el Almacenamiento_Local, THEN THE Sección_Clientes SHALL mostrar el mensaje "La placa ya está registrada" y no duplicar el registro.
5. THE Sección_Clientes SHALL mostrar las listas de automóviles y motocicletas en subsecciones separadas, identificadas con los encabezados "Automóviles" y "Motocicletas" respectivamente.
6. IF el empleado ingresa un Número_de_Placa con más de 7 caracteres o caracteres no alfanuméricos, THEN THE Sección_Clientes SHALL mostrar un mensaje de error indicando que el formato de placa es inválido y no guardar el registro.
7. WHEN el Almacenamiento_Local no contiene vehículos registrados para un tipo de vehículo, THE Sección_Clientes SHALL mostrar la subsección correspondiente con un mensaje indicando que no hay vehículos registrados en esa categoría.

---

### Requisito 5: Agendamiento de citas

**User Story:** Como empleado de la serviteca, quiero agendar citas para nuevos y existentes clientes, para organizar el flujo de trabajo diario.

#### Criterios de Aceptación

1. THE Sección_Agenda SHALL mostrar el Formulario_Cita con los campos: número de documento del cliente (texto, máximo 15 dígitos, obligatorio), número de teléfono del cliente (texto, máximo 15 dígitos, obligatorio), tipo de vehículo (selección: "Automóvil" o "Motocicleta", obligatorio), tipo de reparación (texto, máximo 100 caracteres, obligatorio), fecha y hora de revisión (datetime-local, obligatorio), fecha y hora de devolución (datetime-local, obligatorio), y servicios solicitados (casillas de verificación: "Cambio de aceite", "Alineación", "Sincronización", "Lavado"; al menos una obligatoria).
2. WHEN el empleado ingresa un número de documento en el Formulario_Cita y el documento coincide con un Cliente registrado, THE Formulario_Cita SHALL mostrar automáticamente el nombre del cliente debajo del campo de documento en menos de 1 segundo.
3. IF el número de documento ingresado en el Formulario_Cita no coincide con ningún Cliente registrado, THEN THE Formulario_Cita SHALL mostrar el mensaje "Cliente no encontrado" debajo del campo de documento.
4. WHEN el empleado ingresa un número de teléfono en el Formulario_Cita y el teléfono coincide con un Vehículo registrado, THE Formulario_Cita SHALL mostrar automáticamente la marca, modelo y Número_de_Placa del Vehículo asociado en menos de 1 segundo.
5. IF el número de teléfono ingresado en el Formulario_Cita no coincide con ningún Vehículo registrado, THEN THE Formulario_Cita SHALL mostrar el mensaje "Vehículo no encontrado" debajo del campo de teléfono.
6. WHEN el empleado envía el Formulario_Cita con todos los campos válidos, THE Sección_Agenda SHALL guardar la Cita en el Almacenamiento_Local con Estado_de_Servicio igual a "pendiente", mostrarla en la lista de citas agendadas y mostrar una confirmación de éxito.
7. IF el empleado envía el Formulario_Cita con uno o más campos obligatorios vacíos o sin ninguna casilla de servicio seleccionada, THEN THE Formulario_Cita SHALL mostrar un mensaje de error indicando los campos faltantes y no guardar la Cita.
8. IF la fecha y hora de devolución ingresada en el Formulario_Cita es anterior o igual a la fecha y hora de revisión, THEN THE Formulario_Cita SHALL mostrar el mensaje "La fecha de devolución debe ser posterior a la fecha de revisión" y no guardar la Cita.
9. THE Sección_Agenda SHALL mostrar las citas agendadas en una lista ordenada ascendentemente por fecha y hora de revisión, que incluya: nombre del cliente, Número_de_Placa, marca, modelo, tipo de reparación, servicios solicitados, fecha y hora de revisión, fecha y hora de devolución y Estado_de_Servicio.

---

### Requisito 6: Gestión del estado de servicio

**User Story:** Como empleado de la serviteca, quiero actualizar el estado de una cita de "pendiente" a "atendido", para registrar que el servicio fue completado.

#### Criterios de Aceptación

1. THE Sección_Agenda SHALL mostrar un botón "Marcar como atendido" en cada Cita con Estado_de_Servicio igual a "pendiente".
2. WHEN el empleado hace clic en el botón "Marcar como atendido" de una Cita, THE Sección_Agenda SHALL actualizar el Estado_de_Servicio de esa Cita a "atendido" en el Almacenamiento_Local.
3. IF el Almacenamiento_Local no está disponible al intentar actualizar el Estado_de_Servicio, THEN THE Sección_Agenda SHALL mostrar un mensaje de error indicando que no fue posible guardar el cambio y mantener el Estado_de_Servicio en "pendiente".
4. WHEN el Estado_de_Servicio de una Cita cambia a "atendido", THE Landing_Page SHALL mover automáticamente esa Cita al Historial en la Sección_Historial sin recargar la página.
5. WHILE una Cita tiene Estado_de_Servicio igual a "atendido", THE Sección_Agenda SHALL dejar de mostrar esa Cita en la lista de citas activas.

---

### Requisito 7: Historial de vehículos atendidos

**User Story:** Como empleado de la serviteca, quiero consultar el historial de vehículos ya atendidos, para hacer seguimiento a los servicios prestados.

#### Criterios de Aceptación

1. THE Sección_Historial SHALL mostrar todas las Citas con Estado_de_Servicio igual a "atendido", ordenadas de la más reciente a la más antigua según la fecha y hora de revisión.
2. THE Sección_Historial SHALL mostrar por cada Cita: Número_de_Placa, marca, modelo, tipo de vehículo, tipo de reparación, servicios realizados, fecha y hora de revisión y fecha y hora de devolución.
3. WHEN la Sección_Historial no tiene Citas con Estado_de_Servicio igual a "atendido", THE Sección_Historial SHALL mostrar el mensaje "No hay vehículos atendidos aún" en lugar de la lista de Citas.
4. THE Sección_Historial SHALL mostrar el número total de Citas con Estado_de_Servicio igual a "atendido" en un contador visible en el encabezado de la sección, actualizado cada vez que el listado cambia.

---

### Requisito 8: Persistencia de datos

**User Story:** Como empleado de la serviteca, quiero que los datos se mantengan al recargar la página, para no perder la información ingresada.

#### Criterios de Aceptación

1. WHEN la Landing_Page se carga en el navegador, THE Landing_Page SHALL recuperar todos los Clientes, Vehículos y Citas del Almacenamiento_Local y mostrarlos en sus secciones correspondientes antes de que el usuario pueda interactuar con la página.
2. WHEN una operación de guardado de un nuevo Cliente, Vehículo o Cita es exitosa, THE Landing_Page SHALL persistir el registro en el Almacenamiento_Local antes de cerrar el formulario correspondiente.
3. WHEN el Estado_de_Servicio de una Cita cambia, THE Landing_Page SHALL actualizar el registro de esa Cita en el Almacenamiento_Local de forma sincrónica, de modo que el nuevo estado quede reflejado al recargar la página.
4. IF el Almacenamiento_Local no está disponible o su lectura falla al cargar la Landing_Page, THEN THE Landing_Page SHALL mostrar un mensaje de error indicando que no fue posible recuperar los datos guardados y mostrar las secciones vacías.

---

### Requisito 9: Pie de página

**User Story:** Como visitante, quiero ver la identidad del negocio al final de la página, para saber a quién pertenece el sitio.

#### Criterios de Aceptación

1. THE Pie_de_Página SHALL mostrar el texto "Serviteca ADSO" de forma visible.
2. THE Pie_de_Página SHALL aparecer al final de la Landing_Page, después de la Sección_Historial.
3. THE Pie_de_Página SHALL mostrar el texto "Serviteca ADSO" con una relación de contraste mínima de 4.5:1 entre el color del texto y el color de fondo, conforme al criterio de éxito 1.4.3 de WCAG 2.1 nivel AA.
