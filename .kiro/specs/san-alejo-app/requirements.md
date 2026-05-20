# Documento de Requisitos — San Alejo

## Introducción

**San Alejo** es una aplicación móvil de organización del hogar con una experiencia visual de nivel premium. Permite al usuario inventariar y localizar objetos físicos dentro de contenedores jerárquicos (cajas, maletas, cajones, estantes, etc.) almacenados localmente mediante SQLite. La interfaz sigue un lenguaje de diseño cinematográfico inspirado en Netflix y Apple: tema oscuro, glassmorphism, animaciones fluidas, tarjetas tipo poster y un dashboard visual de alto impacto. El objetivo es que organizar el hogar se sienta tan atractivo como navegar una plataforma de streaming.

---

## Glosario

- **App**: La aplicación móvil San Alejo.
- **Usuario**: La persona que utiliza la App en su dispositivo móvil.
- **Contenedor**: Cualquier elemento físico que puede albergar objetos u otros contenedores (caja, maleta, cajón, estante, bolsa, etc.).
- **Objeto**: Artículo físico del hogar almacenado dentro de un Contenedor.
- **Jerarquía**: Relación padre-hijo entre Contenedores (un cajón dentro de un armario, por ejemplo).
- **Etiqueta**: Palabra clave asignada a un Contenedor u Objeto para facilitar la búsqueda.
- **Base_de_Datos**: Instancia SQLite local que persiste todos los datos en el dispositivo.
- **Dashboard**: Pantalla principal de la App que muestra un resumen visual del inventario.
- **Buscador**: Componente de búsqueda global que localiza Contenedores y Objetos por nombre, etiqueta o ubicación.
- **Tarjeta_Premium**: Componente visual de tipo poster/card que representa un Contenedor u Objeto con imagen, nombre y metadatos.
- **Sistema_de_Diseño**: Conjunto de tokens de color, tipografía, espaciado, sombras y animaciones que definen la identidad visual de la App.
- **Tema_Oscuro**: Paleta de colores con fondos oscuros (#0A0A0F, #12121A) y acentos vibrantes definidos en el Sistema_de_Diseño.
- **Glassmorphism**: Efecto visual de cristal esmerilado aplicado a superficies de la UI (blur + transparencia + borde sutil).
- **Animación_Fluida**: Transición o micro-animación con duración entre 200 ms y 400 ms y curva de easing ease-in-out-cubic.
- **Foto**: Imagen capturada por la cámara del dispositivo o seleccionada de la galería, asociada a un Contenedor u Objeto.
- **Ubicación_Textual**: Descripción en lenguaje natural de dónde se encuentra un Contenedor (ej. "Cuarto de servicio, estante superior").
- **Notificación_Local**: Alerta generada por la App sin conexión a internet.
- **Exportación**: Archivo generado por la App que contiene una copia del inventario en formato JSON o CSV.
- **Onboarding**: Flujo de bienvenida que se muestra al Usuario la primera vez que abre la App.

---

## Requisitos

---

### Requisito 1: Onboarding y Primera Experiencia

**User Story:** Como Usuario nuevo, quiero un flujo de bienvenida visualmente impactante, para que entienda el valor de la App y me motive a comenzar a organizar mis pertenencias.

#### Criterios de Aceptación

1. WHEN el Usuario abre la App por primera vez, THE App SHALL mostrar el flujo de Onboarding con un mínimo de 3 pantallas animadas que presenten las funciones principales.
2. WHEN el Usuario completa el Onboarding, THE App SHALL marcar el estado de primera apertura como completado en la Base_de_Datos y no volver a mostrar el Onboarding.
3. WHEN el Usuario abre la App en aperturas posteriores, THE App SHALL navegar directamente al Dashboard sin mostrar el Onboarding.
4. THE App SHALL renderizar cada pantalla del Onboarding con Animación_Fluida de entrada y salida entre slides.
5. WHERE el dispositivo soporta haptic feedback, THE App SHALL activar una vibración suave al avanzar entre pantallas del Onboarding.

---

### Requisito 2: Dashboard Visual Principal

**User Story:** Como Usuario, quiero un Dashboard visualmente impresionante al abrir la App, para tener una visión general de mi inventario de forma atractiva y rápida.

#### Criterios de Aceptación

1. THE Dashboard SHALL mostrar el total de Contenedores y el total de Objetos registrados en contadores animados.
2. WHEN el Usuario accede al Dashboard, THE Dashboard SHALL renderizar las Tarjetas_Premium de los Contenedores más recientes en un carrusel horizontal con scroll suave.
3. THE Dashboard SHALL aplicar el Tema_Oscuro con fondos en los colores definidos por el Sistema_de_Diseño.
4. THE Dashboard SHALL mostrar una sección de "Búsqueda Rápida" con el Buscador accesible sin navegación adicional.
5. WHEN el número de Contenedores es cero, THE Dashboard SHALL mostrar un estado vacío con ilustración animada e instrucción para crear el primer Contenedor.
6. THE Dashboard SHALL aplicar Glassmorphism en la barra de navegación inferior y en las tarjetas de estadísticas.
7. WHEN el Usuario hace scroll hacia abajo en el Dashboard, THE Dashboard SHALL aplicar un efecto parallax sutil en el encabezado.

---

### Requisito 3: Gestión de Contenedores

**User Story:** Como Usuario, quiero crear, editar y eliminar Contenedores con una experiencia visual premium, para organizar mis pertenencias de forma jerárquica y atractiva.

#### Criterios de Aceptación

1. WHEN el Usuario crea un Contenedor, THE App SHALL persistir en la Base_de_Datos: nombre (obligatorio, máximo 80 caracteres), tipo (caja, maleta, cajón, estante, bolsa u otro), Ubicación_Textual (opcional, máximo 200 caracteres), descripción (opcional, máximo 500 caracteres), color de acento (seleccionado de una paleta de 12 colores predefinidos), Foto (opcional) y fecha de creación.
2. WHEN el Usuario asigna un Contenedor como hijo de otro Contenedor, THE App SHALL validar que la jerarquía no supere 5 niveles de profundidad y persistir la relación padre-hijo en la Base_de_Datos.
3. IF el Usuario intenta crear un Contenedor con nombre vacío, THEN THE App SHALL mostrar un mensaje de error inline sin cerrar el formulario.
4. WHEN el Usuario edita un Contenedor, THE App SHALL pre-cargar todos los campos con los valores actuales y persistir únicamente los campos modificados.
5. WHEN el Usuario elimina un Contenedor que contiene Objetos o Contenedores hijos, THE App SHALL mostrar un diálogo de confirmación que indique el número de elementos que serán eliminados en cascada.
6. WHEN el Usuario confirma la eliminación de un Contenedor, THE App SHALL eliminar en cascada todos los Objetos y Contenedores hijos asociados de la Base_de_Datos.
7. THE App SHALL renderizar cada Contenedor como una Tarjeta_Premium con su Foto (o un ícono generativo basado en el tipo si no hay Foto), nombre, tipo, cantidad de Objetos directos y color de acento.
8. WHEN el Usuario abre un Contenedor, THE App SHALL mostrar la pantalla de detalle con Animación_Fluida de tipo "shared element transition" desde la Tarjeta_Premium.

---

### Requisito 4: Gestión de Objetos

**User Story:** Como Usuario, quiero registrar objetos dentro de mis Contenedores con detalle visual, para saber exactamente qué tengo y dónde está cada cosa.

#### Criterios de Aceptación

1. WHEN el Usuario crea un Objeto, THE App SHALL persistir en la Base_de_Datos: nombre (obligatorio, máximo 80 caracteres), Contenedor padre (obligatorio), descripción (opcional, máximo 500 caracteres), cantidad (entero positivo, mínimo 1), Foto (opcional), Etiquetas (opcional, máximo 10 por Objeto) y fecha de creación.
2. IF el Usuario intenta crear un Objeto con cantidad menor a 1 o no numérica, THEN THE App SHALL mostrar un mensaje de error inline y mantener el foco en el campo de cantidad.
3. WHEN el Usuario registra un Objeto con Foto, THE App SHALL comprimir la imagen a un máximo de 800×800 píxeles y 200 KB antes de almacenarla en la Base_de_Datos.
4. THE App SHALL renderizar cada Objeto como una Tarjeta_Premium con su Foto (o ícono genérico si no hay Foto), nombre, cantidad y Etiquetas.
5. WHEN el Usuario mueve un Objeto a otro Contenedor, THE App SHALL actualizar la referencia del Contenedor padre en la Base_de_Datos y confirmar la acción con una Animación_Fluida de transición.
6. WHEN el Usuario elimina un Objeto, THE App SHALL mostrar un diálogo de confirmación antes de proceder con la eliminación.

---

### Requisito 5: Sistema de Búsqueda Global

**User Story:** Como Usuario, quiero encontrar cualquier Objeto o Contenedor en segundos, para no perder tiempo buscando mis pertenencias.

#### Criterios de Aceptación

1. WHEN el Usuario escribe en el Buscador, THE Buscador SHALL ejecutar la consulta en la Base_de_Datos y mostrar resultados en tiempo real con una latencia máxima de 300 ms desde el último carácter ingresado.
2. THE Buscador SHALL buscar coincidencias parciales en: nombre de Contenedor, nombre de Objeto, Etiquetas, Ubicación_Textual y descripción.
3. WHEN el Buscador no encuentra resultados, THE Buscador SHALL mostrar un estado vacío con sugerencia de ampliar los términos de búsqueda.
4. WHEN el Usuario selecciona un resultado del Buscador, THE App SHALL navegar a la pantalla de detalle del Contenedor u Objeto correspondiente con Animación_Fluida.
5. THE Buscador SHALL mostrar el "camino de ubicación" de cada resultado (ej. "Armario > Cajón superior > Caja azul") para que el Usuario identifique la jerarquía sin abrir la pantalla de detalle.
6. WHEN el Buscador está activo, THE App SHALL mostrar las últimas 5 búsquedas recientes del Usuario almacenadas en la Base_de_Datos.

---

### Requisito 6: Sistema de Etiquetas

**User Story:** Como Usuario, quiero etiquetar mis Contenedores y Objetos con palabras clave, para filtrar y agrupar mis pertenencias por categoría.

#### Criterios de Aceptación

1. THE App SHALL permitir al Usuario crear Etiquetas con nombre (máximo 30 caracteres) y color de acento seleccionado de la paleta del Sistema_de_Diseño.
2. WHEN el Usuario asigna una Etiqueta a un Contenedor u Objeto, THE App SHALL persistir la relación en la Base_de_Datos.
3. THE App SHALL mostrar una pantalla de gestión de Etiquetas donde el Usuario pueda ver todas las Etiquetas creadas con el conteo de elementos asociados a cada una.
4. WHEN el Usuario filtra por una Etiqueta, THE App SHALL mostrar todos los Contenedores y Objetos que tengan esa Etiqueta asignada.
5. WHEN el Usuario elimina una Etiqueta, THE App SHALL desasociar la Etiqueta de todos los Contenedores y Objetos relacionados sin eliminar dichos elementos.

---

### Requisito 7: Galería de Fotos y Gestión de Imágenes

**User Story:** Como Usuario, quiero capturar o seleccionar fotos para mis Contenedores y Objetos, para identificarlos visualmente de forma rápida y atractiva.

#### Criterios de Aceptación

1. WHEN el Usuario agrega una Foto a un Contenedor u Objeto, THE App SHALL ofrecer las opciones: capturar con cámara o seleccionar de la galería del dispositivo.
2. WHEN el Usuario captura o selecciona una Foto, THE App SHALL mostrar una vista previa con opción de recorte en proporción 1:1 antes de confirmar.
3. WHEN el Usuario confirma la Foto, THE App SHALL comprimir y almacenar la imagen según lo definido en el Requisito 4, criterio 3.
4. IF el dispositivo deniega el permiso de cámara o galería, THEN THE App SHALL mostrar un mensaje explicativo con un botón que redirija al Usuario a la configuración del sistema operativo.
5. THE App SHALL mostrar una pantalla de Galería que agrupe todas las Fotos registradas en la App ordenadas por fecha de captura descendente.

---

### Requisito 8: Pantalla de Detalle de Contenedor

**User Story:** Como Usuario, quiero ver el contenido completo de un Contenedor en una pantalla detallada y visualmente rica, para explorar su inventario de forma inmersiva.

#### Criterios de Aceptación

1. WHEN el Usuario abre la pantalla de detalle de un Contenedor, THE App SHALL mostrar: Foto en formato hero (pantalla completa con overlay degradado), nombre, tipo, Ubicación_Textual, descripción, Etiquetas, lista de Objetos directos y lista de Contenedores hijos.
2. THE App SHALL renderizar la lista de Objetos y Contenedores hijos como una cuadrícula de Tarjetas_Premium con scroll vertical.
3. WHEN el Contenedor no tiene Objetos ni Contenedores hijos, THE App SHALL mostrar un estado vacío con botón de acción rápida para agregar el primer elemento.
4. THE App SHALL mostrar el "camino de ubicación" jerárquico (breadcrumb) en la parte superior de la pantalla de detalle.
5. WHEN el Usuario hace tap en la Foto hero, THE App SHALL abrir la imagen en pantalla completa con soporte de zoom mediante gesto de pinch.

---

### Requisito 9: Exportación e Importación de Inventario

**User Story:** Como Usuario, quiero exportar e importar mi inventario, para hacer respaldos y transferir mis datos entre dispositivos.

#### Criterios de Aceptación

1. WHEN el Usuario solicita exportar el inventario, THE App SHALL generar un archivo JSON que contenga todos los Contenedores, Objetos, Etiquetas y relaciones jerárquicas de la Base_de_Datos.
2. WHEN el archivo de exportación es generado, THE App SHALL invocar el sistema de compartir nativo del sistema operativo para que el Usuario elija el destino del archivo.
3. WHEN el Usuario importa un archivo JSON de exportación válido, THE App SHALL fusionar los datos importados con los existentes en la Base_de_Datos sin duplicar elementos con el mismo identificador único.
4. IF el archivo de importación tiene un formato inválido o está corrupto, THEN THE App SHALL mostrar un mensaje de error descriptivo sin modificar la Base_de_Datos existente.
5. THE App SHALL exportar también en formato CSV con columnas: id, nombre, tipo, ubicación, descripción, etiquetas, contenedor_padre_id, fecha_creación.

---

### Requisito 10: Sistema de Diseño y Experiencia Visual

**User Story:** Como Usuario, quiero que la App tenga una apariencia visual premium y consistente en todas las pantallas, para disfrutar de una experiencia de uso elegante y moderna.

#### Criterios de Aceptación

1. THE Sistema_de_Diseño SHALL definir y aplicar de forma consistente en toda la App: paleta de colores (fondo primario #0A0A0F, fondo secundario #12121A, superficie #1C1C2E, acento primario #6C63FF, acento secundario #FF6584, texto primario #FFFFFF, texto secundario #A0A0B0).
2. THE Sistema_de_Diseño SHALL definir y aplicar tipografía con jerarquía de 5 niveles: Display (32 sp), Headline (24 sp), Title (18 sp), Body (14 sp) y Caption (12 sp), usando la fuente Inter o SF Pro según el sistema operativo.
3. THE App SHALL aplicar Glassmorphism en: barra de navegación inferior, modales, tarjetas de estadísticas del Dashboard y encabezados de sección, con blur de 20 px, opacidad de fondo del 15% y borde de 1 px con opacidad del 20%.
4. THE App SHALL ejecutar Animación_Fluida en todas las transiciones de pantalla, aparición de modales, expansión de tarjetas y respuesta a gestos del Usuario.
5. THE App SHALL aplicar sombras suaves en las Tarjetas_Premium con un radio de 16 px, desplazamiento Y de 8 px, opacidad del 40% y color basado en el acento del elemento.
6. WHERE el dispositivo soporta modo oscuro del sistema operativo, THE App SHALL respetar la preferencia del sistema y mantener el Tema_Oscuro como único tema disponible en esta versión.
7. THE App SHALL mostrar estados de carga (skeleton screens) en lugar de spinners genéricos mientras se recuperan datos de la Base_de_Datos.
8. WHEN el Usuario interactúa con cualquier elemento táctil, THE App SHALL proporcionar retroalimentación visual inmediata (ripple effect o scale animation) en un tiempo máximo de 16 ms.

---

### Requisito 11: Navegación y Arquitectura de Pantallas

**User Story:** Como Usuario, quiero navegar entre las secciones de la App de forma intuitiva y fluida, para acceder a cualquier función sin confusión.

#### Criterios de Aceptación

1. THE App SHALL implementar una barra de navegación inferior con 4 destinos principales: Dashboard, Explorar (árbol de Contenedores), Buscar y Ajustes.
2. WHEN el Usuario navega entre destinos de la barra inferior, THE App SHALL aplicar Animación_Fluida de transición entre pantallas con duración de 250 ms.
3. THE App SHALL soportar navegación gestual (swipe back) en todas las pantallas de detalle en dispositivos iOS y Android.
4. WHEN el Usuario presiona el botón de acción flotante (FAB) en el Dashboard o en la pantalla de Explorar, THE App SHALL mostrar un menú expandible con opciones para crear Contenedor u Objeto con Animación_Fluida de expansión.
5. THE App SHALL mantener el estado de scroll y posición de cada pantalla al regresar desde una pantalla de detalle (scroll restoration).

---

### Requisito 12: Pantalla de Ajustes y Configuración

**User Story:** Como Usuario, quiero configurar preferencias de la App, para personalizar mi experiencia y gestionar mis datos.

#### Criterios de Aceptación

1. THE App SHALL mostrar en la pantalla de Ajustes: opciones de exportación/importación, información de la versión de la App, estadísticas de uso (total de Contenedores, Objetos, Fotos y tamaño de la Base_de_Datos) y opción para restablecer todos los datos.
2. WHEN el Usuario solicita restablecer todos los datos, THE App SHALL mostrar un diálogo de confirmación con texto explícito sobre la irreversibilidad de la acción antes de proceder.
3. WHEN el Usuario confirma el restablecimiento, THE App SHALL eliminar todos los registros de la Base_de_Datos y navegar al estado de App vacía sin reiniciar la aplicación.
4. THE App SHALL mostrar el tamaño actual de la Base_de_Datos en megabytes, actualizado en tiempo real al abrir la pantalla de Ajustes.

---

### Requisito 13: Persistencia Local y Rendimiento

**User Story:** Como Usuario, quiero que la App funcione completamente sin conexión a internet y responda de forma rápida, para usarla en cualquier lugar sin depender de la red.

#### Criterios de Aceptación

1. THE App SHALL operar en su totalidad sin requerir conexión a internet, almacenando todos los datos en la Base_de_Datos SQLite local del dispositivo.
2. THE App SHALL iniciar y mostrar el Dashboard en un tiempo máximo de 2 segundos desde que el Usuario toca el ícono de la App en dispositivos con hardware de gama media (procesador de 4 núcleos, 3 GB RAM).
3. WHEN la Base_de_Datos contiene más de 500 Contenedores u Objetos, THE App SHALL implementar paginación o carga virtual (lazy loading) para mantener el tiempo de renderizado de listas por debajo de 16 ms por frame.
4. THE App SHALL ejecutar todas las operaciones de escritura en la Base_de_Datos en un hilo secundario para no bloquear el hilo principal de la UI.
5. IF una operación de escritura en la Base_de_Datos falla, THEN THE App SHALL revertir la transacción completa y mostrar un mensaje de error al Usuario sin dejar la Base_de_Datos en estado inconsistente.

---

### Requisito 14: Accesibilidad

**User Story:** Como Usuario con necesidades de accesibilidad, quiero que la App sea usable con tecnologías de asistencia, para no quedar excluido de sus funciones.

#### Criterios de Aceptación

1. THE App SHALL asignar etiquetas de accesibilidad descriptivas a todos los elementos interactivos (botones, tarjetas, campos de formulario) para su lectura por TalkBack (Android) y VoiceOver (iOS).
2. THE App SHALL garantizar un ratio de contraste mínimo de 4.5:1 entre el texto y su fondo en todos los componentes, conforme al estándar WCAG 2.1 nivel AA.
3. THE App SHALL soportar el escalado de texto del sistema operativo hasta un 200% sin que el contenido quede cortado ni superpuesto.
4. WHERE el Usuario activa el modo de reducción de movimiento del sistema operativo, THE App SHALL reducir o eliminar las Animaciones_Fluidas sin afectar la funcionalidad.
