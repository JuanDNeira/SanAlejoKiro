# Requirements Document

## Introduction

El módulo **Reciclador Inteligente** es una nueva funcionalidad de la app San Alejo (React Native + Expo) que permite al usuario identificar objetos en desuso y asignarles una acción ecológica: reciclar, donar, vender, reutilizar, reparar o desechar. El módulo se integra con la arquitectura existente (SQLite vía expo-sqlite, Zustand, TabNavigator personalizado) y añade un tercer tab "Eco" al navegador principal. Incluye un panel de resumen, un flujo de clasificación de ítems, detalle de acciones ecológicas, gamificación mediante logros y puntos, y filtros ecológicos en la búsqueda global.

---

## Glossary

- **EcoAction**: Acción ecológica asignada a un ítem. Valores posibles: `recycle` (Reciclar), `donate` (Donar), `sell` (Vender), `reuse` (Reutilizar), `repair` (Reparar), `discard` (Desechar).
- **EcoStatus**: Estado de la acción ecológica de un ítem. Valores posibles: `pending` (pendiente), `completed` (completado), `skipped` (omitido).
- **Ítem sin clasificar**: Ítem cuyo campo `eco_action` es NULL en la base de datos.
- **Ítem candidato a desuso**: Ítem sin clasificar cuyo `updated_at` tiene más de 90 días de antigüedad respecto a la fecha actual.
- **EcoHub**: Pantalla principal del módulo Reciclador Inteligente, accesible desde el tab "Eco".
- **EcoClassify**: Pantalla de flujo de clasificación de ítems candidatos a desuso.
- **EcoItemDetail**: Pantalla de detalle y edición de la acción ecológica de un ítem específico.
- **EcoHistory**: Pantalla de historial de acciones ecológicas completadas.
- **EcoAchievement**: Logro desbloqueado por el usuario al completar acciones ecológicas.
- **PuntosEco**: Puntuación acumulada del usuario basada en acciones ecológicas completadas. Valor entero no negativo.
- **EcoRepository**: Repositorio de acceso a datos para operaciones ecológicas sobre la tabla `items` y la tabla `eco_achievements`.
- **useEcoStore**: Store de Zustand que gestiona el estado del módulo Reciclador Inteligente.
- **EcoStore**: Sinónimo de `useEcoStore`.
- **Migration_v4**: Migración de base de datos versión 4 que añade los campos ecológicos a `items` y crea la tabla `eco_achievements`.
- **Sistema**: La aplicación San Alejo en su conjunto.
- **Reciclador_Inteligente**: El módulo descrito en este documento.
- **TabNavigator**: Componente de navegación por pestañas personalizado de San Alejo (`src/navigation/TabNavigator.tsx`).
- **Dashboard**: Pantalla existente de estadísticas generales de San Alejo (tab "Panel").
- **Colors.accent**: Color teal `#00D4AA` definido en `src/theme/colors.ts`, color primario del módulo ecológico.
- **Colors.accentGlow**: Variante oscura de `Colors.accent` usada como fondo del ícono activo del tab "Eco" (a definir en `colors.ts` como `#003D30` o equivalente).

---

## Requirements

---

### Requirement 1: Migración de base de datos v4

**User Story:** Como desarrollador, quiero extender el esquema de la base de datos con campos ecológicos, para que los ítems puedan almacenar su acción ecológica, notas y estado de completado.

#### Acceptance Criteria

1. THE Migration_v4 SHALL añadir la columna `eco_action` de tipo TEXT con valor por defecto NULL a la tabla `items`, aceptando únicamente los valores `'recycle'`, `'donate'`, `'sell'`, `'reuse'`, `'repair'`, `'discard'` o NULL.
2. THE Migration_v4 SHALL añadir la columna `eco_notes` de tipo TEXT con valor por defecto NULL a la tabla `items`.
3. THE Migration_v4 SHALL añadir la columna `eco_completed_at` de tipo INTEGER con valor por defecto NULL a la tabla `items`.
4. THE Migration_v4 SHALL añadir la columna `eco_status` de tipo TEXT con valor por defecto NULL a la tabla `items`, aceptando únicamente los valores `'pending'`, `'completed'`, `'skipped'` o NULL.
5. THE Migration_v4 SHALL crear la tabla `eco_achievements` con las columnas: `id` TEXT PRIMARY KEY, `type` TEXT NOT NULL CHECK(type IN ('first_rescue','guardian_verde','eco_heroe','maestro_reciclaje','leyenda_sostenible','campeon_planeta')), `unlocked_at` INTEGER NOT NULL, `metadata` TEXT.
6. THE Migration_v4 SHALL crear el índice `idx_items_eco_action` sobre la columna `eco_action` de la tabla `items`.
7. THE Migration_v4 SHALL crear el índice `idx_items_eco_status` sobre la columna `eco_status` de la tabla `items`.
8. WHEN Migration_v4 se ejecuta sobre una base de datos que ya contiene ítems, THE Migration_v4 SHALL preservar todos los valores existentes en todas las columnas de la tabla `items` (incluyendo columnas añadidas por migraciones v2 y v3) sin modificarlos; los nuevos campos `eco_action`, `eco_notes`, `eco_completed_at` y `eco_status` quedarán como NULL en los registros preexistentes.
9. IF Migration_v4 ya fue aplicada previamente (versión 4 presente en la tabla `_migrations`), THEN THE Sistema SHALL omitir su ejecución sin producir errores, siguiendo el patrón del sistema de migraciones versionado existente.
10. WHEN Migration_v4 falla en cualquiera de sus sentencias DDL, THE Sistema SHALL revertir todos los cambios de esa migración de forma que la base de datos quede en el estado previo a su ejecución, sin registrar la versión 4 en `_migrations`.

---

### Requirement 2: Tipos TypeScript para el módulo ecológico

**User Story:** Como desarrollador, quiero tipos TypeScript que representen las entidades ecológicas, para que el código del módulo sea seguro en tipos y consistente con la arquitectura existente.

#### Acceptance Criteria

1. THE Sistema SHALL definir el tipo `EcoAction` como unión de literales: `'recycle' | 'donate' | 'sell' | 'reuse' | 'repair' | 'discard'`.
2. THE Sistema SHALL definir el tipo `EcoStatus` como unión de literales: `'pending' | 'completed' | 'skipped'`.
3. THE Sistema SHALL extender la interfaz `Item` (en `src/types/Item.ts`) con los campos opcionales: `eco_action?: EcoAction`, `eco_notes?: string`, `eco_completed_at?: UnixTimestamp`, `eco_status?: EcoStatus`.
4. THE Sistema SHALL definir la interfaz `EcoAchievement` con los campos: `id: string`, `type: string`, `unlocked_at: UnixTimestamp`, `metadata?: string`.
5. THE Sistema SHALL definir la interfaz `EcoStats` con los campos: `totalPending: number`, `totalCompleted: number`, `totalSkipped: number`, `totalDiscarded: number`, `totalRescued: number`, `ecoPoints: number`; donde `totalRescued` cuenta ítems con `eco_action !== 'discard'` y `eco_status === 'completed'`; `totalDiscarded` cuenta ítems con `eco_action === 'discard'` y `eco_status === 'completed'`; y `ecoPoints` es un entero no negativo calculado como la suma ponderada de puntos de todas las acciones completadas según la tabla del Requirement 14.

---

### Requirement 3: EcoRepository — acceso a datos ecológicos

**User Story:** Como desarrollador, quiero un repositorio dedicado para las operaciones ecológicas sobre la base de datos, para que la lógica de acceso a datos esté encapsulada y sea reutilizable.

#### Acceptance Criteria

1. THE EcoRepository SHALL exponer un método `updateEcoAction(itemId: string, action: EcoAction, notes?: string): Promise<void>` que actualice `eco_action` con `action`, `eco_notes` con `notes` (o NULL si `notes` es undefined), establezca `eco_status` en `'pending'` y actualice `updated_at` con el timestamp Unix actual para el ítem indicado.
2. THE EcoRepository SHALL exponer un método `completeEcoAction(itemId: string): Promise<void>` que establezca `eco_status` en `'completed'`, `eco_completed_at` con el timestamp Unix actual y actualice `updated_at` con el timestamp Unix actual para el ítem indicado.
3. THE EcoRepository SHALL exponer un método `skipEcoAction(itemId: string): Promise<void>` que establezca `eco_status` en `'skipped'` y actualice `updated_at` con el timestamp Unix actual para el ítem indicado.
4. THE EcoRepository SHALL exponer un método `findByEcoAction(action: EcoAction): Promise<Item[]>` que retorne todos los ítems con `eco_action` igual a `action`, ordenados por `updated_at` descendente.
5. THE EcoRepository SHALL exponer un método `findUnclassified(limitDays?: number): Promise<Item[]>` que retorne ítems cuyo `eco_action` sea NULL, ordenados por `updated_at` ascendente.
6. WHEN `limitDays` se proporciona en `findUnclassified`, THE EcoRepository SHALL filtrar únicamente ítems cuyo `updated_at` sea estrictamente menor que `(Date.now() - limitDays * 86_400_000)`.
7. THE EcoRepository SHALL exponer un método `getEcoStats(): Promise<EcoStats>` que calcule y retorne las estadísticas ecológicas agregadas del usuario mediante una única consulta SQL con agregaciones por `eco_action` y `eco_status`.
8. THE EcoRepository SHALL exponer un método `findCompleted(): Promise<Item[]>` que retorne todos los ítems con `eco_status` igual a `'completed'`, ordenados por `eco_completed_at` descendente.
9. THE EcoRepository SHALL exponer un método `saveAchievement(achievement: Omit<EcoAchievement, 'id'>): Promise<EcoAchievement>` que genere un UUID, persista el logro en `eco_achievements` y retorne el registro completo.
10. THE EcoRepository SHALL exponer un método `findAllAchievements(): Promise<EcoAchievement[]>` que retorne todos los logros desbloqueados, ordenados por `unlocked_at` descendente.
11. IF el `itemId` proporcionado a `updateEcoAction`, `completeEcoAction` o `skipEcoAction` no existe en la tabla `items`, THEN THE EcoRepository SHALL completar la operación sin error (no-op), siguiendo el patrón de `ItemRepository.update`.
12. IF una operación de lectura o escritura del EcoRepository falla por error de base de datos (distinto del caso del criterio 11), THEN THE EcoRepository SHALL propagar el error como excepción, siguiendo el patrón async/await de los repositorios existentes.

---

### Requirement 4: useEcoStore — estado global del módulo ecológico

**User Story:** Como desarrollador, quiero un store de Zustand dedicado al módulo ecológico, para que el estado de las acciones ecológicas sea accesible desde cualquier pantalla de la app.

#### Acceptance Criteria

1. THE useEcoStore SHALL exponer el estado: `ecoStats: EcoStats | null`, `pendingItems: Item[]`, `completedItems: Item[]`, `unclassifiedItems: Item[]`, `achievements: EcoAchievement[]`, `isLoading: boolean`, `error: string | null`, y la acción `clearError: () => void`.
2. THE useEcoStore SHALL exponer la acción `loadEcoStats(): Promise<void>` que establezca `isLoading: true`, cargue las estadísticas desde `EcoRepository.getEcoStats()`, las almacene en `ecoStats` y establezca `isLoading: false` al finalizar.
3. THE useEcoStore SHALL exponer la acción `loadPendingItems(): Promise<void>` que cargue los ítems con `eco_status === 'pending'` desde `EcoRepository.findByEcoAction` para cada acción y los almacene en `pendingItems`.
4. THE useEcoStore SHALL exponer la acción `loadUnclassifiedItems(limitDays?: number): Promise<void>` que llame a `EcoRepository.findUnclassified(limitDays)` y almacene el resultado en `unclassifiedItems`; WHEN `limitDays` es undefined, THE useEcoStore SHALL llamar a `EcoRepository.findUnclassified()` sin argumentos.
5. THE useEcoStore SHALL exponer la acción `loadCompletedItems(): Promise<void>` que cargue los ítems con `eco_status === 'completed'` desde `EcoRepository.findCompleted()` y los almacene en `completedItems`.
6. THE useEcoStore SHALL exponer la acción `assignEcoAction(itemId: string, action: EcoAction, notes?: string): Promise<void>` que llame a `EcoRepository.updateEcoAction`, mueva el ítem de `unclassifiedItems` a `pendingItems` en el estado local (si estaba en `unclassifiedItems`), y recargue `ecoStats` llamando a `EcoRepository.getEcoStats()`.
7. THE useEcoStore SHALL exponer la acción `completeEcoAction(itemId: string): Promise<void>` que llame a `EcoRepository.completeEcoAction`, mueva el ítem de `pendingItems` a `completedItems` en el estado local, recargue `ecoStats` llamando a `EcoRepository.getEcoStats()`, y evalúe logros usando el `totalRescued` actualizado.
8. THE useEcoStore SHALL exponer la acción `skipItem(itemId: string): Promise<void>` que llame a `EcoRepository.skipEcoAction` y elimine el ítem de `unclassifiedItems` en el estado local.
9. THE useEcoStore SHALL exponer la acción `loadAchievements(): Promise<void>` que cargue los logros desde `EcoRepository.findAllAchievements()` y los almacene en `achievements`.
10. WHEN `completeEcoAction` se ejecuta y el `totalRescued` actualizado en `ecoStats` alcanza exactamente un umbral de logro (1, 5, 10, 25, 50, 100), AND el `type` del logro correspondiente no está ya presente en `achievements`, THEN THE useEcoStore SHALL llamar a `EcoRepository.saveAchievement` y añadir el nuevo logro al inicio de `achievements`.
11. Las acciones de carga (`loadEcoStats`, `loadPendingItems`, `loadUnclassifiedItems`, `loadCompletedItems`, `loadAchievements`) SHALL establecer `isLoading: true` al inicio y `isLoading: false` al finalizar (con éxito o error).
12. Las acciones de mutación (`assignEcoAction`, `completeEcoAction`, `skipItem`) SHALL NO modificar `isLoading`; los errores de estas acciones se almacenan en `error` sin afectar `isLoading`.
13. IF cualquier acción del useEcoStore falla, THEN THE useEcoStore SHALL almacenar el mensaje de error en `error` y establecer `isLoading: false` si la acción fallida era una acción de carga, sin propagar la excepción al componente llamante.

---

### Requirement 5: Tab "Eco" en el TabNavigator

**User Story:** Como usuario, quiero acceder al módulo Reciclador Inteligente desde la barra de navegación principal, para que sea tan accesible como las secciones "Inicio" y "Panel".

#### Acceptance Criteria

1. THE TabNavigator SHALL mostrar un tercer tab con la etiqueta "Eco" y el ícono `leaf-outline` (inactivo) / `leaf` (activo) de la librería `@expo/vector-icons` (Ionicons).
2. WHEN el usuario selecciona el tab "Eco", THE TabNavigator SHALL renderizar la pantalla EcoHub en el área de contenido.
3. THE TabNavigator SHALL aplicar `Colors.accent` como color del ícono y etiqueta del tab "Eco" cuando está activo.
4. THE TabNavigator SHALL aplicar `Colors.textTertiary` como color del ícono y etiqueta del tab "Eco" cuando está inactivo, igual que los demás tabs.
5. THE TabNavigator SHALL aplicar `Colors.accentGlow` como color de fondo del contenedor del ícono activo del tab "Eco" (equivalente al rol que `Colors.primaryGlow` cumple para los tabs "Home" y "Dashboard").
6. THE TabNavigator SHALL mantener el comportamiento de animación de escala (`Animated.spring`) y feedback háptico (`Haptics.selectionAsync`) existente para el nuevo tab "Eco".
7. THE TabNavigator SHALL actualizar el tipo `TabName` (definido localmente en `TabNavigator.tsx`) para incluir `'Eco'` como valor válido junto a `'Home'` y `'Dashboard'`.

---

### Requirement 6: Rutas de navegación del módulo ecológico

**User Story:** Como desarrollador, quiero que las pantallas del módulo ecológico estén registradas en el stack de navegación principal, para que puedan ser accedidas desde cualquier parte de la app.

#### Acceptance Criteria

1. THE Sistema SHALL añadir las siguientes rutas a `RootStackParamList` en `src/navigation/types.ts`: `EcoHub: undefined`, `EcoItemDetail: { itemId: string }`, `EcoClassify: undefined`, `EcoHistory: undefined`.
2. THE Sistema SHALL registrar las pantallas `EcoHub`, `EcoItemDetail`, `EcoClassify` y `EcoHistory` como `<Stack.Screen>` en `RootNavigator.tsx`.
3. WHEN se navega a `EcoItemDetail`, THE Sistema SHALL pasar el parámetro `itemId` de tipo `string` como parte de los route params; IF `itemId` está ausente o es una cadena vacía al montar `EcoItemDetail`, THEN THE EcoItemDetail SHALL mostrar el estado de error definido en el Requirement 9, criterio 8.

---

### Requirement 7: Pantalla EcoHub — panel principal del Reciclador Inteligente

**User Story:** Como usuario, quiero ver un resumen de mis acciones ecológicas en una pantalla dedicada, para que pueda entender de un vistazo mi impacto ambiental y las tareas pendientes.

#### Acceptance Criteria

1. THE EcoHub SHALL mostrar una sección "Impacto Ecológico" con cuatro `StatCard`: ítems rescatados (`totalRescued`), total completados (`totalCompleted`), total pendientes (`totalPending`) y PuntosEco (`ecoPoints`), usando `Colors.accent` como color de acento.
2. THE EcoHub SHALL mostrar una sección "Pendientes" con la lista de `pendingItems` agrupados por `eco_action`, mostrando el ícono Ionicons correspondiente a cada acción y la etiqueta en español.
3. THE EcoHub SHALL mostrar un indicador visual de progreso ecológico (barra de progreso) cuyo valor sea `totalRescued / (totalRescued + totalDiscarded)` cuando `(totalRescued + totalDiscarded) > 0`; WHEN `(totalRescued + totalDiscarded) === 0`, THE EcoHub SHALL mostrar la barra en 0%.
4. WHEN `pendingItems` está vacío Y `unclassifiedItems` está vacío, THE EcoHub SHALL mostrar el componente `EmptyState` con el mensaje "Todo en orden. No tienes ítems pendientes de clasificar." en lugar de las secciones "Pendientes" y de progreso.
5. THE EcoHub SHALL mostrar un `FloatingActionButton` con el ícono `leaf-outline` que navegue a la pantalla `EcoClassify` al ser pulsado.
6. WHEN el usuario pulsa sobre un ítem de la lista de pendientes, THE EcoHub SHALL navegar a `EcoItemDetail` pasando el `itemId` del ítem seleccionado.
7. THE EcoHub SHALL mostrar una sección "Logros Recientes" con los últimos 3 elementos de `achievements` ordenados por `unlocked_at` descendente; WHEN `achievements` tiene menos de 3 elementos, THE EcoHub SHALL mostrar únicamente los disponibles sin espacios vacíos.
8. THE EcoHub SHALL aplicar el gradiente `Colors.gradients.accent` en el encabezado de la pantalla y `Colors.accent` como color de acento en íconos y elementos interactivos.
9. THE EcoHub SHALL usar `ScreenContainer` como contenedor raíz y `SectionHeader` para los títulos de cada sección ("Impacto Ecológico", "Pendientes", "Logros Recientes").
10. WHEN EcoHub se monta, THE EcoHub SHALL llamar a `Promise.all([loadEcoStats(), loadPendingItems(), loadAchievements()])` para cargar los datos en paralelo.
11. WHILE `isLoading` es `true` en el EcoStore, THE EcoHub SHALL mostrar el componente `SkeletonLoader` ocupando el área de contenido en lugar de las secciones de datos.
12. IF cualquiera de las cargas paralelas del criterio 10 falla, THE EcoHub SHALL mostrar el mensaje de `error` del EcoStore en un banner de error no bloqueante, manteniendo visible el contenido que sí se cargó correctamente.

---

### Requirement 8: Pantalla EcoClassify — flujo de clasificación de ítems

**User Story:** Como usuario, quiero revisar mis ítems en desuso y asignarles una acción ecológica de forma rápida, para que pueda gestionar mis objetos almacenados sin tener que buscarlos manualmente.

#### Acceptance Criteria

1. WHEN EcoClassify se monta, THE EcoClassify SHALL llamar a `useEcoStore.loadUnclassifiedItems(90)` para cargar los ítems candidatos a desuso (sin `eco_action` y con `updated_at` anterior a 90 días).
2. THE EcoClassify SHALL mostrar los ítems candidatos de uno en uno en formato de tarjeta con glassmorphism (`Colors.glass`, `Colors.glassBorder`), mostrando: nombre, descripción (si existe), imagen de portada (si existe, usando la `cover_image_uri`) y el nombre del contenedor al que pertenece (obtenido del `ContainerRepository` o del `containerStore`).
3. THE EcoClassify SHALL mostrar un selector de acción ecológica con las seis opciones en español: "Reciclar" (`recycle`, ícono `leaf-outline`), "Donar" (`donate`, ícono `heart-outline`), "Vender" (`sell`, ícono `pricetag-outline`), "Reutilizar" (`reuse`, ícono `refresh-outline`), "Reparar" (`repair`, ícono `construct-outline`), "Desechar" (`discard`, ícono `trash-outline`).
4. WHEN el usuario selecciona una acción ecológica para el ítem actual, THE EcoClassify SHALL llamar a `useEcoStore.assignEcoAction(itemId, action)` y avanzar automáticamente al siguiente ítem candidato.
5. WHEN el usuario pulsa "Omitir", THE EcoClassify SHALL llamar a `useEcoStore.skipItem(itemId)` y avanzar al siguiente ítem candidato sin asignar acción.
6. WHEN el índice del ítem actual supera el total de candidatos (todos clasificados u omitidos), THE EcoClassify SHALL mostrar una pantalla de resumen con el recuento de ítems clasificados en esa sesión (excluyendo omitidos) y un botón "Finalizar" que navegue de vuelta a EcoHub.
7. THE EcoClassify SHALL mostrar en el encabezado el texto "Ítem X de N" donde X es el índice 1-based del ítem actual y N es el total de candidatos cargados al montar la pantalla.
8. WHEN `unclassifiedItems` está vacío al montar EcoClassify, THE EcoClassify SHALL mostrar el componente `EmptyState` con el mensaje "No hay ítems candidatos a clasificar en este momento." y un botón "Volver" que navegue a EcoHub.
9. THE EcoClassify SHALL aplicar el tema oscuro premium con glassmorphism en las tarjetas de ítem, usando `Colors.glass` como fondo y `Colors.glassBorder` como borde.

---

### Requirement 9: Pantalla EcoItemDetail — detalle y edición de acción ecológica

**User Story:** Como usuario, quiero ver y editar la acción ecológica de un ítem específico, para que pueda añadir notas relevantes y marcar la acción como completada cuando la haya realizado.

#### Acceptance Criteria

1. WHEN EcoItemDetail se monta con un `itemId` válido, THE EcoItemDetail SHALL cargar el ítem llamando a `ItemRepository.findById(itemId)` y mostrar: nombre, descripción (si existe), imagen de portada (si existe), `eco_action` actual y `eco_notes` actuales.
2. THE EcoItemDetail SHALL mostrar el selector de acción ecológica (mismas seis opciones que EcoClassify) que permita cambiar la `eco_action` asignada al ítem.
3. THE EcoItemDetail SHALL mostrar un campo `PremiumInput` para editar `eco_notes` con placeholder contextual: "Precio estimado (EUR)" para `sell`, "Punto de donación" para `donate`, "Descripción de la reparación" para `repair`, "Notas adicionales" para `recycle`, `reuse` y `discard`.
4. WHEN el usuario pulsa "Guardar cambios", THE EcoItemDetail SHALL llamar a `useEcoStore.assignEcoAction(itemId, selectedAction, notes)` y mostrar una confirmación visual (cambio de color del botón a `Colors.success` durante 1,5 segundos).
5. WHEN el usuario pulsa "Marcar como completado", THE EcoItemDetail SHALL llamar a `useEcoStore.completeEcoAction(itemId)`, actualizar la UI local para reflejar `eco_status: 'completed'` y mostrar una animación de celebración (escala pulsante del ícono de acción durante 600ms).
6. WHILE `eco_status` es `'completed'`, THE EcoItemDetail SHALL mostrar el ítem en modo solo lectura: los controles de selección de acción y el campo de notas estarán deshabilitados, y se mostrará la fecha de completado formateada como `DD/MM/YYYY` junto al ícono de acción.
7. THE EcoItemDetail SHALL mostrar el ícono Ionicons correspondiente a la `eco_action` actual con color `Colors.accent`; WHEN `eco_action` es NULL, THE EcoItemDetail SHALL mostrar el ícono `help-circle-outline` con color `Colors.textTertiary`.
8. IF `itemId` no corresponde a ningún ítem en la base de datos (o es inválido), THEN THE EcoItemDetail SHALL mostrar el mensaje "Ítem no encontrado" y un botón "Volver a Eco" que navegue a EcoHub.

---

### Requirement 10: Pantalla EcoHistory — historial de acciones completadas

**User Story:** Como usuario, quiero ver el historial de todos los ítems cuyas acciones ecológicas he completado, para que pueda hacer seguimiento de mi contribución ambiental a lo largo del tiempo.

#### Acceptance Criteria

1. WHEN EcoHistory se monta, THE EcoHistory SHALL llamar a `useEcoStore.loadCompletedItems()` para cargar los ítems con `eco_status === 'completed'`, ordenados por `eco_completed_at` descendente.
2. THE EcoHistory SHALL mostrar cada ítem completado en una tarjeta con: nombre del ítem, ícono Ionicons de la `eco_action` con su etiqueta en español, `eco_notes` (si existe), y la fecha de completado formateada como `DD/MM/YYYY` derivada de `eco_completed_at`.
3. THE EcoHistory SHALL mostrar en el encabezado dos contadores: "Rescatados: N" (ítems con `eco_action !== 'discard'` y `eco_status === 'completed'`) y "Desechados: N" (ítems con `eco_action === 'discard'` y `eco_status === 'completed'`).
4. WHEN `completedItems` está vacío, THE EcoHistory SHALL mostrar el componente `EmptyState` con el mensaje "Aún no has completado ninguna acción ecológica." en lugar de la lista.
5. THE EcoHistory SHALL aplicar `Colors.accent` como color del ícono de acción para ítems con `eco_action !== 'discard'` y `Colors.textTertiary` para ítems con `eco_action === 'discard'`.

---

### Requirement 11: Integración ecológica en ContainerDetailScreen

**User Story:** Como usuario, quiero poder asignar una acción ecológica a un ítem directamente desde la pantalla de detalle de contenedor, para que no tenga que navegar al módulo Eco para clasificar ítems individuales.

#### Acceptance Criteria

1. THE ContainerDetailScreen SHALL mostrar el ícono `leaf-outline` con color `Colors.accent` junto al nombre de cada ítem que tenga `eco_action !== null`.
2. THE ContainerDetailScreen SHALL mostrar el ícono Ionicons específico de la `eco_action` asignada (según la tabla de íconos del Requirement 15) junto al nombre del ítem cuando `eco_action` no es NULL, en lugar del ícono genérico `leaf-outline`.
3. WHEN el usuario mantiene pulsado un ítem en ContainerDetailScreen durante más de 500ms, THE ContainerDetailScreen SHALL mostrar un menú contextual con al menos la opción "Acción ecológica" que navegue a `EcoItemDetail` pasando el `itemId` del ítem seleccionado.

---

### Requirement 12: Integración ecológica en CreateItemScreen

**User Story:** Como usuario, quiero poder asignar una acción ecológica al crear un nuevo ítem, para que pueda clasificar objetos ecológicamente desde el momento de su registro.

#### Acceptance Criteria

1. THE CreateItemScreen SHALL incluir una sección colapsable "Acción ecológica (opcional)" al final del formulario, colapsada por defecto, que al expandirse muestre el selector de las seis acciones ecológicas.
2. WHEN el usuario selecciona una acción ecológica en CreateItemScreen y confirma la creación, THE CreateItemScreen SHALL incluir `eco_action` y `eco_status: 'pending'` en los datos enviados a `ItemRepository.create`.
3. WHEN el usuario no selecciona ninguna acción ecológica (sección colapsada o sin selección), THE CreateItemScreen SHALL crear el ítem con `eco_action: null` y `eco_status: null`.

---

### Requirement 13: Filtros ecológicos en SearchScreen

**User Story:** Como usuario, quiero filtrar los resultados de búsqueda por acción ecológica asignada, para que pueda encontrar rápidamente todos los ítems que tengo pendientes de reciclar, donar o vender.

#### Acceptance Criteria

1. THE SearchScreen SHALL mostrar una fila de chips de filtro ecológico con las seis opciones de `EcoAction` (con sus íconos y etiquetas en español) más el chip "Sin clasificar" (para `eco_action IS NULL`).
2. WHEN el usuario activa un chip de `EcoAction`, THE SearchScreen SHALL restringir los resultados a ítems cuyo `eco_action` coincida exactamente con el valor del chip seleccionado.
3. WHEN el usuario activa el chip "Sin clasificar", THE SearchScreen SHALL mostrar únicamente ítems con `eco_action IS NULL`.
4. THE SearchScreen SHALL permitir combinar el filtro ecológico activo con el texto de búsqueda existente, aplicando ambos criterios con operador AND en la consulta a `ItemRepository.search`.
5. WHEN ningún ítem coincide con la combinación de filtros activos, THE SearchScreen SHALL mostrar el componente `EmptyState` existente con el mensaje "No se encontraron ítems con los filtros seleccionados."
6. WHEN el usuario desactiva el chip de filtro ecológico activo (pulsándolo de nuevo), THE SearchScreen SHALL eliminar el filtro ecológico y mostrar los resultados sin esa restricción.

---

### Requirement 14: Gamificación ecológica — sistema de puntos y logros

**User Story:** Como usuario, quiero ganar puntos y desbloquear logros por completar acciones ecológicas, para que la experiencia de gestionar mis objetos en desuso sea motivadora y satisfactoria.

#### Acceptance Criteria

1. THE Sistema SHALL asignar PuntosEco al completar cada acción ecológica según: `reuse` = 10 pts, `repair` = 10 pts, `donate` = 8 pts, `recycle` = 6 pts, `sell` = 5 pts, `discard` = 1 pt.
2. THE EcoRepository.getEcoStats SHALL calcular `ecoPoints` como la suma de los puntos de todas las acciones con `eco_status === 'completed'` aplicando la tabla del criterio 1.
3. THE Sistema SHALL definir los logros desbloqueables por `totalRescued` (ítems con `eco_action !== 'discard'` y `eco_status === 'completed'`): `first_rescue` "Primer Rescate" (1), `guardian_verde` "Guardián Verde" (5), `eco_heroe` "Eco Héroe" (10), `maestro_reciclaje` "Maestro del Reciclaje" (25), `leyenda_sostenible` "Leyenda Sostenible" (50), `campeon_planeta` "Campeón del Planeta" (100).
4. WHEN el usuario desbloquea un logro, THE EcoHub SHALL mostrar un banner de notificación no bloqueante con el nombre del logro durante 3 segundos usando `Animated` de React Native, sin interrumpir la navegación.
5. THE EcoHub SHALL mostrar una sección "Mis Logros" con todos los `achievements` desbloqueados, mostrando para cada uno: ícono `trophy-outline` (Ionicons), nombre del logro, descripción y fecha de desbloqueo formateada como `DD/MM/YYYY`.
6. THE Dashboard SHALL mostrar el total de `ecoPoints` del usuario en una `StatCard` con ícono `leaf-outline` y color `Colors.accent`, añadida al final de las `StatCard` existentes.
7. WHEN el `type` de un logro ya está presente en `achievements` al evaluar desbloqueos, THE useEcoStore SHALL omitir la llamada a `EcoRepository.saveAchievement` y no mostrar la notificación de desbloqueo.

---

### Requirement 15: Consistencia visual del módulo ecológico

**User Story:** Como usuario, quiero que el módulo Reciclador Inteligente tenga una identidad visual coherente con el resto de San Alejo, para que la experiencia sea premium y unificada.

#### Acceptance Criteria

1. THE Reciclador_Inteligente SHALL usar `Colors.accent` como color primario de todos sus elementos interactivos (botones de acción, íconos activos, barras de progreso, chips de filtro activos).
2. THE Reciclador_Inteligente SHALL usar el gradiente `Colors.gradients.accent` (`['#00D4AA', '#6C63FF']`) en los encabezados de EcoHub y en las tarjetas de estadísticas destacadas.
3. THE Reciclador_Inteligente SHALL usar los íconos de Ionicons con la siguiente asignación fija: `leaf`/`leaf-outline` para el tab Eco y la acción `recycle`, `refresh-outline` para `reuse`, `heart-outline` para `donate`, `pricetag-outline` para `sell`, `construct-outline` para `repair`, `trash-outline` para `discard`.
4. THE Reciclador_Inteligente SHALL aplicar glassmorphism en las tarjetas de ítem de EcoClassify y EcoHub usando `backgroundColor: Colors.glass` y `borderColor: Colors.glassBorder` con `borderWidth: 1`.
5. THE Reciclador_Inteligente SHALL mantener `Colors.background` como fondo de todas sus pantallas y `Colors.surface` / `Colors.surfaceElevated` para las tarjetas secundarias.
6. THE Reciclador_Inteligente SHALL usar exclusivamente los componentes UI existentes: `Button`, `Card`, `EmptyState`, `FloatingActionButton`, `PremiumInput`, `ScreenContainer`, `SectionHeader`, `SkeletonLoader`, `StatCard`, `TagBadge`, `Text`, sin añadir nuevas dependencias de UI externas al `package.json`.
