# Implementation Plan: San Alejo App

## Overview

Plan de implementación incremental para San Alejo, una app móvil de organización del hogar con interfaz visual premium (React Native + Expo + TypeScript). Las tareas siguen el roadmap de 4 fases definido en los requisitos, construyendo desde la fundación hasta el pulido final. Cada tarea referencia los requisitos específicos que implementa.

**Stack**: React Native · Expo SDK · TypeScript (strict) · Expo SQLite · React Navigation v6 · Zustand · Reanimated v3 · NativeWind · fast-check · Jest · RNTL

---

## Tasks

- [ ] 1. Phase 1 — Foundation: Configuración del proyecto y estructura base
  - Inicializar proyecto Expo con TypeScript strict: `npx create-expo-app san-alejo --template expo-template-blank-typescript`
  - Instalar todas las dependencias del stack: `expo-sqlite`, `@react-navigation/native`, `@react-navigation/stack`, `@react-navigation/bottom-tabs`, `zustand`, `react-native-reanimated`, `nativewind`, `expo-image-picker`, `expo-file-system`, `expo-sharing`, `react-native-gesture-handler`, `expo-haptics`, `fast-check`, `jest-expo`, `@testing-library/react-native`
  - Configurar `tsconfig.json` con `strict: true`, `paths` aliases para `@/` apuntando a `src/`
  - Configurar `babel.config.js` para NativeWind, Reanimated y path aliases
  - Configurar `jest.config.js` con preset `jest-expo` y módulos de transformación
  - Crear la estructura de carpetas completa bajo `/src` según el diseño: `components/common`, `components/containers`, `components/items`, `components/ui`, `screens/dashboard`, `screens/containers`, `screens/items`, `screens/search`, `screens/settings`, `screens/onboarding`, `navigation`, `store`, `database/repositories`, `hooks`, `utils`, `assets`, `theme`, `types`, `src/__tests__/properties`, `src/__tests__/unit`, `src/__tests__/integration`, `src/__tests__/components`
  - _Requirements: 1.1, 1.2, 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 15.1_

- [ ] 2. Phase 1 — Foundation: Sistema de tipos TypeScript
  - [ ] 2.1 Crear interfaces de dominio en `src/types/`
    - Crear `src/types/common.ts` con `UUID`, `UnixTimestamp`, `ContainerType`
    - Crear `src/types/Location.ts` con interfaz `Location`
    - Crear `src/types/Tag.ts` con interfaz `Tag`
    - Crear `src/types/Container.ts` con interfaces `Container`, `CreateContainerInput`, `UpdateContainerInput`
    - Crear `src/types/Item.ts` con interfaces `Item`, `CreateItemInput`, `UpdateItemInput`
    - Crear `src/types/index.ts` re-exportando todos los tipos
    - _Requirements: 2.6, 3.1, 3.2, 3.3, 3.4_

  - [ ]* 2.2 Escribir tests de ejemplo para tipos y validaciones
    - Verificar que `ContainerType` solo acepta los 6 valores válidos
    - Verificar que `CreateContainerInput` requiere `name` y `type`
    - _Requirements: 2.6_

- [ ] 3. Phase 1 — Foundation: Theme Engine
  - [ ] 3.1 Implementar tokens del sistema de diseño
    - Crear `src/theme/colors.ts` con paleta oscura: `background: '#0A0A0F'`, `surface: '#12121A'`, `accent: '#6C63FF'`, colores de texto, bordes y gradientes
    - Crear `src/theme/typography.ts` con escala Inter: Display (32/700), Heading (24/600), Title (18/600), Body (16/400), Caption (12/400)
    - Crear `src/theme/spacing.ts` con grid de 8px: `xs:4`, `sm:8`, `md:16`, `lg:24`, `xl:32`, `xxl:48`
    - Crear `src/theme/shadows.ts` con tres niveles: `shadow-sm`, `shadow-md`, `shadow-lg`
    - Crear `src/theme/animations.ts` con constantes de duración (350ms transiciones), spring configs y easing curves
    - Crear `src/theme/index.ts` exportando el objeto `theme` unificado
    - _Requirements: 11.1, 11.2, 11.4, 11.7_

  - [ ]* 3.2 Escribir property test para contraste de colores (Property 13)
    - **Property 13: Theme color contrast ratio ≥ 4.5:1**
    - Para cualquier par (texto, fondo) definido en `colors.ts`, calcular ratio WCAG y verificar ≥ 4.5:1
    - **Validates: Requirements 11.4, 17.4**
    - _Archivo: `src/__tests__/properties/theme.property.test.ts`_

- [ ] 4. Phase 1 — Foundation: Base de datos SQLite — schema y migraciones
  - [ ] 4.1 Implementar singleton de conexión y schema
    - Crear `src/database/db.ts` con singleton `SQLiteDatabase` usando `expo-sqlite`, función `getDb()` y función `initializeDb()` que ejecuta schema + seed
    - Crear `src/database/schema.ts` con todas las sentencias `CREATE TABLE IF NOT EXISTS` para: `locations`, `containers`, `items`, `tags`, `container_tags`, `item_tags`, y tabla `_migrations`
    - Implementar seed de Location "Sin ubicación" en `initializeDb()` (solo si no existe)
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7_

  - [ ] 4.2 Implementar sistema de migraciones
    - Crear `src/database/migrations.ts` con interfaz `Migration { version, up }` y función `runMigrations(db)`
    - Implementar lógica de migración incremental: leer versión actual de `_migrations`, aplicar solo migraciones pendientes en orden
    - _Requirements: 3.8_

  - [ ]* 4.3 Escribir tests de integración para inicialización de DB
    - Verificar que todas las tablas se crean correctamente en DB in-memory
    - Verificar que el seed de "Sin ubicación" se inserta en primer lanzamiento
    - Verificar que el seed no se duplica en lanzamientos posteriores
    - _Archivo: `src/__tests__/integration/database.test.ts`_
    - _Requirements: 3.7_

  - [ ]* 4.4 Escribir property test para preservación de datos en migraciones (Property 4)
    - **Property 4: Migration preserves existing data**
    - Para cualquier estado válido de DB, aplicar migración incremental debe preservar todos los registros existentes
    - **Validates: Requirements 3.8**
    - _Archivo: `src/__tests__/properties/container.property.test.ts`_

- [ ] 5. Phase 1 — Foundation: Repositorios de datos
  - [ ] 5.1 Implementar `ContainerRepository`
    - Crear `src/database/repositories/ContainerRepository.ts` implementando `IContainerRepository`
    - Métodos: `findAll`, `findRoots`, `findById`, `findByParentId`, `findByTagIds`, `findByLocationId`, `create` (genera UUID, timestamps), `update`, `delete` (cascading via FK), `countItems` (recursivo), `search`
    - Lanzar errores de dominio tipados: `ContainerNotFoundError`, `InvalidContainerTypeError`
    - _Requirements: 3.1, 6.3, 6.8, 6.10_

  - [ ]* 5.2 Escribir property test para round-trip de Container (Property 1)
    - **Property 1: Container creation round-trip**
    - Para cualquier input válido (nombre no vacío, tipo válido), crear y leer por ID debe producir objeto equivalente
    - **Validates: Requirements 3.9, 6.3**
    - _Archivo: `src/__tests__/properties/container.property.test.ts`_

  - [ ]* 5.3 Escribir property test para delete en cascada (Property 5)
    - **Property 5: Cascading delete removes all descendants**
    - Para cualquier árbol de Containers con Items, eliminar el raíz debe dejar la DB sin ningún descendiente ni Item
    - **Validates: Requirements 6.8**
    - _Archivo: `src/__tests__/properties/container.property.test.ts`_

  - [ ]* 5.4 Escribir property test para conteo recursivo de Items (Property 6)
    - **Property 6: Recursive item count invariant**
    - `countItems(rootId)` debe igualar la suma de todos los Items en el subárbol a cualquier profundidad
    - **Validates: Requirements 6.10**
    - _Archivo: `src/__tests__/properties/container.property.test.ts`_

  - [ ] 5.5 Implementar `ItemRepository`
    - Crear `src/database/repositories/ItemRepository.ts` implementando `IItemRepository`
    - Métodos: `findByContainerId`, `findById`, `create`, `update`, `delete`, `move`, `search`
    - _Requirements: 3.2, 7.3, 7.8_

  - [ ]* 5.6 Escribir property test para round-trip de Item (Property 2)
    - **Property 2: Item creation round-trip**
    - Para cualquier input válido (nombre no vacío, quantity ≥ 1, container_id existente), crear y leer debe producir objeto equivalente
    - **Validates: Requirements 7.3**
    - _Archivo: `src/__tests__/properties/item.property.test.ts`_

  - [ ]* 5.7 Escribir property test para mover Item (Property 7)
    - **Property 7: Move item updates container membership**
    - Mover un Item debe aparecer en destino y desaparecer de origen con `container_id` actualizado
    - **Validates: Requirements 7.8**
    - _Archivo: `src/__tests__/properties/item.property.test.ts`_

  - [ ] 5.8 Implementar `LocationRepository`
    - Crear `src/database/repositories/LocationRepository.ts`
    - Métodos: `findAll`, `findById`, `create`, `update`, `delete` (nullifica `location_id` en containers)
    - Lanzar `LocationNotFoundError` cuando corresponda
    - _Requirements: 3.3, 10.2, 10.4_

  - [ ]* 5.9 Escribir property test para delete de Location (Property 12)
    - **Property 12: Delete location nullifies container location_id**
    - Para cualquier Location con N Containers asignados, eliminarla debe dejar todos esos Containers con `location_id = NULL` sin modificar otros campos
    - **Validates: Requirements 10.4**
    - _Archivo: `src/__tests__/properties/tags.property.test.ts`_

  - [ ] 5.10 Implementar `TagRepository`
    - Crear `src/database/repositories/TagRepository.ts`
    - Métodos: `findAll`, `findById`, `create`, `update`, `delete`, `assignToContainer`, `assignToItem`, `removeFromContainer`, `removeFromItem`, `findByContainerId`, `findByItemId`
    - Lanzar `DuplicateTagNameError` en nombre duplicado
    - _Requirements: 3.4, 3.5, 3.6, 9.1, 9.2_

  - [ ]* 5.11 Escribir property test para asignación idempotente de Tags (Property 11)
    - **Property 11: Tag assignment persists in junction table**
    - Asignar el mismo Tag dos veces no debe crear duplicados en la junction table
    - **Validates: Requirements 9.2**
    - _Archivo: `src/__tests__/properties/tags.property.test.ts`_

- [ ] 6. Phase 1 — Foundation: Validaciones y utilidades
  - [ ] 6.1 Implementar funciones de validación
    - Crear `src/utils/validation.ts` con `validateContainerName`, `validateItemName`, `validateQuantity`, `validateExportJSON`
    - _Requirements: 6.4, 7.4, 14.5_

  - [ ]* 6.2 Escribir property test para validación de nombres (Property 3)
    - **Property 3: Name validation rejects empty and whitespace inputs**
    - Para cualquier cadena de solo whitespace (vacía, espacios, tabs, newlines), la validación debe retornar error y no modificar la DB
    - **Validates: Requirements 6.4, 7.4**
    - _Archivo: `src/__tests__/properties/container.property.test.ts`_

  - [ ]* 6.3 Escribir tests de ejemplo para validaciones
    - Casos: nombre válido, nombre vacío, nombre solo espacios, nombre de 101 chars, quantity 0, quantity negativa, quantity decimal
    - _Archivo: `src/__tests__/unit/validation.test.ts`_
    - _Requirements: 6.4, 7.4_

  - [ ] 6.4 Implementar utilidades generales
    - Crear `src/utils/uuid.ts` con función `generateUUID()` usando `crypto.randomUUID()` o polyfill
    - Crear `src/utils/dateUtils.ts` con `nowTimestamp(): UnixTimestamp` y `formatDate(ts: UnixTimestamp): string`
    - _Requirements: 3.1, 3.2_

- [ ] 7. Phase 1 — Foundation: Stores Zustand
  - [ ] 7.1 Implementar `containerStore`
    - Crear `src/store/containerStore.ts` con estado `containers`, `loading`, `error` y acciones: `fetchRootContainers`, `fetchContainerById`, `createContainer`, `updateContainer`, `deleteContainer`, `moveContainer`
    - Implementar optimistic updates: actualizar store inmediatamente, persistir en background
    - Envolver todas las acciones async en `try/catch`, almacenar errores en `store.error`
    - _Requirements: 5.1, 6.3, 6.7, 6.8_

  - [ ] 7.2 Implementar `itemStore`
    - Crear `src/store/itemStore.ts` con estado `itemsByContainer`, `loading` y acciones: `fetchItemsByContainer`, `createItem`, `updateItem`, `deleteItem`, `moveItem`
    - _Requirements: 7.3, 7.8_

  - [ ] 7.3 Implementar `tagStore` y `locationStore`
    - Crear `src/store/tagStore.ts` con acciones CRUD de Tags y asignación a Container/Item
    - Crear `src/store/locationStore.ts` con acciones CRUD de Locations
    - _Requirements: 9.1, 9.2, 10.2_

  - [ ] 7.4 Implementar `uiStore`
    - Crear `src/store/uiStore.ts` con estado: `dashboardViewMode: 'grid' | 'list'`, `selectedTags: string[]`, `selectedLocationId: string | null`, `onboardingCompleted: boolean`
    - _Requirements: 4.3, 5.6_

- [ ] 8. Phase 1 — Foundation: Navegación skeleton
  - [ ] 8.1 Implementar Bottom Tab Navigator y Stack Navigator
    - Crear `src/navigation/BottomTabNavigator.tsx` con 4 tabs: Dashboard (home), Search (search), Locations (map-pin), Settings (settings)
    - Crear `src/navigation/DashboardStack.tsx` con Stack: DashboardScreen → ContainerDetailScreen → ItemDetailScreen
    - Crear `src/navigation/RootNavigator.tsx` que decide entre OnboardingStack y MainApp según `onboardingCompleted`
    - Aplicar estilos del Theme Engine al Tab Bar (fondo `surface`, tint `accent`)
    - _Requirements: 12.1, 12.2, 12.4_

  - [ ] 8.2 Crear pantallas placeholder para verificar navegación
    - Crear pantallas stub para: `DashboardScreen`, `ContainerDetailScreen`, `SearchScreen`, `LocationsScreen`, `SettingsScreen`
    - Verificar que la app corre en iOS y Android simulators con navegación funcional
    - _Requirements: 15.2_

- [ ] 9. Phase 1 — Foundation: Onboarding
  - [ ] 9.1 Implementar pantalla de Onboarding con 3 slides
    - Crear `src/screens/onboarding/OnboardingScreen.tsx` con 3 slides usando `FlatList` horizontal paginado
    - Slide 1: logo + tagline de la app
    - Slide 2: explicación de Containers e Items con ilustración
    - Slide 3: call-to-action para crear el primer Container
    - Implementar indicadores de página (dots) y botones "Siguiente" / "Omitir" / "Comenzar"
    - Animar transiciones entre slides con `react-native-reanimated` (interpolación de opacidad y traslación)
    - _Requirements: 4.1, 4.2, 4.4_

  - [ ] 9.2 Persistir flag de onboarding completado
    - Al completar o saltar el Onboarding, guardar flag en `AsyncStorage` y actualizar `uiStore.onboardingCompleted`
    - Verificar en `RootNavigator` que el Onboarding no se muestra en lanzamientos posteriores
    - _Requirements: 4.3_

  - [ ]* 9.3 Escribir tests de ejemplo para Onboarding
    - Test: primer lanzamiento muestra OnboardingScreen
    - Test: "Omitir" persiste flag y navega al Dashboard
    - Test: completar los 3 slides persiste flag y navega al Dashboard
    - _Archivo: `src/__tests__/components/OnboardingScreen.test.tsx`_
    - _Requirements: 4.1, 4.2, 4.3_

- [ ] 10. Phase 1 — Checkpoint
  - Verificar que la app corre en iOS y Android simulators
  - Verificar que la navegación skeleton funciona (Bottom Tabs + Stack)
  - Verificar que el Onboarding se muestra en primer lanzamiento y no en el segundo
  - Verificar que la DB se inicializa con todas las tablas y el seed de "Sin ubicación"
  - Ejecutar todos los tests: `npx jest --testPathPattern="Phase1|database|validation|onboarding" --passWithNoTests`
  - Asegurarse de que todos los tests pasan; preguntar al usuario si hay dudas.


- [ ] 11. Phase 2 — Core Features: Componentes UI primitivos
  - [ ] 11.1 Implementar `GlassCard`
    - Crear `src/components/ui/GlassCard.tsx` con props `blur`, `borderOpacity`, `backgroundOpacity`, `children`, `style`
    - Usar `expo-blur` o simulación con `rgba` + border para glassmorphism
    - _Requirements: 11.3, 16.1, 16.2_

  - [ ] 11.2 Implementar `PosterCard`
    - Crear `src/components/ui/PosterCard.tsx` con props `imageUri`, `title`, `subtitle`, `badge`, `colorTag`, `containerType`, `onPress`, `onLongPress`
    - Cuando hay `imageUri`: imagen full-bleed con gradiente overlay y título en negrita al fondo
    - Cuando no hay `imageUri`: gradiente derivado de `colorTag` con icono del tipo de Container
    - Aplicar animación de scale-down (0.97) con spring physics al presionar
    - Asignar `accessibilityLabel` y `accessibilityHint`
    - _Requirements: 5.4, 5.5, 11.6, 16.1, 16.3, 17.1_

  - [ ] 11.3 Implementar `FAB`
    - Crear `src/components/ui/FAB.tsx` con props `onPress`, `icon`, `label`, `position`
    - Usar `HapticButton` internamente para feedback háptico
    - _Requirements: 5.8, 16.1_

  - [ ] 11.4 Implementar `BottomSheet`
    - Crear `src/components/ui/BottomSheet.tsx` con props `snapPoints`, `onClose`, `children`, `initialSnapIndex`
    - Implementar drag-to-dismiss con `react-native-gesture-handler` y animación con Reanimated v3
    - _Requirements: 6.1, 7.1, 12.4, 16.1, 16.4_

  - [ ] 11.5 Implementar `SkeletonLoader`
    - Crear `src/components/ui/SkeletonLoader.tsx` con props `width`, `height`, `borderRadius`
    - Animar shimmer de izquierda a derecha con loop de 1.5s usando Reanimated v3
    - _Requirements: 11.8, 16.1, 16.5_

  - [ ] 11.6 Implementar `TagChip`, `SearchBar`, `EmptyState`, `QuantityStepper`, `HapticButton`
    - Crear `src/components/ui/TagChip.tsx` con props `tag`, `selected`, `onPress`, `onRemove`
    - Crear `src/components/ui/SearchBar.tsx` con props `value`, `onChangeText`, `onFocus`, `onBlur`, `placeholder`
    - Crear `src/components/ui/EmptyState.tsx` con props `illustration`, `title`, `subtitle`, `ctaLabel`, `onCta`
    - Crear `src/components/ui/QuantityStepper.tsx` con props `value`, `min`, `max`, `onChange`; mínimo default 1
    - Crear `src/components/ui/HapticButton.tsx` extendiendo `Pressable` con `hapticStyle` y `expo-haptics`
    - Asignar `accessibilityLabel` a todos los componentes interactivos
    - _Requirements: 11.10, 16.1, 17.1_

  - [ ]* 11.7 Escribir property test para crash-safety de componentes UI (Property 16)
    - **Property 16: UI primitive components are crash-safe with invalid props**
    - Para cualquier combinación de props inválidos/ausentes, ningún componente primitivo debe lanzar excepción en producción
    - **Validates: Requirements 16.6**
    - _Archivo: `src/__tests__/properties/container.property.test.ts`_

  - [ ]* 11.8 Escribir tests de ejemplo para componentes UI
    - `GlassCard`: renderiza children, aplica opacidad correcta
    - `PosterCard`: muestra imagen cuando hay URI, muestra gradiente cuando no hay URI, badge visible
    - `SkeletonLoader`: renderiza con dimensiones correctas
    - `QuantityStepper`: botón + incrementa, botón - decrementa, no baja del mínimo
    - `EmptyState`: muestra título, CTA visible y llamable
    - _Archivos: `src/__tests__/components/`_
    - _Requirements: 16.1, 16.2, 16.3, 16.4, 16.5_

- [ ] 12. Phase 2 — Core Features: Dashboard
  - [ ] 12.1 Implementar `DashboardScreen`
    - Crear `src/screens/dashboard/DashboardScreen.tsx`
    - Mostrar grid de 2 columnas de `PosterCard` para root Containers usando `FlatList`
    - Mostrar `EmptyState` cuando no hay Containers con CTA para crear el primero
    - Mostrar summary bar: total Containers, total Items, Locations en uso
    - Incluir `SearchBar` en la parte superior
    - Incluir barra horizontal de `TagChip` para filtro por Tags
    - Incluir fila de chips de Location para filtro por Location
    - Incluir `FAB` para crear Container
    - Implementar toggle grid/list en el header
    - Implementar pull-to-refresh que llama `fetchRootContainers`
    - Mostrar `SkeletonLoader` mientras `loading === true`
    - Implementar long-press en Card para context menu (Edit, Move, Delete, Share)
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 5.8, 5.9, 8.1, 9.3, 10.5_

  - [ ]* 12.2 Escribir tests de ejemplo para Dashboard
    - Test: empty state visible cuando no hay containers
    - Test: FAB visible y navegable
    - Test: toggle grid/list cambia layout
    - Test: pull-to-refresh llama fetchRootContainers
    - _Archivo: `src/__tests__/components/DashboardScreen.test.tsx`_
    - _Requirements: 5.1, 5.2, 5.6, 5.7, 5.8_

- [ ] 13. Phase 2 — Core Features: Gestión de Containers
  - [ ] 13.1 Implementar `CreateContainerModal`
    - Crear `src/screens/containers/CreateContainerModal.tsx` como bottom sheet modal
    - Campos: name (required, con validación inline), type (picker), location (picker), description, color tag (paleta de colores), cover image (trigger Camera_Module)
    - Al submit válido: llamar `containerStore.createContainer`, cerrar modal, animar nueva Card al grid
    - Al submit sin nombre: mostrar error inline sin cerrar el modal
    - _Requirements: 6.1, 6.2, 6.3, 6.4_

  - [ ] 13.2 Implementar `ContainerDetailScreen`
    - Crear `src/screens/containers/ContainerDetailScreen.tsx`
    - Hero header con cover image (o gradiente) del Container
    - Sección de metadata: nombre, tipo, location con icono, descripción, tags
    - Badge con total de Items (incluyendo nested)
    - Lista de sub-Containers como `PosterCard` en scroll horizontal
    - Lista de Items con soporte swipe-left para Delete y Edit
    - FAB secundario para agregar sub-Container
    - Botón "Add Item" para abrir `CreateItemModal`
    - _Requirements: 6.5, 6.6, 6.10, 7.7_

  - [ ] 13.3 Implementar `EditContainerModal`
    - Crear `src/screens/containers/EditContainerModal.tsx` pre-poblado con datos actuales del Container
    - Reutilizar el mismo formulario de `CreateContainerModal`
    - Al submit: llamar `containerStore.updateContainer`
    - _Requirements: 6.7_

  - [ ] 13.4 Implementar lógica de eliminación de Container con confirmación
    - En `ContainerDetailScreen` y context menu del Dashboard, implementar flujo de delete
    - Si el Container tiene sub-Containers o Items: mostrar `Alert` de confirmación con advertencia
    - Al confirmar: llamar `containerStore.deleteContainer` (cascading delete via FK en SQLite)
    - _Requirements: 6.8, 6.9_

  - [ ]* 13.5 Escribir tests de ejemplo para gestión de Containers
    - Test: crear Container con nombre válido persiste y aparece en Dashboard
    - Test: crear Container sin nombre muestra error inline
    - Test: editar Container pre-popula formulario
    - Test: eliminar Container con sub-containers muestra confirmación
    - _Archivo: `src/__tests__/components/ContainerScreens.test.tsx`_
    - _Requirements: 6.3, 6.4, 6.7, 6.8, 6.9_

- [ ] 14. Phase 2 — Core Features: Gestión de Items
  - [ ] 14.1 Implementar `CreateItemModal`
    - Crear `src/screens/items/CreateItemModal.tsx` como bottom sheet modal
    - Campos: name (required), description, quantity (`QuantityStepper`, min 1), cover image, tags (multi-select `TagChip`)
    - Al submit válido: llamar `itemStore.createItem`, animar nuevo Item a la lista
    - Al submit sin nombre: mostrar error inline
    - _Requirements: 7.1, 7.2, 7.3, 7.4_

  - [ ] 14.2 Implementar `ItemDetailBottomSheet`
    - Crear `src/screens/items/ItemDetailBottomSheet.tsx`
    - Mostrar: cover image, nombre, descripción, quantity, tags, breadcrumb del Container
    - _Requirements: 7.5_

  - [ ] 14.3 Implementar `EditItemModal` y lógica de quantity cero
    - Crear `src/screens/items/EditItemModal.tsx` pre-poblado con datos del Item
    - Cuando quantity se edita a 0: mostrar `Alert` preguntando si eliminar o mantener con quantity 0
    - _Requirements: 7.6_

  - [ ] 14.4 Implementar swipe-left en lista de Items y mover Item
    - En `ContainerDetailScreen`, implementar swipe-left con `react-native-gesture-handler` para revelar acciones Delete y Edit
    - Implementar `moveItem`: picker de Container destino, llamar `itemStore.moveItem`, refrescar ambas listas
    - _Requirements: 7.7, 7.8_

  - [ ]* 14.5 Escribir tests de ejemplo para gestión de Items
    - Test: crear Item con nombre válido aparece en lista del Container
    - Test: crear Item sin nombre muestra error inline
    - Test: swipe-left revela acciones Delete y Edit
    - Test: quantity a 0 muestra confirmación
    - _Archivo: `src/__tests__/components/ItemScreens.test.tsx`_
    - _Requirements: 7.3, 7.4, 7.6, 7.7_

- [ ] 15. Phase 2 — Core Features: Camera Module
  - [ ] 15.1 Implementar `useCamera` hook
    - Crear `src/hooks/useCamera.ts` con función `pickImage(): Promise<string | null>`
    - Presentar action sheet con "Tomar foto" y "Elegir de galería"
    - Solicitar permisos de cámara/galería con `expo-image-picker`; si se deniegan, mostrar `Alert` con instrucciones
    - Redimensionar imagen a máximo 1024×1024 preservando aspect ratio con `expo-image-manipulator`
    - Guardar imagen en directorio local de la app con `expo-file-system`; retornar URI local
    - Almacenar solo la URI en SQLite (no binario)
    - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.5_

  - [ ]* 15.2 Escribir property test para resize de imágenes (Property 14)
    - **Property 14: Image resize enforces max 1024×1024**
    - Para cualquier dimensión de imagen arbitraria, el resultado debe tener ancho y alto ≤ 1024 preservando aspect ratio
    - **Validates: Requirements 13.4**
    - _Archivo: `src/__tests__/properties/export.property.test.ts`_

  - [ ]* 15.3 Escribir tests de ejemplo para Camera Module
    - Test: action sheet muestra dos opciones
    - Test: permiso denegado muestra Alert con instrucciones
    - _Archivo: `src/__tests__/unit/useCamera.test.ts`_
    - _Requirements: 13.1, 13.3_

- [ ] 16. Phase 2 — Checkpoint
  - Verificar CRUD completo de Containers: crear, ver detalle, editar, eliminar con cascada
  - Verificar CRUD completo de Items: crear, ver detalle, editar, eliminar, mover
  - Verificar que la cámara funciona en simulador (galería) y que las imágenes se muestran en Cards
  - Ejecutar todos los tests: `npx jest --passWithNoTests`
  - Asegurarse de que todos los tests pasan; preguntar al usuario si hay dudas.


- [ ] 17. Phase 3 — Discovery: Search Engine
  - [ ] 17.1 Implementar `SearchScreen` y lógica de búsqueda
    - Crear `src/screens/search/SearchScreen.tsx`
    - Implementar `useSearch` hook en `src/hooks/useSearch.ts` con debounce de 300ms
    - El hook llama `ContainerRepository.search(query)` e `ItemRepository.search(query)` en paralelo
    - Mostrar resultados agrupados: Containers primero, luego Items, cada uno con thumbnail, nombre y breadcrumb de location
    - Cuando la búsqueda está vacía: mostrar los 5 Containers más recientemente accedidos como "Recientes"
    - Cuando no hay resultados: mostrar `EmptyState` con "No se encontraron resultados" y el término resaltado
    - Al tocar resultado: navegar a `ContainerDetailScreen` o abrir `ItemDetailBottomSheet`
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6_

  - [ ] 17.2 Implementar `searchStore`
    - Crear `src/store/searchStore.ts` (o integrar en `uiStore`) con estado `query`, `results`, `recentContainers`, `isSearching`
    - Acción `setQuery` con debounce, `search`, `clearSearch`
    - Persistir `recentContainers` en `AsyncStorage` (máximo 5 entradas)
    - _Requirements: 8.2, 8.5_

  - [ ]* 17.3 Escribir property test para resultados de búsqueda contienen query (Property 8)
    - **Property 8: Search results contain query string**
    - Para cualquier query no vacía, todos los resultados deben tener `name` o `description` conteniendo la query (case-insensitive)
    - **Validates: Requirements 8.2**
    - _Archivo: `src/__tests__/properties/search.property.test.ts`_

  - [ ]* 17.4 Escribir property test para orden de resultados (Property 9)
    - **Property 9: Search results grouped — containers before items**
    - Para cualquier query que retorne mezcla de Containers e Items, todos los Containers deben aparecer antes que todos los Items
    - **Validates: Requirements 8.3**
    - _Archivo: `src/__tests__/properties/search.property.test.ts`_

  - [ ]* 17.5 Escribir tests de ejemplo para búsqueda
    - Test: empty state cuando no hay resultados
    - Test: sugerencias recientes cuando query está vacía
    - Test: resultados aparecen dentro de 300ms del último keystroke (mock de debounce)
    - _Archivo: `src/__tests__/components/SearchScreen.test.tsx`_
    - _Requirements: 8.2, 8.5, 8.6_

- [ ] 18. Phase 3 — Discovery: Sistema de Tags
  - [ ] 18.1 Implementar `TagManagerModal`
    - Crear `src/screens/settings/TagManagerModal.tsx` (modal accesible desde Settings y formularios)
    - Listar todos los Tags existentes con su color
    - Crear Tag: campo de nombre + selector de color (paleta de 12 colores predefinidos)
    - Editar y eliminar Tags existentes
    - _Requirements: 9.1_

  - [ ] 18.2 Implementar filtro de Tags en Dashboard
    - En `DashboardScreen`, conectar la barra horizontal de `TagChip` al `containerStore`
    - Cuando se seleccionan uno o más Tags: llamar `ContainerRepository.findByTagIds(selectedTagIds)` (semántica AND)
    - Al tocar un Tag chip en una Card: navegar a vista filtrada con ese Tag
    - _Requirements: 9.3, 9.4, 9.5_

  - [ ]* 18.3 Escribir property test para filtro AND de Tags (Property 10)
    - **Property 10: Tag filter returns only containers with ALL selected tags**
    - Para cualquier conjunto no vacío de Tags seleccionados, todos los Containers retornados deben tener TODOS los Tags del conjunto
    - **Validates: Requirements 9.4**
    - _Archivo: `src/__tests__/properties/tags.property.test.ts`_

- [ ] 19. Phase 3 — Discovery: Gestión de Ubicaciones
  - [ ] 19.1 Implementar `LocationsScreen`
    - Crear `src/screens/settings/LocationsScreen.tsx` accesible desde el Tab de Locations
    - Listar todas las Locations con nombre e icono
    - Crear Location: campo nombre + selector de icono (set predefinido de iconos Ionicons)
    - Editar y eliminar Locations (con confirmación al eliminar)
    - Al eliminar: llamar `LocationRepository.delete` que nullifica `location_id` en Containers afectados
    - _Requirements: 10.1, 10.2, 10.3, 10.4_

  - [ ] 19.2 Implementar filtro de Locations en Dashboard
    - En `DashboardScreen`, conectar la fila de chips de Location al `containerStore`
    - Cuando se selecciona una Location: llamar `ContainerRepository.findByLocationId(locationId)`
    - _Requirements: 10.5_

  - [ ]* 19.3 Escribir tests de ejemplo para Locations
    - Test: crear Location persiste y aparece en lista
    - Test: eliminar Location muestra confirmación
    - Test: filtro por Location en Dashboard muestra solo Containers de esa Location
    - _Archivo: `src/__tests__/components/LocationsScreen.test.tsx`_
    - _Requirements: 10.1, 10.2, 10.4, 10.5_

- [ ] 20. Phase 3 — Checkpoint
  - Verificar búsqueda en tiempo real con resultados agrupados y debounce de 300ms
  - Verificar filtro de Tags (semántica AND) en Dashboard
  - Verificar filtro de Locations en Dashboard
  - Verificar gestión completa de Tags y Locations
  - Ejecutar todos los tests: `npx jest --passWithNoTests`
  - Asegurarse de que todos los tests pasan; preguntar al usuario si hay dudas.


- [ ] 21. Phase 4 — Polish & Export: Animaciones y transiciones
  - [ ] 21.1 Implementar transiciones de pantalla con shared elements
    - En `DashboardStack.tsx`, configurar transición de expansión de Card al navegar a `ContainerDetailScreen` (hero transition, 350ms)
    - Usar `react-native-reanimated` shared values para animar posición y tamaño de la Card
    - Implementar reverse animation al navegar hacia atrás
    - _Requirements: 11.5, 12.3, 12.5_

  - [ ] 21.2 Refinar animaciones de interacción
    - Verificar que todos los `PosterCard` aplican scale-down (0.97) con spring physics al presionar
    - Verificar que el `BottomSheet` tiene drag-to-dismiss fluido
    - Verificar que el `SkeletonLoader` tiene shimmer continuo de 1.5s
    - Verificar que el FAB tiene animación de entrada (scale + fade) al aparecer
    - _Requirements: 11.6, 16.4, 16.5_

  - [ ] 21.3 Implementar haptic feedback en todas las acciones primarias
    - Agregar `expo-haptics` en: crear Container/Item (medium), eliminar (heavy), confirmar (medium), presionar Card (light)
    - Verificar que `HapticButton` se usa consistentemente en todos los botones primarios
    - _Requirements: 11.10_

- [ ] 22. Phase 4 — Polish & Export: Accesibilidad
  - [ ] 22.1 Auditar y completar accessibilityLabel en todos los elementos interactivos
    - Revisar todos los componentes en `components/ui/` y pantallas en `screens/`
    - Agregar `accessibilityLabel` descriptivo en español y `accessibilityHint` donde aplique
    - Verificar que los títulos de pantalla se anuncian correctamente con VoiceOver/TalkBack
    - _Requirements: 17.1, 17.3_

  - [ ] 22.2 Implementar soporte de Dynamic Type y safe area insets
    - Verificar que todos los textos usan el sistema de tipografía del Theme Engine (no tamaños hardcoded)
    - Envolver todas las pantallas con `SafeAreaView` o usar `useSafeAreaInsets`
    - Verificar que los layouts no se rompen con tamaños de fuente grandes (iOS Dynamic Type, Android font scale)
    - _Requirements: 11.9, 17.2_

  - [ ] 22.3 Verificar que los elementos color-coded tienen acompañamiento de texto/icono
    - Tags: verificar que el nombre del Tag siempre acompaña al color
    - Container types: verificar que el icono del tipo siempre acompaña al color de gradiente
    - _Requirements: 17.5_

- [ ] 23. Phase 4 — Polish & Export: Export Module
  - [ ] 23.1 Implementar `exportUtils` y `Export Module`
    - Crear `src/utils/exportUtils.ts` con funciones:
      - `exportToJSON(db): Promise<ExportSchema>` — lee todos los datos de SQLite y construye el objeto `ExportSchema`
      - `importFromJSON(db, data: ExportSchema): Promise<void>` — valida el JSON con `validateExportJSON` e inserta todos los registros en DB vacía
      - `shareExportFile(data: ExportSchema): Promise<void>` — serializa a JSON, guarda con `expo-file-system`, abre share sheet con `expo-sharing`
    - _Requirements: 14.1, 14.2, 14.3_

  - [ ] 23.2 Implementar `SettingsScreen` con opciones de exportación e importación
    - Crear `src/screens/settings/SettingsScreen.tsx`
    - Opción "Exportar datos": llama `shareExportFile`, muestra indicador de progreso
    - Opción "Importar datos": abre file picker, llama `importFromJSON`, muestra confirmación de éxito o error descriptivo
    - Si el JSON importado es inválido: mostrar mensaje de error sin modificar la DB existente
    - _Requirements: 14.1, 14.4, 14.5_

  - [ ]* 23.3 Escribir property test para round-trip de exportación (Property 15)
    - **Property 15: Export/import round-trip**
    - Para cualquier estado válido de DB (Containers, Items, Locations, Tags y relaciones), exportar a JSON e importar en DB vacía debe producir estado equivalente al original
    - **Validates: Requirements 14.6**
    - _Archivo: `src/__tests__/properties/export.property.test.ts`_

  - [ ]* 23.4 Escribir tests de integración para Export/Import
    - Test: exportar DB con datos reales produce JSON válido con todos los registros
    - Test: importar JSON válido restaura todos los datos
    - Test: importar JSON inválido muestra error y no modifica la DB
    - _Archivo: `src/__tests__/integration/export.test.ts`_
    - _Requirements: 14.2, 14.3, 14.4, 14.5_

- [ ] 24. Phase 4 — Polish & Export: Optimización de rendimiento
  - [ ] 24.1 Optimizar render del Dashboard para 200+ Containers
    - Verificar que `FlatList` usa `getItemLayout` para evitar mediciones dinámicas
    - Implementar `windowSize` y `maxToRenderPerBatch` apropiados en `FlatList`
    - Usar `React.memo` en `PosterCard` para evitar re-renders innecesarios
    - Verificar que las queries SQLite de `findRoots` usan índices (agregar `CREATE INDEX` en `schema.ts` para `parent_container_id`, `location_id`)
    - _Requirements: 15.5_

  - [ ] 24.2 Implementar lazy loading de imágenes y sub-containers
    - Verificar que las imágenes de cover se cargan bajo demanda (no en el listado del Dashboard)
    - Verificar que los sub-containers e Items se cargan solo al navegar al `ContainerDetailScreen`
    - Mostrar `SkeletonLoader` mientras se cargan los datos del detalle
    - _Requirements: 11.8_

- [ ] 25. Phase 4 — Checkpoint final
  - Verificar que las transiciones de pantalla son fluidas (hero transition Dashboard → ContainerDetail)
  - Verificar que el haptic feedback funciona en dispositivo físico
  - Verificar que la exportación genera JSON válido y abre el share sheet
  - Verificar que la importación restaura datos correctamente
  - Verificar que el Dashboard renderiza 200 Containers sin lag perceptible
  - Ejecutar suite completa de tests: `npx jest --passWithNoTests`
  - Asegurarse de que todos los tests pasan; preguntar al usuario si hay dudas.

- [ ] 26. Phase 5 — Visual Excellence: Nuevas dependencias y tokens de tema
  - Instalar nuevas dependencias: `lottie-react-native`, `expo-blur`, `expo-linear-gradient`, `@expo-google-fonts/inter`, `expo-splash-screen`, `expo-image-manipulator`
  - Crear `src/theme/gradients.ts` con: `brand: ['#6C63FF', '#FF6584']`, `brandOverlay`, `meshStops`, `vignette`, `cardBottom`, `shimmer`, `heatMapLow`, `heatMapHigh`
  - Crear `src/theme/glassmorphism.ts` con `GlassmorphismLevel` type y `GlassmorphismLevels` record con los 4 niveles exactos (`ultra-light`, `light`, `medium`, `heavy`)
  - Actualizar `src/theme/colors.ts` con paleta expandida: `primary`, `secondary`, `tertiary`, `success`, `warning`, `error` con variantes `light`/`dark`
  - Actualizar `src/theme/typography.ts` con Inter Variable font, micro-typography settings (letter-spacing, line-height)
  - Actualizar `src/theme/index.ts` para exportar los nuevos tokens
  - Añadir campo `last_accessed_at` al schema SQLite en `src/database/schema.ts` y crear migración correspondiente
  - _Requirements: 11.11, 11.12, 11.13, 11.14, 25.1, 26.1, 26.2_

- [ ] 27. Phase 5 — Visual Excellence: Hooks de animación y utilidades visuales
  - [ ] 27.1 Implementar `useDominantColor` hook
    - Crear `src/hooks/useDominantColor.ts` que extrae el color dominante de una imagen URI mediante muestreo de píxeles
    - Retornar string `#RRGGBB` o null si no hay imagen; usar fallback al `color_tag` si falla
    - _Requirements: 24.1_

  - [ ] 27.2 Implementar `useAnimatedCounter` hook
    - Crear `src/hooks/useAnimatedCounter.ts` con Reanimated v3 SharedValue que anima de 0 a targetValue
    - Usar easing `easeOutExpo`, duración configurable (default 800ms)
    - _Requirements: 20.5, 21.6_

  - [ ] 27.3 Implementar `useParallax` hook
    - Crear `src/hooks/useParallax.ts` que retorna DerivedValue = scrollY * factor, limitado a headerHeight
    - _Requirements: 18.1, 27.10_

  - [ ]* 27.4 Escribir property tests para hooks visuales (Properties 19, 20)
    - **Property 19: `useDominantColor` produce hex válido `#RRGGBB`**
    - **Property 20: `useParallax` limita la traslación a headerHeight**
    - **Validates: Requirements 24.1, 18.1, 27.10**
    - _Archivo: `src/__tests__/properties/animations.property.test.ts`_

- [ ] 28. Phase 5 — Visual Excellence: Componentes visuales nuevos
  - [ ] 28.1 Implementar `AnimatedCounter`
    - Crear `src/components/ui/AnimatedCounter.tsx` con props `value`, `duration`, `prefix`, `suffix`, `style`
    - Usar `useAnimatedCounter` hook internamente
    - _Requirements: 16.10, 20.5, 21.6_

  - [ ] 28.2 Implementar `GradientBorder`, `GlowContainer`, `RibbonBadge`
    - `GradientBorder`: usar `expo-linear-gradient` para renderizar borde como gradiente
    - `GlowContainer`: aplicar box shadow con color y opacidad configurables
    - `RibbonBadge`: ribbon diagonal en esquina superior derecha con brand gradient
    - _Requirements: 11.17, 16.1, 19.5, 19.6_

  - [ ] 28.3 Implementar `MeshGradientBackground`
    - Crear `src/components/ui/MeshGradientBackground.tsx` con animación continua de 8s entre color stops
    - Usar Reanimated v3 para interpolar colores
    - _Requirements: 5.13, 20.8, 24.2_

  - [ ] 28.4 Implementar `BlobTabIndicator`
    - Crear `src/components/ui/BlobTabIndicator.tsx` con spring physics (damping 18, stiffness 350)
    - Integrar en `BottomTabNavigator.tsx`
    - _Requirements: 22.7, 22.8, 27.7_

  - [ ] 28.5 Implementar `FABRadialMenu`
    - Crear `src/components/ui/FABRadialMenu.tsx` con morph animation de 250ms, opciones en arco, máximo 4 opciones
    - Integrar en el `FAB` existente mediante prop `radialMenu`
    - _Requirements: 5.8, 22.1_

  - [ ]* 28.6 Escribir property test para FABRadialMenu (Property 21)
    - **Property 21: FABRadialMenu con N > 4 opciones renderiza exactamente 4**
    - **Validates: Requirements 22.1**
    - _Archivo: `src/__tests__/properties/animations.property.test.ts`_

  - [ ]* 28.7 Escribir property test para Glassmorphism levels (Property 17)
    - **Property 17: todos los niveles tienen valores dentro de rangos válidos**
    - **Validates: Requirements 25.1**
    - _Archivo: `src/__tests__/properties/theme.property.test.ts`_

- [ ] 29. Phase 5 — Visual Excellence: PosterCard expandida y efectos visuales
  - [ ] 29.1 Actualizar `PosterCard` con aspect ratio 2:3 y capas visuales
    - Actualizar `src/components/ui/PosterCard.tsx` para renderizar con aspect ratio 2:3
    - Implementar las 5 capas visuales: imagen/gradiente, gradiente inferior, glassmorphism ultra-light, vignette, texto+badge
    - Añadir props `showRibbon`, `dominantColor`, `featured`
    - _Requirements: 19.1, 19.2, 16.7_

  - [ ] 29.2 Implementar tilt effect y shimmer on press en PosterCard
    - Tilt effect: máximo 5 grados en X e Y basado en posición del toque, spring physics (damping 15, stiffness 300)
    - Shimmer on press: sweep de 600ms al presionar y mantener
    - Badge pulse animation al cambiar el conteo
    - _Requirements: 19.3, 19.4, 19.7, 22.3, 16.8_

  - [ ] 29.3 Implementar Vignette_Effect y Dominant_Color gradient en imágenes
    - Aplicar vignette como radial gradient en PosterCard y Container Detail hero
    - Usar `useDominantColor` para generar gradiente dinámico desde la imagen de cover
    - _Requirements: 5.4, 24.1, 24.6_

  - [ ]* 29.4 Escribir tests de ejemplo para PosterCard expandida
    - Test: aspect ratio 2:3 correcto
    - Test: RibbonBadge visible cuando `showRibbon=true`
    - Test: Gradient_Border visible cuando `featured=true`
    - _Archivo: `src/__tests__/components/PosterCard.test.tsx`_

- [ ] 30. Phase 5 — Visual Excellence: Dashboard expandido (Hero, Carousel, Netflix rows)
  - [ ] 30.1 Implementar `HeroCarousel`
    - Crear `src/components/ui/HeroCarousel.tsx` con auto-play de 5s, crossfade de 800ms, pagination dots
    - Integrar Parallax_Scroll en el hero usando `useParallax` (factor 0.6)
    - _Requirements: 5.10, 18.1, 18.5, 20.1, 27.1_

  - [ ] 30.2 Implementar secciones Netflix-style en Dashboard
    - Añadir sección "Continuar organizando" con containers de la sesión actual
    - Añadir sección "Recientes" con los 8 containers más recientes como fila horizontal
    - Añadir sección "Por ubicación" con una fila por Location
    - Implementar fade-out gradient en el borde derecho de cada fila (40px)
    - _Requirements: 20.2, 20.3, 20.4, 24.8, 27.3_

  - [ ] 30.3 Implementar navegación bar transparente y large title
    - Implementar transición de nav bar de transparente a glassmorphism medium al hacer scroll
    - Implementar large title que colapsa al hacer scroll (estilo iOS)
    - _Requirements: 5.14, 20.7, 27.5_

  - [ ] 30.4 Integrar MeshGradientBackground y mini activity chart en Dashboard
    - Añadir `MeshGradientBackground` como fondo del Dashboard
    - Implementar mini bar chart de actividad de los últimos 7 días con animación de entrada
    - Añadir `AnimatedCounter` en el summary bar
    - _Requirements: 5.3, 5.15, 20.5, 20.6, 20.8_

  - [ ]* 30.5 Escribir tests de ejemplo para Dashboard expandido
    - Test: HeroCarousel visible y rota automáticamente
    - Test: sección "Recientes" visible con containers
    - Test: nav bar se vuelve glassmorphism al hacer scroll
    - _Archivo: `src/__tests__/components/DashboardScreen.test.tsx`_

- [ ] 31. Phase 5 — Visual Excellence: Shared Element Transitions y animaciones de navegación
  - [ ] 31.1 Implementar Shared_Element_Transition Dashboard → Container Detail
    - Configurar transición de expansión de Card a pantalla completa (350ms, spring damping 20, stiffness 200)
    - Implementar reverse transition al navegar hacia atrás
    - _Requirements: 18.2, 18.6, 27.4_

  - [ ] 31.2 Implementar blur transition entre pantallas
    - Al navegar entre pantallas, aplicar blur de 0 a 10px en la pantalla saliente mientras la entrante hace fade-in (300ms total)
    - _Requirements: 18.3_

  - [ ] 31.3 Implementar Context_Menu_Preview (peek and pop)
    - Al hacer long press en PosterCard, mostrar preview escalado del Container Detail con blur background
    - _Requirements: 5.9, 27.6_

  - [ ] 31.4 Implementar pull-to-refresh personalizado
    - Reemplazar spinner del sistema con animación del logo de la app (rotación + pulso)
    - _Requirements: 5.7, 22.4_

  - [ ]* 31.5 Escribir tests de ejemplo para transiciones
    - Test: Shared_Element_Transition se ejecuta al navegar a Container Detail
    - Test: Context_Menu_Preview aparece al hacer long press
    - _Archivo: `src/__tests__/components/NavigationTransitions.test.tsx`_

- [ ] 32. Phase 5 — Visual Excellence: Microinteracciones y efectos de interacción
  - [ ] 32.1 Implementar ripple effect en botones primarios
    - Añadir ripple effect en `HapticButton` que se expande desde el punto de toque (400ms, 30% opacity)
    - _Requirements: 22.2_

  - [ ] 32.2 Implementar swipe actions con spring physics y color feedback
    - Actualizar swipe-left en Items con spring physics (damping 20, stiffness 400)
    - Añadir tint rojo para Delete y azul para Edit al revelar acciones
    - _Requirements: 22.5_

  - [ ] 32.3 Implementar Glow_Effect en elementos activos
    - Aplicar `GlowContainer` en: Tag chips seleccionados, tab activo, inputs enfocados, Cards destacadas
    - _Requirements: 11.15, 24.4_

  - [ ] 32.4 Implementar Noise_Texture overlay en superficies glassmorphism
    - Añadir textura de ruido al 3% de opacidad sobre todas las superficies glassmorphism
    - _Requirements: 11.14, 24.3, 25.6_

  - [ ]* 32.5 Escribir tests de ejemplo para microinteracciones
    - Test: ripple effect se activa al presionar HapticButton
    - Test: swipe-left muestra tint correcto para Delete y Edit
    - _Archivo: `src/__tests__/components/Microinteractions.test.tsx`_

- [ ] 33. Phase 5 — Visual Excellence: Glassmorphism expandido y Tab Bar
  - [ ] 33.1 Actualizar Tab Bar con glassmorphism real y BlobTabIndicator
    - Usar `expo-blur` con intensidad 80 en el Tab Bar
    - Integrar `BlobTabIndicator` para el tab activo
    - _Requirements: 12.1, 25.2, 27.7_

  - [ ] 33.2 Actualizar BottomSheet con glassmorphism heavy y blur background
    - Aplicar glassmorphism level `heavy` al BottomSheet
    - Usar `expo-blur` con intensidad 60 para el fondo detrás del modal
    - _Requirements: 25.3_

  - [ ] 33.3 Implementar glassmorphism en navigation bar y floating cards
    - Navigation bar: glassmorphism `medium` al hacer scroll
    - Floating cards (Hero overlay, summary bar): glassmorphism `light` con colored shadow
    - _Requirements: 25.4, 25.5_

  - [ ]* 33.4 Escribir tests de ejemplo para glassmorphism
    - Test: Tab Bar usa expo-blur
    - Test: BottomSheet tiene glassmorphism heavy
    - _Archivo: `src/__tests__/components/GlassmorphismComponents.test.tsx`_

- [ ] 34. Phase 5 — Visual Excellence: Splash Screen animado y Loaders
  - [ ] 34.1 Implementar Splash_Screen animado
    - Configurar `expo-splash-screen` con animación de logo: scale-in (0.5 → 1.0) + fade-in de 600ms
    - Transición a Dashboard/Onboarding con fade-out
    - Completar en ≤ 1.5 segundos
    - _Requirements: 23.3, 23.7_

  - [ ] 34.2 Implementar custom spinner y progress indicator
    - Reemplazar todos los spinners del sistema con spinner circular personalizado (accent color, trailing gradient, 800ms/rotación)
    - Implementar progress bar con brand gradient para Export/Import
    - _Requirements: 23.4, 23.5_

  - [ ] 34.3 Actualizar EmptyState con Lottie animations
    - Actualizar `EmptyState` para usar `lottie-react-native` con prop `lottieSource`
    - Añadir assets Lottie para: empty containers, empty search results, empty items
    - _Requirements: 5.2, 23.6, 16.11_

  - [ ]* 34.4 Escribir tests de ejemplo para loaders
    - Test: Splash_Screen se muestra al lanzar la app
    - Test: EmptyState muestra Lottie animation
    - Test: progress indicator visible durante export
    - _Archivo: `src/__tests__/components/LoadersAndSplash.test.tsx`_

- [ ] 35. Phase 5 — Visual Excellence: Statistics Screen
  - [ ] 35.1 Implementar `StatisticsScreen` con donut chart y bar chart
    - Crear `src/screens/statistics/StatisticsScreen.tsx`
    - Implementar `DonutChart` con Items por tipo de Container, animación de entrada staggered
    - Implementar `ActivityBarChart` con Items por día (últimos 7 días), barras animadas con stagger de 80ms
    - Añadir `AnimatedCounter` para el total de Items en Display typography
    - _Requirements: 21.1, 21.2, 21.3, 21.6, 21.7_

  - [ ] 35.2 Implementar progress bars y location heat map
    - Implementar `ProgressBarList` con los 5 Containers más llenos, animación de 800ms
    - Implementar `LocationHeatMap` con grid de tiles coloreados por intensidad
    - _Requirements: 21.4, 21.5_

  - [ ] 35.3 Añadir Statistics tab y actualización en tiempo real
    - Añadir Statistics como Tab 4 en `BottomTabNavigator.tsx` (ajustar tabs existentes)
    - Implementar actualización de valores al navegar de vuelta a la pantalla
    - _Requirements: 21.1, 21.8_

  - [ ]* 35.4 Escribir property test para Statistics (Property 22)
    - **Property 22: valores de StatisticsScreen coinciden con agregados de SQLite**
    - **Validates: Requirements 21.6, 21.2, 21.4**
    - _Archivo: `src/__tests__/properties/statistics.property.test.ts`_

  - [ ]* 35.5 Escribir property test para AnimatedCounter (Property 18)
    - **Property 18: AnimatedCounter alcanza exactamente el valor objetivo**
    - **Validates: Requirements 21.6, 20.5**
    - _Archivo: `src/__tests__/properties/statistics.property.test.ts`_

  - [ ]* 35.6 Escribir tests de ejemplo para StatisticsScreen
    - Test: donut chart visible con segmentos por tipo
    - Test: bar chart visible con 7 barras
    - Test: AnimatedCounter muestra valor correcto
    - _Archivo: `src/__tests__/components/StatisticsScreen.test.tsx`_

- [ ] 36. Phase 5 — Checkpoint final visual
  - Verificar que el Splash_Screen animado se muestra y completa en ≤ 1.5s
  - Verificar que el Dashboard muestra Hero_Section, secciones Netflix y MeshGradient
  - Verificar que las Shared_Element_Transitions funcionan en iOS y Android
  - Verificar que el Tab Bar tiene glassmorphism real con BlobTabIndicator
  - Verificar que la StatisticsScreen muestra todos los charts con animaciones
  - Verificar que los Lottie empty states se muestran correctamente
  - Ejecutar suite completa de tests: `npx jest --passWithNoTests`
  - Asegurarse de que todos los tests pasan; preguntar al usuario si hay dudas.

---

## Task Dependency Graph

```json
{
  "waves": [
    {
      "wave": 1,
      "tasks": ["1"],
      "description": "Configuración del proyecto y estructura base"
    },
    {
      "wave": 2,
      "tasks": ["2"],
      "description": "Sistema de tipos TypeScript",
      "dependsOn": ["1"]
    },
    {
      "wave": 3,
      "tasks": ["3"],
      "description": "Theme Engine",
      "dependsOn": ["2"]
    },
    {
      "wave": 4,
      "tasks": ["4"],
      "description": "Base de datos SQLite — schema y migraciones",
      "dependsOn": ["3"]
    },
    {
      "wave": 5,
      "tasks": ["5"],
      "description": "Repositorios de datos",
      "dependsOn": ["4"]
    },
    {
      "wave": 6,
      "tasks": ["6"],
      "description": "Validaciones y utilidades",
      "dependsOn": ["5"]
    },
    {
      "wave": 7,
      "tasks": ["7"],
      "description": "Stores Zustand",
      "dependsOn": ["6"]
    },
    {
      "wave": 8,
      "tasks": ["8"],
      "description": "Navegación skeleton",
      "dependsOn": ["7"]
    },
    {
      "wave": 9,
      "tasks": ["9"],
      "description": "Onboarding",
      "dependsOn": ["8"]
    },
    {
      "wave": 10,
      "tasks": ["10"],
      "description": "Phase 1 — Checkpoint",
      "dependsOn": ["9"]
    },
    {
      "wave": 11,
      "tasks": ["11"],
      "description": "Componentes UI primitivos",
      "dependsOn": ["10"]
    },
    {
      "wave": 12,
      "tasks": ["12"],
      "description": "Dashboard",
      "dependsOn": ["11"]
    },
    {
      "wave": 13,
      "tasks": ["13"],
      "description": "Gestión de Containers",
      "dependsOn": ["12"]
    },
    {
      "wave": 14,
      "tasks": ["14"],
      "description": "Gestión de Items",
      "dependsOn": ["13"]
    },
    {
      "wave": 15,
      "tasks": ["15"],
      "description": "Camera Module",
      "dependsOn": ["14"]
    },
    {
      "wave": 16,
      "tasks": ["16"],
      "description": "Phase 2 — Checkpoint",
      "dependsOn": ["15"]
    },
    {
      "wave": 17,
      "tasks": ["17", "18", "19"],
      "description": "Search Engine, Sistema de Tags, Gestión de Ubicaciones (paralelo)",
      "dependsOn": ["16"]
    },
    {
      "wave": 18,
      "tasks": ["20"],
      "description": "Phase 3 — Checkpoint",
      "dependsOn": ["17", "18", "19"]
    },
    {
      "wave": 19,
      "tasks": ["21", "22", "23", "24"],
      "description": "Animaciones, Accesibilidad, Export Module, Rendimiento (paralelo)",
      "dependsOn": ["20"]
    },
    {
      "wave": 20,
      "tasks": ["25"],
      "description": "Phase 4 — Checkpoint final",
      "dependsOn": ["21", "22", "23", "24"]
    },
    {
      "wave": 21,
      "tasks": ["26", "27", "28"],
      "description": "Phase 5 — Tokens de tema, hooks de animación y componentes visuales nuevos (paralelo)",
      "dependsOn": ["25"]
    },
    {
      "wave": 22,
      "tasks": ["29", "30", "31", "32", "33", "34", "35"],
      "description": "Phase 5 — PosterCard expandida, Dashboard, Transiciones, Microinteracciones, Glassmorphism, Loaders, Statistics (paralelo)",
      "dependsOn": ["26", "27", "28"]
    },
    {
      "wave": 23,
      "tasks": ["36"],
      "description": "Phase 5 — Checkpoint final visual",
      "dependsOn": ["29", "30", "31", "32", "33", "34", "35"]
    }
  ]
}
```

---

## Notes

- Las tareas marcadas con `*` son opcionales y pueden omitirse para un MVP más rápido
- Cada tarea referencia los requisitos específicos para trazabilidad completa
- Los checkpoints al final de cada fase garantizan validación incremental
- Los property tests usan `fast-check` con mínimo 100 iteraciones (`numRuns: 100`)
- Los tests de ejemplo usan Jest + React Native Testing Library
- La DB de test usa `expo-sqlite` in-memory para aislamiento
- El lenguaje de implementación es **TypeScript** (strict mode) en todo el proyecto
- Las Properties 1–16 del documento de diseño están cubiertas por las tareas de property testing (fases 1–4)
- Las Properties 17–22 del documento de diseño están cubiertas por las tareas de la Phase 5 (tareas 27.4, 28.6, 28.7, 35.4, 35.5)
- Las tareas 26–36 implementan los requisitos visuales 18–27 añadidos en la expansión del documento de requisitos
- La Phase 5 (tareas 26–36) depende de la Phase 4 completada (tarea 25) y puede comenzarse una vez superado ese checkpoint
