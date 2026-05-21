# Requirements Document

## Introduction

San Alejo es una aplicación móvil de organización del hogar con una interfaz visual premium estilo Netflix/Apple. Permite al usuario catalogar cajas, contenedores, maletas, cajones y objetos físicos del hogar mediante una experiencia cinematográfica: tarjetas tipo poster, tema oscuro, glassmorphism, animaciones suaves y un dashboard visual de alto impacto. Toda la información se almacena localmente en SQLite, sin necesidad de conexión a internet.

La app está diseñada para ser usada por una sola persona o familia, sin autenticación remota, priorizando velocidad, privacidad y una experiencia visual impresionante.

---

## Glossary

- **App**: La aplicación móvil San Alejo.
- **Container**: Unidad de almacenamiento física (caja, maleta, cajón, estante, bolsa, etc.) que puede contener Items u otros Containers anidados.
- **Item**: Objeto físico del hogar almacenado dentro de un Container.
- **Location**: Lugar físico donde se encuentra un Container (habitación, bodega, garaje, etc.).
- **Tag**: Etiqueta de texto libre asignada a un Container o Item para facilitar búsquedas y filtrado.
- **Cover_Image**: Imagen representativa asignada a un Container o Item, tomada desde la cámara o galería del dispositivo.
- **Dashboard**: Pantalla principal de la App que muestra un resumen visual de todos los Containers y estadísticas de uso.
- **Search_Engine**: Módulo de búsqueda en tiempo real que consulta la base de datos SQLite local.
- **SQLite_DB**: Base de datos SQLite local embebida en el dispositivo del usuario.
- **Navigator**: Sistema de navegación entre pantallas de la App.
- **Theme_Engine**: Módulo responsable de aplicar el sistema de diseño visual (colores, tipografía, sombras, glassmorphism, gradientes y efectos visuales).
- **Camera_Module**: Módulo que accede a la cámara y galería del dispositivo para capturar o seleccionar Cover_Images.
- **Export_Module**: Módulo que genera y comparte archivos de respaldo de la base de datos.
- **Onboarding**: Flujo de bienvenida mostrado al usuario la primera vez que abre la App.
- **Root_Container**: Container cuyo campo `parent_container_id` es NULL; representa una unidad de almacenamiento de nivel superior.
- **Breadcrumb**: Ruta jerárquica de Containers que indica la ubicación de un Item o sub-Container dentro de la jerarquía.
- **Animation_Engine**: Subsistema de animaciones basado en React Native Reanimated v3 responsable de todas las transiciones, microinteracciones y efectos de movimiento de la App.
- **Glassmorphism_Level**: Nivel de intensidad del efecto glassmorphism aplicado a una superficie: `ultra-light`, `light`, `medium` o `heavy`, cada uno con valores distintos de opacidad de fondo, opacidad de borde y radio de blur.
- **Hero_Section**: Área de cabecera de pantalla completa que muestra el Container más reciente o destacado con formato cinematográfico full-width.
- **Poster_Card**: Tarjeta visual con aspect ratio 2:3 (portrait) que representa un Container con múltiples capas visuales: imagen de fondo, gradiente, overlay glassmorphism y texto.
- **Shimmer_Effect**: Animación de brillo que recorre una superficie de izquierda a derecha para indicar estado de carga, implementada con un gradiente lineal animado.
- **Count_Up_Animation**: Animación numérica que incrementa un valor desde cero hasta el valor final al aparecer en pantalla, usada en estadísticas y contadores.
- **Mesh_Gradient**: Gradiente de malla animado con múltiples puntos de color que se desplazan suavemente, usado como fondo del Dashboard.
- **Parallax_Scroll**: Efecto de desplazamiento donde el fondo se mueve a una velocidad diferente al contenido principal, creando sensación de profundidad.
- **Shared_Element_Transition**: Transición de pantalla donde un elemento visual (como una Card) se expande animadamente para convertirse en la pantalla de destino.
- **FAB_Radial_Menu**: Menú radial que emerge del FAB al presionarlo, mostrando opciones secundarias en arco con animación de morph.
- **Blob_Tab_Indicator**: Indicador animado del tab activo en el Tab Bar que se desplaza fluidamente entre tabs como una forma orgánica.
- **Statistics_Screen**: Pantalla dedicada a mostrar estadísticas visuales de la colección: donut charts, bar charts, progress bars animadas y mapas de calor.
- **Dominant_Color**: Color dominante extraído de la Cover_Image de un Container, usado para generar gradientes dinámicos personalizados.
- **Noise_Texture**: Textura de ruido sutil superpuesta sobre fondos para añadir profundidad visual y evitar superficies planas.
- **Vignette_Effect**: Oscurecimiento gradual de los bordes de una imagen hacia el centro, aplicado sobre Cover_Images para mejorar la legibilidad del texto superpuesto.
- **Glow_Effect**: Halo de luz difusa alrededor de un elemento activo o seleccionado, implementado con sombras de color del acento.
- **Lottie_Animation**: Animación vectorial en formato JSON reproducida mediante la librería Lottie, usada en estados vacíos y splash screen.
- **Splash_Screen**: Pantalla de carga inicial animada que muestra el logo de la App mientras se inicializa la base de datos.
- **Context_Menu_Preview**: Menú contextual con vista previa del contenido (estilo peek and pop de iOS) que aparece al hacer long press sobre una Card.
- **Gradient_Border**: Borde de un elemento implementado con un gradiente de colores en lugar de un color sólido, usado en Cards destacadas.
- **Inter_Variable**: Fuente Inter en formato variable que permite ajustar peso, ancho y otros ejes tipográficos de forma continua.
- **SF_Symbols_Style**: Estilo de iconografía con variantes redondeadas y rellenas, inspirado en SF Symbols de Apple, aplicado a todos los iconos de la App.

---

## Requirements

---

### Requirement 1: Arquitectura y Stack Tecnológico

**User Story:** As a developer, I want a well-defined technology stack and project architecture, so that the app is maintainable, scalable, and delivers a premium visual experience.

#### Acceptance Criteria

1. THE App SHALL be built using React Native with Expo SDK (latest stable version at project initialization) as the cross-platform mobile framework targeting iOS and Android.
2. THE App SHALL use TypeScript as the primary programming language for all source files, with strict mode enabled in `tsconfig.json`.
3. THE App SHALL use Expo SQLite (`expo-sqlite`) as the local persistence layer, with no remote database dependency.
4. THE App SHALL use React Navigation v6 with Stack Navigator and Bottom Tab Navigator as the Navigator.
5. THE App SHALL use Zustand as the global state management library for all shared application state.
6. THE App SHALL use React Native Reanimated v3 for all animated transitions and interactions.
7. THE App SHALL use NativeWind (Tailwind CSS for React Native) combined with a custom Theme_Engine for all styling.
8. THE App SHALL use Expo Image Picker for Camera_Module integration.
9. THE App SHALL use Expo File System and Expo Sharing for Export_Module functionality.
10. THE App SHALL use React Native Gesture Handler for swipe and drag interactions.

---

### Requirement 2: Estructura de Carpetas del Proyecto

**User Story:** As a developer, I want a professional folder structure, so that the codebase is organized, easy to navigate, and follows industry conventions.

#### Acceptance Criteria

1. THE App SHALL organize source code under a `/src` root directory with the following top-level subdirectories: `components`, `screens`, `navigation`, `store`, `database`, `hooks`, `utils`, `assets`, `theme`, and `types`.
2. THE `components` directory SHALL be subdivided into `common` (reusable atomic components), `containers` (Container-specific components), `items` (Item-specific components), and `ui` (design system primitives).
3. THE `database` directory SHALL contain: `schema.ts` (table definitions), `migrations.ts` (version migrations), `repositories/` (one repository file per entity: `ContainerRepository`, `ItemRepository`, `LocationRepository`, `TagRepository`), and `db.ts` (singleton connection manager).
4. THE `theme` directory SHALL contain: `colors.ts`, `typography.ts`, `spacing.ts`, `shadows.ts`, `animations.ts`, and `index.ts` (unified theme export).
5. THE `screens` directory SHALL be subdivided by feature: `dashboard/`, `containers/`, `items/`, `search/`, `settings/`, and `onboarding/`.
6. THE `types` directory SHALL contain TypeScript interface definitions for all domain entities: `Container.ts`, `Item.ts`, `Location.ts`, `Tag.ts`, and `common.ts`.

---

### Requirement 3: Modelo de Datos SQLite

**User Story:** As a user, I want my containers and items to be stored reliably on my device, so that my organization data is always available offline.

#### Acceptance Criteria

1. THE SQLite_DB SHALL contain a `containers` table with columns: `id` (TEXT PRIMARY KEY, UUID v4), `name` (TEXT NOT NULL), `description` (TEXT), `type` (TEXT NOT NULL, constrained to values: `'box'`, `'suitcase'`, `'drawer'`, `'shelf'`, `'bag'`, `'other'`), `location_id` (TEXT, FK → `locations.id`), `parent_container_id` (TEXT, FK → `containers.id`, NULL for Root_Containers), `cover_image_uri` (TEXT), `color_tag` (TEXT), `created_at` (INTEGER, Unix timestamp in milliseconds), `updated_at` (INTEGER, Unix timestamp in milliseconds).
2. THE SQLite_DB SHALL contain an `items` table with columns: `id` (TEXT PRIMARY KEY, UUID v4), `name` (TEXT NOT NULL), `description` (TEXT), `quantity` (INTEGER NOT NULL DEFAULT 1, minimum value 0), `container_id` (TEXT NOT NULL, FK → `containers.id`), `cover_image_uri` (TEXT), `created_at` (INTEGER, Unix timestamp in milliseconds), `updated_at` (INTEGER, Unix timestamp in milliseconds).
3. THE SQLite_DB SHALL contain a `locations` table with columns: `id` (TEXT PRIMARY KEY, UUID v4), `name` (TEXT NOT NULL), `icon` (TEXT), `created_at` (INTEGER, Unix timestamp in milliseconds).
4. THE SQLite_DB SHALL contain a `tags` table with columns: `id` (TEXT PRIMARY KEY, UUID v4), `name` (TEXT NOT NULL UNIQUE), `color` (TEXT NOT NULL).
5. THE SQLite_DB SHALL contain a `container_tags` junction table with columns: `container_id` (TEXT NOT NULL, FK → `containers.id`), `tag_id` (TEXT NOT NULL, FK → `tags.id`), with PRIMARY KEY (`container_id`, `tag_id`).
6. THE SQLite_DB SHALL contain an `item_tags` junction table with columns: `item_id` (TEXT NOT NULL, FK → `items.id`), `tag_id` (TEXT NOT NULL, FK → `tags.id`), with PRIMARY KEY (`item_id`, `tag_id`).
7. WHEN the App is launched for the first time on a device with no existing SQLite_DB, THE SQLite_DB SHALL execute all schema creation scripts and seed a default Location with `name` = `"Sin ubicación"`.
8. WHEN a new version of the App introduces schema changes, THE SQLite_DB SHALL run incremental migration scripts in version order without deleting existing user data.
9. IF a SQLite_DB write operation fails due to a constraint violation or I/O error, THEN THE App SHALL roll back the transaction and display an error message to the user without leaving the database in a partial state.
10. WHEN a Container is written to SQLite_DB and then read back by its `id`, THE SQLite_DB SHALL return a Container object with field values identical to those that were written (round-trip property).

---

### Requirement 4: Onboarding

**User Story:** As a new user, I want a welcoming onboarding experience, so that I understand the app's purpose and feel excited to start organizing.

#### Acceptance Criteria

1. WHEN the App is launched for the first time and no onboarding-complete flag exists in local storage, THE Navigator SHALL display the Onboarding screen before the Dashboard.
2. THE Onboarding SHALL consist of exactly 3 slides: (1) App presentation with logo and tagline, (2) explanation of the Containers and Items concept, (3) call-to-action to create the first Container.
3. WHEN the user completes the final Onboarding slide or taps the skip button, THE App SHALL persist an onboarding-complete flag in local storage so that the Onboarding is not shown on subsequent launches.
4. THE Onboarding SHALL use full-screen animated illustrations with slide transitions powered by React Native Reanimated at a duration of 300ms per slide.
5. WHEN the App is launched on a device where the onboarding-complete flag is already set, THE Navigator SHALL display the Dashboard directly, bypassing the Onboarding screen.

---

### Requirement 5: Dashboard Principal

**User Story:** As a user, I want a visually stunning dashboard with a cinematic experience, so that I can see all my containers at a glance and feel motivated to keep my home organized.

#### Acceptance Criteria

1. THE Dashboard SHALL display all Root_Containers as a grid of Poster_Cards sorted by `updated_at` descending.
2. WHEN the user has no Containers, THE Dashboard SHALL display un estado vacío con una Lottie_Animation y un botón de llamada a la acción para crear el primer Container.
3. THE Dashboard SHALL display a summary bar showing: total number of Root_Containers, total number of Items across all Containers, and number of distinct Locations in use, con Count_Up_Animation al aparecer en pantalla.
4. WHEN a Container has a Cover_Image, THE Dashboard Card SHALL display the Cover_Image as a full-bleed background with a gradient overlay, a Vignette_Effect on the edges, and the Container `name` in bold typography at the bottom.
5. WHEN a Container has no Cover_Image, THE Dashboard Card SHALL display a color gradient background derived from the Container's `color_tag` field and an icon representing the Container `type`.
6. THE Dashboard SHALL support both grid view (2 columns) and list view, toggled by a button in the header, with the selected view persisted across sessions.
7. WHEN the user performs a pull-to-refresh gesture on the Dashboard, THE App SHALL re-query the SQLite_DB and update the Container list within 500ms, displaying a custom pull-to-refresh animation with the App logo.
8. THE Dashboard SHALL include a FAB_Radial_Menu that expands into a radial menu with options: crear Container, crear Item rápido, and escanear código QR, with a morph animation of 250ms duration.
9. WHEN the user long-presses a Container Card, THE App SHALL display a Context_Menu_Preview with options: Edit, Move, Delete, and Share, with haptic feedback (medium) triggered simultaneously.
10. THE Dashboard SHALL display a Hero_Section at the top showing the most recently updated Container in full-width cinematic format with Parallax_Scroll effect as the user scrolls down.
11. THE Dashboard SHALL display a "Recientes" horizontal scroll section below the Hero_Section showing the 5 most recently accessed Containers as Poster_Cards, styled as a Netflix-style row.
12. THE Dashboard SHALL display a "Por ubicación" section grouping Root_Containers by their assigned Location, with each group displayed as a horizontally scrollable row.
13. THE Dashboard SHALL display a Mesh_Gradient as the background that animates subtly with a cycle duration of 8 seconds, using the accent color `#6C63FF` and its complementary `#FF6584` as gradient points.
14. WHEN the user scrolls down on the Dashboard, THE Navigator SHALL transition the navigation bar from opaque to transparent with a large title style, revealing the Mesh_Gradient background behind it.
15. THE Dashboard SHALL display a mini activity chart showing the number of Items added per day over the last 7 days, rendered as a compact bar chart below the summary bar.

---

### Requirement 6: Gestión de Containers

**User Story:** As a user, I want to create, edit, and organize containers, so that I can represent my physical storage units digitally.

#### Acceptance Criteria

1. WHEN the user taps the FAB on the Dashboard, THE Navigator SHALL present the Create Container screen as a bottom sheet modal with an upward slide animation of 300ms duration.
2. THE Create Container screen SHALL include the following fields: `name` (text input, required), `type` (picker with values: box, suitcase, drawer, shelf, bag, other), `location` (picker populated from existing Locations), `description` (text input, optional), `color_tag` (color palette picker), and `cover_image` (Camera_Module trigger).
3. WHEN the user submits the Create Container form with a non-empty `name`, THE App SHALL generate a UUID v4, persist the Container to SQLite_DB, dismiss the modal, and animate the new Card into the Dashboard grid.
4. IF the user submits the Create Container form with an empty `name` field, THEN THE App SHALL display an inline validation error message below the `name` field without dismissing the form.
5. WHEN the user taps a Container Card, THE Navigator SHALL push the Container Detail screen displaying the Container's Cover_Image as a hero header, its metadata, nested sub-Containers, and its Items list.
6. THE Container Detail screen SHALL include a secondary FAB that opens the Create Container screen pre-configured to create a sub-Container nested under the current Container.
7. WHEN the user taps Edit on a Container, THE App SHALL open the Edit Container screen pre-populated with all of the Container's current field values.
8. WHEN the user confirms deletion of a Container, THE App SHALL delete the Container and all of its nested sub-Containers and Items from SQLite_DB in a single cascading transaction.
9. IF a Container has nested sub-Containers or Items, THEN THE App SHALL display a confirmation dialog stating the number of nested sub-Containers and Items that will be deleted before executing the deletion.
10. THE Container Detail screen SHALL display the total Item count, including Items in all nested sub-Containers at any depth, as a numeric badge.

---

### Requirement 7: Gestión de Items

**User Story:** As a user, I want to add and manage items inside containers, so that I can track exactly what is stored where.

#### Acceptance Criteria

1. WHEN the user taps "Add Item" inside a Container Detail screen, THE Navigator SHALL present the Create Item screen as a bottom sheet modal with an upward slide animation of 300ms duration.
2. THE Create Item screen SHALL include the following fields: `name` (text input, required), `description` (text input, optional), `quantity` (numeric stepper with minimum value 1 and increment/decrement buttons), `cover_image` (Camera_Module trigger), and `tags` (multi-select Tag picker).
3. WHEN the user submits the Create Item form with a non-empty `name`, THE App SHALL persist the Item to SQLite_DB linked to the current Container's `id` and animate the new Item into the list.
4. IF the user submits the Create Item form with an empty `name` field, THEN THE App SHALL display an inline validation error below the `name` field without dismissing the form.
5. WHEN the user taps an Item in the list, THE App SHALL display an Item Detail bottom sheet showing the Cover_Image, `name`, `description`, `quantity`, assigned Tags, and the Breadcrumb path to the Container.
6. WHEN the user edits an Item's `quantity` to zero, THE App SHALL display a confirmation dialog offering two options: delete the Item permanently or retain the Item with `quantity` = 0.
7. THE Item list inside a Container Detail screen SHALL support a swipe-left gesture on each row to reveal Delete and Edit quick-action buttons.
8. WHEN the user moves an Item to a different Container, THE App SHALL update the Item's `container_id` in SQLite_DB and reflect the change immediately in both the source and destination Container Detail screens without requiring a manual refresh.

---

### Requirement 8: Búsqueda Global

**User Story:** As a user, I want to search across all my containers and items instantly, so that I can find any object without manually browsing.

#### Acceptance Criteria

1. THE Dashboard SHALL include a persistent search bar at the top that activates the Search_Engine when the user taps it.
2. WHEN the user types in the search bar, THE Search_Engine SHALL query SQLite_DB for Containers and Items whose `name` or `description` fields contain the search string, with results rendered within 300ms of the last keystroke.
3. THE Search_Engine SHALL display results grouped by type: Containers first, then Items, each entry showing the Cover_Image thumbnail, `name`, and Location Breadcrumb.
4. WHEN the user taps a search result, THE Navigator SHALL navigate to the corresponding Container Detail or Item Detail screen.
5. WHEN the search bar is cleared or empty, THE Search_Engine SHALL display the 5 most recently accessed Containers as "Recientes" suggestions, ordered by last access time descending.
6. IF the search query returns no results, THEN THE Search_Engine SHALL display a "No se encontraron resultados" empty state message that includes the search term.
7. WHEN the user clears the search bar, THE Search_Engine SHALL dismiss the results list and return the Dashboard to its default state within 100ms.

---

### Requirement 9: Sistema de Etiquetas (Tags)

**User Story:** As a user, I want to tag containers and items with custom labels, so that I can filter and find related objects across different containers.

#### Acceptance Criteria

1. THE App SHALL allow the user to create Tags with a `name` (text, required, unique) and a `color` selected from a predefined palette of exactly 12 colors.
2. IF the user attempts to create a Tag with a `name` that already exists in SQLite_DB, THEN THE App SHALL display an inline error message indicating the name is already in use without creating a duplicate Tag.
3. WHEN the user assigns a Tag to a Container or Item, THE App SHALL persist the relationship in the corresponding junction table (`container_tags` or `item_tags`) in SQLite_DB.
4. THE Dashboard SHALL include a horizontal Tag filter bar below the search bar that filters the Container grid by selected Tags.
5. WHEN the user selects one or more Tags in the filter bar, THE Dashboard SHALL display only Root_Containers that have ALL selected Tags assigned.
6. WHEN the user taps a Tag chip on a Container Card or Item Detail, THE App SHALL navigate to a filtered view showing all Containers and Items that have that Tag assigned.

---

### Requirement 10: Gestión de Ubicaciones

**User Story:** As a user, I want to define physical locations, so that I can know in which room or place each container is stored.

#### Acceptance Criteria

1. THE App SHALL provide a Locations management screen accessible from the Settings tab.
2. WHEN the user creates a Location with a non-empty `name` and an optional `icon`, THE App SHALL persist the Location to the `locations` table in SQLite_DB.
3. IF the user attempts to create a Location with an empty `name`, THEN THE App SHALL display an inline validation error without persisting the Location.
4. WHEN the user assigns a Location to a Container, THE Container Detail screen SHALL display the Location `name` and `icon` in the Container's metadata section.
5. WHEN the user deletes a Location, THE App SHALL set `location_id` to NULL for all Containers previously assigned to that Location in a single transaction.
6. THE Dashboard SHALL support filtering Root_Containers by Location via a Location filter chip row displayed above the Container grid.

---

### Requirement 11: Sistema de Diseño Visual (UI/UX)

**User Story:** As a user, I want a visually stunning, cinematic interface with premium visual identity, so that organizing my home feels like a Netflix/Apple-level experience rather than a chore.

#### Acceptance Criteria

1. THE Theme_Engine SHALL apply a dark color palette as the default theme with the following exact values: primary background `#0A0A0F`, surface color `#12121A`, and accent color `#6C63FF`.
2. THE Theme_Engine SHALL define a typography scale using the Inter_Variable font family with the following levels: Display (32px, weight 700, letter-spacing -0.5px), Heading (24px, weight 600, letter-spacing -0.3px), Title (18px, weight 600, letter-spacing -0.2px), Body (16px, weight 400, line-height 24px), Caption (12px, weight 400, letter-spacing 0.2px).
3. THE Theme_Engine SHALL implement glassmorphism card surfaces using a semi-transparent background (`rgba(255,255,255,0.05)`), a 1px border (`rgba(255,255,255,0.1)`), and a `blur(20px)` backdrop filter.
4. THE Theme_Engine SHALL define a shadow system with three named levels: `shadow-sm` (0 2px 8px rgba(0,0,0,0.4)), `shadow-md` (0 8px 24px rgba(0,0,0,0.5)), `shadow-lg` (0 16px 48px rgba(0,0,0,0.6)).
5. WHEN any screen transition occurs, THE Navigator SHALL execute a Shared_Element_Transition powered by React Native Reanimated with a duration of 350ms.
6. WHEN a Card is pressed, THE App SHALL apply a scale-down animation to scale 0.97 using a spring physics curve and trigger haptic feedback simultaneously.
7. THE App SHALL use a consistent 8px base spacing grid for all margins, paddings, and gaps throughout the interface.
8. WHILE data is being fetched from SQLite_DB, THE App SHALL display animated Shimmer_Effect skeleton placeholders in place of content, with a shimmer gradient moving from left to right with a loop duration of 1500ms.
9. THE App SHALL apply safe area insets on all screens to accommodate device notches and home indicators on both iOS and Android.
10. WHERE the device supports haptic feedback, THE App SHALL trigger haptic feedback on all primary user actions (create, delete, confirm).
11. THE Theme_Engine SHALL define a Glassmorphism_Level system with four named levels: `ultra-light` (background opacity 0.02, border opacity 0.06, blur 10px), `light` (background opacity 0.05, border opacity 0.10, blur 20px), `medium` (background opacity 0.10, border opacity 0.15, blur 30px), and `heavy` (background opacity 0.20, border opacity 0.25, blur 40px).
12. THE Theme_Engine SHALL define an expanded color palette with semantic variants: `primary` (`#6C63FF`), `secondary` (`#FF6584`), `tertiary` (`#43E97B`), `success` (`#43E97B`), `warning` (`#F7971E`), `error` (`#FF4757`), each with `default`, `light` (opacity 0.15), and `dark` (darkened 20%) variants.
13. THE Theme_Engine SHALL define a brand gradient from `#6C63FF` to `#FF6584` (accent to pink) used in featured elements, Gradient_Borders, and the Hero_Section.
14. THE Theme_Engine SHALL define a Noise_Texture overlay with 3% opacity applied over all glassmorphism surfaces to add visual depth.
15. WHEN an element is in active or selected state, THE App SHALL apply a Glow_Effect using a box shadow of `0 0 20px rgba(108,99,255,0.4)` around the element.
16. THE Theme_Engine SHALL define icon styles following SF_Symbols_Style conventions: rounded corners, filled variants for primary actions, and outline variants for secondary actions, using a consistent icon size grid of 16px, 20px, 24px, and 32px.
17. THE Theme_Engine SHALL define Gradient_Border styles for featured Cards using a 1px border rendered as a linear gradient from `rgba(108,99,255,0.6)` to `rgba(255,101,132,0.6)`.

---

### Requirement 12: Navegación de Pantallas

**User Story:** As a user, I want intuitive navigation between screens, so that I can move through the app without confusion.

#### Acceptance Criteria

1. THE Navigator SHALL implement a Bottom Tab Bar with exactly 4 tabs: Dashboard (home icon), Search (magnifier icon), Locations (map-pin icon), and Settings (gear icon).
2. THE Navigator SHALL implement a Stack Navigator nested inside the Dashboard tab supporting the navigation path: Dashboard → Container Detail → Item Detail.
3. WHEN the user navigates from the Dashboard to a Container Detail screen, THE Navigator SHALL animate the selected Container Card expanding into the full screen as a hero transition.
4. THE Navigator SHALL use modal presentation for the following screens: Create Container, Edit Container, Create Item, Edit Item, and Tag Manager.
5. WHEN the user presses the hardware back button on Android or performs a swipe-back gesture on iOS, THE Navigator SHALL return to the previous screen using the reverse of the entry animation.
6. THE Navigator SHALL preserve the scroll position and component state of each screen when the user navigates back to a previously visited screen within the same session.

---

### Requirement 13: Cámara e Imágenes

**User Story:** As a user, I want to add photos to my containers and items, so that I can visually identify them at a glance.

#### Acceptance Criteria

1. WHEN the user taps the cover image field in any form, THE Camera_Module SHALL present an action sheet with exactly two options: "Tomar foto" and "Elegir de galería".
2. WHEN the user selects "Tomar foto", THE Camera_Module SHALL request camera permission if not already granted and then open the device camera.
3. IF the user denies camera permission, THEN THE App SHALL display an informational alert that explains how to enable camera permission in the device's system settings.
4. WHEN the user captures or selects an image, THE Camera_Module SHALL resize the image to fit within a 1024×1024 pixel bounding box while preserving the aspect ratio, and save the result to the App's local file system using Expo File System.
5. THE Camera_Module SHALL store only the local file URI of the saved image in the SQLite_DB `cover_image_uri` field, not the raw binary image data.
6. IF the image save operation fails due to insufficient storage or a file system error, THEN THE Camera_Module SHALL display an error message to the user and leave the `cover_image_uri` field unchanged.

---

### Requirement 14: Exportación y Respaldo

**User Story:** As a user, I want to export and back up my data, so that I don't lose my organization information if I change devices.

#### Acceptance Criteria

1. THE Settings screen SHALL include an "Exportar datos" option that triggers the Export_Module.
2. WHEN the user triggers the Export_Module, THE App SHALL generate a JSON file containing all Containers, Items, Locations, and Tags from SQLite_DB, including all junction table relationships.
3. WHEN the JSON export file is generated, THE Export_Module SHALL open the native share sheet via Expo Sharing so the user can save or send the file.
4. THE Settings screen SHALL include an "Importar datos" option that allows the user to select a previously exported JSON file and restore its data into SQLite_DB.
5. IF the imported JSON file does not conform to the expected export schema, THEN THE Export_Module SHALL display an error message describing the specific validation failure without modifying the existing SQLite_DB data.
6. WHEN a valid database state is exported to a JSON file and that file is imported into a clean SQLite_DB, THE resulting database state SHALL contain the same Containers, Items, Locations, Tags, and junction table relationships as the original (round-trip property).

---

### Requirement 15: Rendimiento

**User Story:** As a user, I want the app to respond quickly to my interactions, so that the experience feels fluid and professional.

#### Acceptance Criteria

1. WHEN the Dashboard is opened with up to 200 Root_Containers, THE App SHALL render the initial Container grid within 100ms of the screen becoming active.
2. WHEN the user types a character in the search bar, THE Search_Engine SHALL return and display results within 300ms of the keystroke.
3. WHEN the user navigates between any two screens, THE Navigator SHALL complete the transition animation within 400ms.
4. THE App SHALL maintain a frame rate of at least 60 frames per second during all scroll and animation interactions on devices released in 2020 or later.
5. WHEN the App is launched from a cold start on a device released in 2020 or later, THE App SHALL display the Dashboard or Onboarding screen within 3 seconds.

---

### Requirement 16: Componentes Reutilizables

**User Story:** As a developer, I want a library of reusable UI components with premium visual effects, so that the app maintains visual consistency and development is efficient.

#### Acceptance Criteria

1. THE `components/ui` directory SHALL contain the following primitive components: `GlassCard` (glassmorphism surface con soporte de Glassmorphism_Level), `PosterCard` (tarjeta portrait 2:3 con múltiples capas visuales), `FAB` (floating action button con soporte de FAB_Radial_Menu), `BottomSheet` (modal sheet animado con glassmorphism background), `SkeletonLoader` (placeholder con Shimmer_Effect), `TagChip` (pill de etiqueta con color), `SearchBar` (input de búsqueda animado), `EmptyState` (ilustración Lottie_Animation + CTA), `QuantityStepper` (input numérico con botones de incremento y decremento), `HapticButton` (pressable con haptic feedback), `AnimatedCounter` (Count_Up_Animation para estadísticas), `GradientBorder` (contenedor con Gradient_Border), `GlowContainer` (contenedor con Glow_Effect), y `RibbonBadge` (ribbon decorativo para Containers recientes).
2. THE `GlassCard` component SHALL accept the following props: `blur` (number), `borderOpacity` (number 0–1), `backgroundOpacity` (number 0–1), `level` (Glassmorphism_Level: `ultra-light` | `light` | `medium` | `heavy`), y `children` (React node).
3. THE `PosterCard` component SHALL accept the following props: `imageUri` (string or null), `title` (string), `subtitle` (string or null), `badge` (string or null), `onPress` (function), `onLongPress` (function), `showRibbon` (boolean para mostrar RibbonBadge), `dominantColor` (string hexadecimal para gradiente dinámico), y `featured` (boolean para aplicar Gradient_Border).
4. THE `BottomSheet` component SHALL accept `snapPoints` (array of pixel heights or percentage strings), an `onClose` callback, y SHALL support an animated drag-to-dismiss gesture, con glassmorphism background de nivel `medium`.
5. THE `SkeletonLoader` component SHALL animate a Shimmer_Effect gradient moving from left to right with a loop duration of 1500ms using React Native Reanimated.
6. WHEN any primitive component receives a missing required prop, THE App SHALL emit a console warning in development mode without throwing an error or crashing in production mode.
7. THE `PosterCard` component SHALL render with an aspect ratio of 2:3 (portrait poster) and SHALL display the following visual layers in order from bottom to top: (1) Cover_Image o gradiente de fondo, (2) gradiente lineal de transparente a negro en el tercio inferior, (3) overlay glassmorphism de nivel `ultra-light`, (4) Vignette_Effect en los bordes, (5) texto del título y badge.
8. WHEN the user presses a `PosterCard`, THE Animation_Engine SHALL apply a tilt effect (inclinación 3D sutil de máximo 5 grados en el eje X e Y) using a spring physics curve with damping 15 and stiffness 300.
9. THE `FAB` component SHALL support a `radialMenu` prop (array de hasta 4 opciones con icono y label) que al activarse despliega las opciones en arco con animación de morph de 250ms y spring physics.
10. THE `AnimatedCounter` component SHALL accept `value` (number), `duration` (number, default 800ms), y `prefix`/`suffix` (string), y SHALL animate the displayed number from 0 to `value` using an easing curve `easeOutExpo` when the component enters the viewport.
11. THE `EmptyState` component SHALL accept a `lottieSource` prop (Lottie JSON asset) en lugar de un React node genérico, y SHALL loop the animation continuously while the empty state is visible.
12. THE `RibbonBadge` component SHALL display a diagonal ribbon in the top-right corner of a Card with the text "Reciente" using the brand gradient from `#6C63FF` to `#FF6584`.

---

### Requirement 17: Accesibilidad

**User Story:** As a user with accessibility needs, I want the app to be usable with assistive technologies, so that I can organize my home regardless of my abilities.

#### Acceptance Criteria

1. THE App SHALL assign `accessibilityLabel` and `accessibilityHint` props to all interactive elements, including buttons, cards, and text inputs.
2. THE App SHALL support Dynamic Type font scaling on iOS and the system font size preference on Android without truncating or overlapping text in any layout.
3. WHEN the user navigates with VoiceOver on iOS or TalkBack on Android, THE App SHALL announce screen titles and interactive element descriptions in Spanish.
4. THE App SHALL maintain a minimum color contrast ratio of 4.5:1 between all text elements and their background colors as defined in the Theme_Engine.
5. THE App SHALL not rely solely on color to convey information; icons or text labels SHALL accompany all color-coded elements, including Tags and Container type indicators.

---

### Requirement 18: Diseño Cinematográfico

**User Story:** As a user, I want a cinematic visual experience with depth and motion, so that navigating the app feels like watching a premium film rather than using a utility tool.

#### Acceptance Criteria

1. THE Dashboard SHALL implement Parallax_Scroll in the Hero_Section so that the Cover_Image of the featured Container moves at 60% of the scroll speed of the foreground content, creating a depth effect.
2. WHEN the user taps a Poster_Card on the Dashboard, THE Navigator SHALL execute a Shared_Element_Transition where the Card expands from its position and size to fill the full screen as the Container Detail hero, with a duration of 350ms using a spring physics curve with damping 20 and stiffness 200.
3. WHEN a screen transition occurs between any two screens, THE Navigator SHALL apply a blur transition that increases the blur of the departing screen from 0 to 10px while the arriving screen fades in, with a total duration of 300ms.
4. WHEN a Container has a Cover_Image, THE Container Detail hero SHALL display a cinematic letterbox effect with black bars of 40px height at the top and bottom of the image, applied as a gradient overlay from black to transparent.
5. THE Hero_Section on the Dashboard SHALL implement an automatic carousel that rotates through the 5 most recently updated Containers every 5 seconds, with a crossfade transition of 800ms between slides.
6. WHEN the user navigates from a Container Detail screen back to the Dashboard, THE Navigator SHALL execute the reverse of the Shared_Element_Transition, collapsing the hero back into the originating Card position.

---

### Requirement 19: Cards Tipo Poster Expandidas

**User Story:** As a user, I want visually rich poster-style cards with depth and animation, so that each container feels like a premium content item worth organizing.

#### Acceptance Criteria

1. THE Poster_Card component SHALL render with an aspect ratio of exactly 2:3 (portrait poster format), with a minimum width of 140px and a maximum width of 200px in grid view.
2. THE Poster_Card component SHALL display the following visual layers in order from bottom to top: (1) Cover_Image o gradiente de fondo a pantalla completa, (2) gradiente lineal de `transparent` a `rgba(0,0,0,0.85)` en el 50% inferior, (3) overlay glassmorphism de nivel `ultra-light` sobre toda la superficie, (4) Vignette_Effect con radio de oscurecimiento del 30% desde cada borde, (5) texto del título en tipografía Title (18px, weight 600) y badge de conteo de Items en la esquina superior derecha.
3. WHEN the user presses and holds a Poster_Card, THE Animation_Engine SHALL apply a Shimmer_Effect that sweeps across the card surface once in 600ms, triggered by the press gesture.
4. THE Poster_Card component SHALL display an animated badge in the top-right corner showing the Item count, with a scale pulse animation (scale 1.0 → 1.2 → 1.0) of 300ms duration when the count changes.
5. WHEN a Container was accessed within the last 24 hours, THE Poster_Card SHALL display a RibbonBadge in the top-right corner with the text "Reciente" using the brand gradient.
6. THE Poster_Card component SHALL support a `featured` prop that, when true, applies a Gradient_Border of 1.5px width using the brand gradient from `#6C63FF` to `#FF6584`.
7. WHEN the user applies a long press on a Poster_Card, THE Animation_Engine SHALL apply a tilt effect of maximum 5 degrees on both X and Y axes simultaneously, with spring physics (damping 15, stiffness 300), and trigger a medium haptic feedback.

---

### Requirement 20: Dashboard Visual Expandido

**User Story:** As a user, I want a rich, multi-section dashboard with Netflix-style content rows, so that I can discover and access my containers in an engaging and intuitive way.

#### Acceptance Criteria

1. THE Dashboard SHALL display a Hero_Section as the first element, occupying the full screen width and 55% of the screen height, showing the most recently updated Root_Container with its Cover_Image, Parallax_Scroll effect, and the brand gradient overlay.
2. THE Dashboard SHALL display a "Continuar organizando" section showing Root_Containers that were opened in the current session but not fully reviewed, displayed as a horizontal scroll row of compact Poster_Cards with a "En progreso" label.
3. THE Dashboard SHALL display a "Recientes" section as a horizontally scrollable row of Poster_Cards showing the 8 most recently updated Root_Containers, styled identically to a Netflix content row with a section title in Heading typography.
4. THE Dashboard SHALL display a "Por ubicación" section with one horizontally scrollable row per Location, each row labeled with the Location name and icon, showing all Root_Containers assigned to that Location as Poster_Cards.
5. THE Dashboard summary statistics (total Containers, total Items, Locations in use) SHALL animate with Count_Up_Animation when the Dashboard screen first becomes visible, incrementing from 0 to the actual value over 800ms using an `easeOutExpo` curve.
6. THE Dashboard SHALL display a mini activity bar chart below the summary statistics showing the number of Items added per day over the last 7 days, with bars that animate from height 0 to their final height over 600ms when the section enters the viewport.
7. WHEN the user scrolls past the Hero_Section, THE Dashboard navigation bar SHALL transition from fully transparent to a semi-transparent glassmorphism surface (level `medium`) with a smooth opacity animation of 200ms.
8. THE Dashboard SHALL display a Mesh_Gradient background that animates continuously with a cycle duration of 8 seconds, interpolating between `#0A0A0F`, `rgba(108,99,255,0.15)`, and `rgba(255,101,132,0.10)` as gradient color stops.

---

### Requirement 21: Estadísticas Modernas

**User Story:** As a user, I want a dedicated statistics screen with modern data visualizations, so that I can understand the composition and activity of my home organization at a glance.

#### Acceptance Criteria

1. THE App SHALL provide a Statistics screen accessible from the Dashboard summary bar or the Settings tab, displaying all statistics with animated entry transitions.
2. THE Statistics_Screen SHALL display a donut chart showing the total number of Items grouped by Container type (box, suitcase, drawer, shelf, bag, other), with each segment colored using the expanded color palette and a Count_Up_Animation for the center total value.
3. THE Statistics_Screen SHALL display a bar chart showing the number of Items added per day over the last 7 days, with bars animated from height 0 to their final height over 600ms using a staggered delay of 80ms per bar.
4. THE Statistics_Screen SHALL display a list of the 5 most-filled Containers (by Item count relative to a configurable capacity), each shown as an animated progress bar that fills from 0% to the actual percentage over 800ms when the section enters the viewport.
5. THE Statistics_Screen SHALL display a location heat map showing a grid of Location tiles, each colored with an intensity gradient from `rgba(108,99,255,0.1)` (few items) to `rgba(108,99,255,0.9)` (many items) based on the total Item count in that Location.
6. THE Statistics_Screen SHALL display the total number of Items across all Containers as a large Count_Up_Animation in Display typography (32px, weight 700) at the top of the screen.
7. WHEN the Statistics_Screen is first displayed, THE Animation_Engine SHALL animate all chart elements with a staggered entrance: the total counter first (0ms delay), then the donut chart (200ms delay), then the bar chart (400ms delay), then the progress bars (600ms delay), then the heat map (800ms delay).
8. THE Statistics_Screen SHALL update all displayed values in real time when the user navigates back to it after creating or deleting Items or Containers.

---

### Requirement 22: Animaciones Suaves y Microinteracciones

**User Story:** As a user, I want smooth animations and delightful micro-interactions throughout the app, so that every interaction feels responsive, polished, and satisfying.

#### Acceptance Criteria

1. THE FAB_Radial_Menu SHALL animate with a morph effect when opened: the FAB button morphs from a circle to a larger circle while the radial menu options emerge in an arc pattern with a staggered delay of 50ms per option, using spring physics (damping 12, stiffness 250).
2. WHEN the user presses any primary button, THE Animation_Engine SHALL apply a ripple effect that expands from the touch point outward, using the button's accent color at 30% opacity, with a duration of 400ms.
3. WHEN the user presses a Poster_Card, THE Animation_Engine SHALL apply a tilt effect of maximum 5 degrees on both X and Y axes based on the touch position relative to the card center, using spring physics (damping 15, stiffness 300).
4. WHEN the user performs a pull-to-refresh gesture on the Dashboard, THE App SHALL display a custom pull-to-refresh animation showing the App logo rotating and pulsing, replacing the default system spinner.
5. WHEN the user swipes left on an Item row, THE Animation_Engine SHALL reveal the Delete and Edit action buttons with spring physics (damping 20, stiffness 400), and the row background SHALL transition to a red tint (`rgba(255,71,87,0.2)`) for Delete and a blue tint (`rgba(108,99,255,0.2)`) for Edit.
6. WHEN the user performs a long press on any interactive element, THE App SHALL trigger a medium haptic feedback at the start of the long press and a heavy haptic feedback when the context menu appears, simultaneously applying a scale animation from 1.0 to 0.95 during the press.
7. THE Tab Bar SHALL display a Blob_Tab_Indicator that moves fluidly between tabs when the active tab changes, using a spring physics animation (damping 18, stiffness 350) with the blob shape morphing to match the width of the active tab label.
8. WHEN the user navigates between tabs, THE Animation_Engine SHALL animate the Blob_Tab_Indicator from the previous tab position to the new tab position in a single continuous motion, without teleporting.
9. WHEN a new Container or Item Card is added to a list, THE Animation_Engine SHALL animate the new Card entering from below with a fade-in and slide-up animation of 300ms duration, pushing existing cards aside with spring physics.

---

### Requirement 23: Loaders Elegantes

**User Story:** As a user, I want elegant loading states and transitions, so that waiting for data never feels abrupt or jarring.

#### Acceptance Criteria

1. WHILE data is being fetched from SQLite_DB, THE App SHALL display Shimmer_Effect skeleton placeholders that match the exact layout and dimensions of the content they replace, including Poster_Card skeletons in the Dashboard grid and list item skeletons in Container Detail screens.
2. THE Shimmer_Effect SHALL animate a gradient from `rgba(255,255,255,0.0)` through `rgba(255,255,255,0.08)` back to `rgba(255,255,255,0.0)` moving from left to right with a loop duration of 1500ms using React Native Reanimated.
3. WHEN the App is launched for the first time or after a cold start, THE App SHALL display a Splash_Screen with the App logo animating with a scale-in effect (from 0.5 to 1.0) and a fade-in of 600ms, followed by a fade-out transition to the Dashboard or Onboarding screen.
4. THE App SHALL use a custom circular spinner with the accent color `#6C63FF` and a trailing gradient tail, replacing all default system spinners, with a rotation speed of one full rotation per 800ms.
5. WHEN the Export_Module or Import_Module is processing data, THE App SHALL display a progress indicator showing the percentage of completion as a linear progress bar with the brand gradient fill, animated smoothly as each record is processed.
6. WHEN a screen has no content to display (empty state), THE App SHALL display a Lottie_Animation relevant to the context (e.g., an empty box animation for empty Container lists, a magnifier animation for empty search results), looping continuously.
7. THE Splash_Screen animation SHALL complete within 1.5 seconds from app launch, after which the App SHALL transition to the Dashboard or Onboarding screen.

---

### Requirement 24: Gradientes y Efectos Visuales

**User Story:** As a user, I want rich gradients and visual effects throughout the app, so that the interface has depth, personality, and a premium aesthetic.

#### Acceptance Criteria

1. WHEN a Container has a Cover_Image, THE App SHALL extract the Dominant_Color from the image using a color sampling algorithm and generate a dynamic gradient from the Dominant_Color to `rgba(10,10,15,1.0)` (background color) for use as the gradient overlay on that Container's Poster_Card and Detail screen hero.
2. THE Dashboard SHALL display a Mesh_Gradient background that continuously animates between three color stops: `#0A0A0F` (base background), `rgba(108,99,255,0.12)` (accent), and `rgba(255,101,132,0.08)` (secondary), with each color stop moving along a bezier path with a cycle duration of 8 seconds.
3. THE App SHALL apply a Noise_Texture overlay with 3% opacity over all glassmorphism surfaces to add visual depth and prevent flat appearance on OLED screens.
4. WHEN an element is in active or selected state (e.g., selected Tag chip, active tab, focused input), THE App SHALL apply a Glow_Effect using a box shadow of `0 0 20px rgba(108,99,255,0.4)` and `0 0 40px rgba(108,99,255,0.2)` around the element.
5. THE App SHALL use `expo-blur` to implement real frosted glass panels for the Tab Bar, modal sheets, and navigation bar, with a blur intensity of 80 on iOS and a simulated semi-transparent background on Android.
6. WHEN a Container has a Cover_Image, THE Poster_Card and Container Detail hero SHALL apply a Vignette_Effect implemented as a radial gradient from `transparent` at the center to `rgba(0,0,0,0.6)` at the edges, with a radius of 70% of the card dimensions.
7. THE App SHALL apply the brand gradient (`#6C63FF` → `#FF6584`) to the following elements: FAB button background, primary action buttons, featured Card Gradient_Borders, Statistics_Screen chart accent colors, and the Splash_Screen logo.
8. WHEN the user scrolls horizontally in a Netflix-style row, THE App SHALL apply a fade-out gradient on the right edge of the row (from transparent to the background color over 40px) to indicate more content is available.

---

### Requirement 25: Glassmorphism Expandido

**User Story:** As a user, I want consistent and layered glassmorphism effects throughout the app, so that the interface feels cohesive, modern, and visually sophisticated.

#### Acceptance Criteria

1. THE Theme_Engine SHALL implement four Glassmorphism_Level variants with the following exact values:
   - `ultra-light`: background `rgba(255,255,255,0.02)`, border `rgba(255,255,255,0.06)`, blur 10px
   - `light`: background `rgba(255,255,255,0.05)`, border `rgba(255,255,255,0.10)`, blur 20px
   - `medium`: background `rgba(255,255,255,0.10)`, border `rgba(255,255,255,0.15)`, blur 30px
   - `heavy`: background `rgba(255,255,255,0.20)`, border `rgba(255,255,255,0.25)`, blur 40px
2. THE Tab Bar SHALL implement glassmorphism using `expo-blur` with a blur intensity of 80, displaying the content behind the Tab Bar blurred through the bar surface, with a Glassmorphism_Level of `medium`.
3. WHEN a modal sheet (BottomSheet) is presented, THE App SHALL apply a glassmorphism background of Glassmorphism_Level `heavy` to the modal surface, and blur the content behind the modal using `expo-blur` with intensity 60.
4. THE App SHALL display floating Cards (e.g., the Hero_Section overlay, the summary statistics bar) with glassmorphism of Glassmorphism_Level `light` and a colored shadow using the Dominant_Color of the featured Container at 30% opacity.
5. THE navigation bar SHALL implement glassmorphism of Glassmorphism_Level `medium` when the user has scrolled past the Hero_Section, transitioning from fully transparent using a smooth opacity animation of 200ms.
6. THE Context_Menu_Preview SHALL display with a glassmorphism background of Glassmorphism_Level `heavy` and a Noise_Texture overlay, with a border-radius of 16px and a Gradient_Border.

---

### Requirement 26: Identidad Visual Premium

**User Story:** As a user, I want a cohesive and premium visual identity throughout the app, so that San Alejo feels like a professionally designed product with a distinctive personality.

#### Acceptance Criteria

1. THE Theme_Engine SHALL define the complete expanded color palette with the following exact values and their variants:
   - `primary`: `#6C63FF` (light: `rgba(108,99,255,0.15)`, dark: `#4B44CC`)
   - `secondary`: `#FF6584` (light: `rgba(255,101,132,0.15)`, dark: `#CC4466`)
   - `tertiary`: `#43E97B` (light: `rgba(67,233,123,0.15)`, dark: `#2ECC5A`)
   - `success`: `#43E97B`
   - `warning`: `#F7971E` (light: `rgba(247,151,30,0.15)`, dark: `#CC7A10`)
   - `error`: `#FF4757` (light: `rgba(255,71,87,0.15)`, dark: `#CC2233`)
2. THE Theme_Engine SHALL load the Inter_Variable font from the `@expo-google-fonts/inter` package, supporting weight values from 100 to 900 and applying the following micro-typography settings: Display (-0.5px letter-spacing), Heading (-0.3px letter-spacing), Title (-0.2px letter-spacing), Body (24px line-height), Caption (+0.2px letter-spacing).
3. THE App SHALL use SF_Symbols_Style icons from the `@expo/vector-icons` Ionicons set, applying rounded variants for all icons, with filled variants for active/primary states and outline variants for inactive/secondary states.
4. THE Theme_Engine SHALL define Gradient_Border styles for featured elements using a 1.5px border rendered as a linear gradient from `rgba(108,99,255,0.6)` to `rgba(255,101,132,0.6)`, applied to: featured Poster_Cards, the active tab indicator, primary modal headers, and the Statistics_Screen chart containers.
5. THE App SHALL maintain a consistent border-radius scale: `xs` (4px), `sm` (8px), `md` (12px), `lg` (16px), `xl` (24px), `full` (9999px), applied consistently across all components.
6. THE App SHALL apply the brand gradient (`#6C63FF` → `#FF6584`) as a text gradient on the App name/logo displayed in the Onboarding, Splash_Screen, and Settings screen header.
7. THE Theme_Engine SHALL define a `coloredShadow` utility that generates a box shadow using a provided color at 40% opacity with a spread of 0 and blur of 20px, used for floating elements to create a colored glow shadow matching the element's accent color.

---

### Requirement 27: Experiencia Tipo Netflix/Apple

**User Story:** As a user, I want a Netflix and Apple-inspired navigation and content discovery experience, so that finding and organizing my containers feels as intuitive and engaging as browsing a streaming service.

#### Acceptance Criteria

1. THE Dashboard SHALL display a Hero_Section carousel that automatically rotates through the 5 most recently updated Root_Containers every 5 seconds, with a crossfade transition of 800ms and pagination dots below the hero indicating the current slide.
2. THE Dashboard SHALL display a "Continuar organizando" section showing Root_Containers that were opened in the current session, displayed as a compact horizontal scroll row with a progress indicator showing how many Items were reviewed.
3. THE Dashboard SHALL organize content in horizontally scrollable category rows (Netflix rows), each with a section title in Heading typography, a "Ver todo" link, and Poster_Cards that can be scrolled horizontally with momentum scrolling.
4. WHEN the user taps a Poster_Card to navigate to Container Detail, THE Navigator SHALL execute a zoom transition where the Card scales up and the background blurs simultaneously, with the Card expanding to fill the screen over 350ms using spring physics.
5. WHEN the user scrolls down on any screen with a large title (Dashboard, Container Detail), THE Navigator SHALL animate the navigation bar title from large (Display typography, 32px) to compact (Title typography, 18px) as the user scrolls past the title position, matching iOS large title behavior.
6. WHEN the user performs a long press on a Poster_Card, THE App SHALL display a Context_Menu_Preview showing a scaled-down preview of the Container Detail screen above the context menu options, with a blur background behind the preview, matching iOS peek and pop behavior.
7. THE Tab Bar SHALL use glassmorphism (Glassmorphism_Level `medium`) with `expo-blur`, display the Blob_Tab_Indicator for the active tab, and animate tab switches with the blob moving fluidly between positions using spring physics (damping 18, stiffness 350).
8. WHEN the user swipes down on a modal sheet (BottomSheet), THE App SHALL dismiss the modal with a spring physics animation (damping 25, stiffness 300) and restore the background content with a reverse blur transition of 200ms.
9. THE App SHALL implement momentum scrolling with deceleration rate `fast` on all horizontal Netflix-style rows, and `normal` on all vertical lists, matching native iOS scroll behavior.
10. WHEN the user navigates to the Container Detail screen, THE App SHALL display the Container name as a large title that collapses into the navigation bar as the user scrolls, with the Cover_Image hero parallax-scrolling at 50% of the scroll speed.
