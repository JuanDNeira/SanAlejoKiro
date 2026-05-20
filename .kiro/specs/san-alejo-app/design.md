# Design Document — San Alejo App

## Overview

San Alejo es una aplicación móvil de organización del hogar con una interfaz visual premium estilo Netflix/Apple. Permite catalogar cajas, contenedores, maletas, cajones y objetos físicos mediante una experiencia cinematográfica: tarjetas tipo poster, tema oscuro, glassmorphism, animaciones suaves y un dashboard visual de alto impacto.

### Objetivos de diseño

- **Offline-first**: toda la persistencia ocurre en SQLite local; no hay dependencia de red.
- **Premium UX**: animaciones fluidas (Reanimated v3), glassmorphism, haptic feedback y transiciones compartidas.
- **Mantenibilidad**: arquitectura en capas (UI → Store → Repository → SQLite) con TypeScript estricto.
- **Escalabilidad local**: soporte para hasta 200+ contenedores con render < 100ms.

### Stack tecnológico

| Capa | Tecnología |
|---|---|
| Framework | React Native + Expo SDK (latest stable) |
| Lenguaje | TypeScript (strict mode) |
| Persistencia | Expo SQLite (`expo-sqlite`) |
| Navegación | React Navigation v6 (Stack + Bottom Tabs) |
| Estado global | Zustand |
| Animaciones | React Native Reanimated v3 |
| Estilos | NativeWind + Theme Engine personalizado |
| Imágenes | Expo Image Picker + Expo File System |
| Exportación | Expo File System + Expo Sharing |
| Gestos | React Native Gesture Handler |

---

## Architecture

La arquitectura sigue un patrón de capas desacopladas:

```
┌─────────────────────────────────────────────────────────┐
│                     UI Layer                            │
│  Screens / Components / Theme Engine / Navigator        │
└────────────────────────┬────────────────────────────────┘
                         │ hooks / selectors
┌────────────────────────▼────────────────────────────────┐
│                   State Layer (Zustand)                  │
│  containerStore / itemStore / tagStore / locationStore  │
└────────────────────────┬────────────────────────────────┘
                         │ async actions
┌────────────────────────▼────────────────────────────────┐
│                  Repository Layer                        │
│  ContainerRepository / ItemRepository /                 │
│  LocationRepository / TagRepository                     │
└────────────────────────┬────────────────────────────────┘
                         │ SQL queries
┌────────────────────────▼────────────────────────────────┐
│               Database Layer (SQLite)                    │
│  db.ts (singleton) / schema.ts / migrations.ts          │
└─────────────────────────────────────────────────────────┘
```

### Principios arquitectónicos

1. **Unidirectional data flow**: UI → Store action → Repository → SQLite → Store state → UI re-render.
2. **Repository pattern**: cada entidad tiene su propio repositorio que encapsula todas las queries SQL. Los stores nunca ejecutan SQL directamente.
3. **Singleton DB connection**: `db.ts` expone una única instancia de `SQLiteDatabase` compartida por todos los repositorios.
4. **Optimistic updates**: las operaciones de escritura actualizan el store inmediatamente y persisten en background para máxima responsividad.
5. **Lazy loading**: las imágenes y los sub-contenedores se cargan bajo demanda al navegar al detalle.

### Diagrama de navegación

```
Bottom Tab Bar
├── Tab 1: Dashboard (Stack)
│   ├── DashboardScreen
│   ├── ContainerDetailScreen
│   └── ItemDetailScreen (bottom sheet)
├── Tab 2: SearchScreen
├── Tab 3: LocationsScreen
└── Tab 4: SettingsScreen

Modals (presentados sobre cualquier tab)
├── CreateContainerModal
├── EditContainerModal
├── CreateItemModal
├── EditItemModal
└── TagManagerModal

Onboarding (Stack independiente, solo primer lanzamiento)
└── OnboardingScreen (3 slides)
```

---

## Components and Interfaces

### Estructura de carpetas

```
/src
├── components/
│   ├── common/          # Componentes reutilizables genéricos
│   ├── containers/      # Componentes específicos de Container
│   ├── items/           # Componentes específicos de Item
│   └── ui/              # Primitivos del design system
├── screens/
│   ├── dashboard/
│   ├── containers/
│   ├── items/
│   ├── search/
│   ├── settings/
│   └── onboarding/
├── navigation/
│   ├── RootNavigator.tsx
│   ├── DashboardStack.tsx
│   └── BottomTabNavigator.tsx
├── store/
│   ├── containerStore.ts
│   ├── itemStore.ts
│   ├── tagStore.ts
│   ├── locationStore.ts
│   └── uiStore.ts
├── database/
│   ├── db.ts
│   ├── schema.ts
│   ├── migrations.ts
│   └── repositories/
│       ├── ContainerRepository.ts
│       ├── ItemRepository.ts
│       ├── LocationRepository.ts
│       └── TagRepository.ts
├── hooks/
│   ├── useContainers.ts
│   ├── useItems.ts
│   ├── useSearch.ts
│   └── useCamera.ts
├── utils/
│   ├── uuid.ts
│   ├── dateUtils.ts
│   └── exportUtils.ts
├── assets/
├── theme/
│   ├── colors.ts
│   ├── typography.ts
│   ├── spacing.ts
│   ├── shadows.ts
│   ├── animations.ts
│   └── index.ts
└── types/
    ├── Container.ts
    ├── Item.ts
    ├── Location.ts
    ├── Tag.ts
    └── common.ts
```

### Componentes UI primitivos (`components/ui`)

#### GlassCard
```typescript
interface GlassCardProps {
  blur?: number;                  // default: 20
  borderOpacity?: number;         // default: 0.1
  backgroundOpacity?: number;     // default: 0.05
  children: React.ReactNode;
  style?: StyleProp<ViewStyle>;
}
```

#### PosterCard
```typescript
interface PosterCardProps {
  imageUri?: string;
  title: string;
  subtitle?: string;
  badge?: string | number;
  colorTag?: string;
  containerType?: ContainerType;
  onPress: () => void;
  onLongPress?: () => void;
}
```

#### BottomSheet
```typescript
interface BottomSheetProps {
  snapPoints: number[];           // e.g. [300, 600]
  onClose: () => void;
  children: React.ReactNode;
  initialSnapIndex?: number;
}
```

#### SkeletonLoader
```typescript
interface SkeletonLoaderProps {
  width: number | string;
  height: number;
  borderRadius?: number;
}
```

#### TagChip
```typescript
interface TagChipProps {
  tag: Tag;
  selected?: boolean;
  onPress?: () => void;
  onRemove?: () => void;
}
```

#### FAB
```typescript
interface FABProps {
  onPress: () => void;
  icon: string;                   // Ionicons name
  label?: string;
  position?: 'bottom-right' | 'bottom-center';
}
```

#### SearchBar
```typescript
interface SearchBarProps {
  value: string;
  onChangeText: (text: string) => void;
  onFocus?: () => void;
  onBlur?: () => void;
  placeholder?: string;
}
```

#### EmptyState
```typescript
interface EmptyStateProps {
  illustration: React.ReactNode;
  title: string;
  subtitle?: string;
  ctaLabel?: string;
  onCta?: () => void;
}
```

#### QuantityStepper
```typescript
interface QuantityStepperProps {
  value: number;
  min?: number;                   // default: 1
  max?: number;
  onChange: (value: number) => void;
}
```

#### HapticButton
```typescript
interface HapticButtonProps extends PressableProps {
  hapticStyle?: 'light' | 'medium' | 'heavy';
  children: React.ReactNode;
}
```

### Interfaces de los Stores (Zustand)

#### ContainerStore
```typescript
interface ContainerStore {
  containers: Container[];
  loading: boolean;
  error: string | null;
  // Actions
  fetchRootContainers: () => Promise<void>;
  fetchContainerById: (id: string) => Promise<Container | null>;
  createContainer: (data: CreateContainerInput) => Promise<Container>;
  updateContainer: (id: string, data: UpdateContainerInput) => Promise<void>;
  deleteContainer: (id: string) => Promise<void>;
  moveContainer: (id: string, newParentId: string | null) => Promise<void>;
}
```

#### ItemStore
```typescript
interface ItemStore {
  itemsByContainer: Record<string, Item[]>;
  loading: boolean;
  createItem: (data: CreateItemInput) => Promise<Item>;
  updateItem: (id: string, data: UpdateItemInput) => Promise<void>;
  deleteItem: (id: string) => Promise<void>;
  moveItem: (id: string, newContainerId: string) => Promise<void>;
  fetchItemsByContainer: (containerId: string) => Promise<void>;
}
```

#### SearchStore
```typescript
interface SearchStore {
  query: string;
  results: SearchResults;
  recentContainers: Container[];
  isSearching: boolean;
  setQuery: (q: string) => void;
  search: (q: string) => Promise<void>;
  clearSearch: () => void;
}
```

### Interfaces de los Repositorios

#### ContainerRepository
```typescript
interface IContainerRepository {
  findAll(): Promise<Container[]>;
  findRoots(): Promise<Container[]>;
  findById(id: string): Promise<Container | null>;
  findByParentId(parentId: string): Promise<Container[]>;
  findByTagIds(tagIds: string[]): Promise<Container[]>;
  findByLocationId(locationId: string): Promise<Container[]>;
  create(data: CreateContainerInput): Promise<Container>;
  update(id: string, data: UpdateContainerInput): Promise<void>;
  delete(id: string): Promise<void>;           // cascading
  countItems(id: string): Promise<number>;     // includes nested
  search(query: string): Promise<Container[]>;
}
```

#### ItemRepository
```typescript
interface IItemRepository {
  findByContainerId(containerId: string): Promise<Item[]>;
  findById(id: string): Promise<Item | null>;
  create(data: CreateItemInput): Promise<Item>;
  update(id: string, data: UpdateItemInput): Promise<void>;
  delete(id: string): Promise<void>;
  move(id: string, newContainerId: string): Promise<void>;
  search(query: string): Promise<Item[]>;
}
```

---

## Data Models

### Tipos TypeScript de dominio

```typescript
// types/common.ts
export type UUID = string;
export type UnixTimestamp = number;
export type ContainerType = 'box' | 'suitcase' | 'drawer' | 'shelf' | 'bag' | 'other';

// types/Location.ts
export interface Location {
  id: UUID;
  name: string;
  icon?: string;
  created_at: UnixTimestamp;
}

// types/Tag.ts
export interface Tag {
  id: UUID;
  name: string;
  color: string;
}

// types/Container.ts
export interface Container {
  id: UUID;
  name: string;
  description?: string;
  type: ContainerType;
  location_id?: UUID;
  parent_container_id?: UUID;
  cover_image_uri?: string;
  color_tag?: string;
  created_at: UnixTimestamp;
  updated_at: UnixTimestamp;
  // Relaciones cargadas bajo demanda
  location?: Location;
  tags?: Tag[];
  children?: Container[];
  item_count?: number;
}

export interface CreateContainerInput {
  name: string;
  type: ContainerType;
  description?: string;
  location_id?: UUID;
  parent_container_id?: UUID;
  cover_image_uri?: string;
  color_tag?: string;
}

export type UpdateContainerInput = Partial<CreateContainerInput>;

// types/Item.ts
export interface Item {
  id: UUID;
  name: string;
  description?: string;
  quantity: number;
  container_id: UUID;
  cover_image_uri?: string;
  created_at: UnixTimestamp;
  updated_at: UnixTimestamp;
  // Relaciones cargadas bajo demanda
  tags?: Tag[];
  container?: Container;
}

export interface CreateItemInput {
  name: string;
  description?: string;
  quantity?: number;
  container_id: UUID;
  cover_image_uri?: string;
  tag_ids?: UUID[];
}

export type UpdateItemInput = Partial<Omit<CreateItemInput, 'container_id'>>;
```

### Esquema SQLite

```sql
-- locations
CREATE TABLE IF NOT EXISTS locations (
  id          TEXT PRIMARY KEY,
  name        TEXT NOT NULL,
  icon        TEXT,
  created_at  INTEGER NOT NULL
);

-- containers
CREATE TABLE IF NOT EXISTS containers (
  id                   TEXT PRIMARY KEY,
  name                 TEXT NOT NULL,
  description          TEXT,
  type                 TEXT NOT NULL CHECK(type IN ('box','suitcase','drawer','shelf','bag','other')),
  location_id          TEXT REFERENCES locations(id) ON DELETE SET NULL,
  parent_container_id  TEXT REFERENCES containers(id) ON DELETE CASCADE,
  cover_image_uri      TEXT,
  color_tag            TEXT,
  created_at           INTEGER NOT NULL,
  updated_at           INTEGER NOT NULL
);

-- items
CREATE TABLE IF NOT EXISTS items (
  id               TEXT PRIMARY KEY,
  name             TEXT NOT NULL,
  description      TEXT,
  quantity         INTEGER NOT NULL DEFAULT 1,
  container_id     TEXT NOT NULL REFERENCES containers(id) ON DELETE CASCADE,
  cover_image_uri  TEXT,
  created_at       INTEGER NOT NULL,
  updated_at       INTEGER NOT NULL
);

-- tags
CREATE TABLE IF NOT EXISTS tags (
  id     TEXT PRIMARY KEY,
  name   TEXT NOT NULL UNIQUE,
  color  TEXT NOT NULL
);

-- container_tags (junction)
CREATE TABLE IF NOT EXISTS container_tags (
  container_id  TEXT NOT NULL REFERENCES containers(id) ON DELETE CASCADE,
  tag_id        TEXT NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (container_id, tag_id)
);

-- item_tags (junction)
CREATE TABLE IF NOT EXISTS item_tags (
  item_id  TEXT NOT NULL REFERENCES items(id) ON DELETE CASCADE,
  tag_id   TEXT NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (item_id, tag_id)
);
```

### Esquema de exportación JSON

```typescript
interface ExportSchema {
  version: number;           // versión del esquema de exportación
  exported_at: UnixTimestamp;
  locations: Location[];
  tags: Tag[];
  containers: ContainerExport[];
  items: ItemExport[];
  container_tags: { container_id: UUID; tag_id: UUID }[];
  item_tags: { item_id: UUID; tag_id: UUID }[];
}

interface ContainerExport extends Container {
  // cover_image_uri apunta a URI local; en exportación se omite o se incluye como base64
  cover_image_data?: string;  // base64 opcional
}

interface ItemExport extends Item {
  cover_image_data?: string;
}
```

### Sistema de migraciones

```typescript
// database/migrations.ts
interface Migration {
  version: number;
  up: (db: SQLiteDatabase) => Promise<void>;
}

// La tabla `_migrations` registra las versiones aplicadas:
// CREATE TABLE _migrations (version INTEGER PRIMARY KEY, applied_at INTEGER)
```

---

## Correctness Properties

*Una propiedad es una característica o comportamiento que debe mantenerse verdadero en todas las ejecuciones válidas del sistema — esencialmente, una declaración formal sobre lo que el sistema debe hacer. Las propiedades sirven como puente entre las especificaciones legibles por humanos y las garantías de corrección verificables por máquina.*

La biblioteca de property-based testing seleccionada es **fast-check** (TypeScript/JavaScript), configurada con un mínimo de 100 iteraciones por propiedad.

---

### Property 1: Container creation round-trip

*Para cualquier* entrada válida de Container (nombre no vacío, tipo válido, campos opcionales arbitrarios), crear el Container en SQLite y luego leerlo por su ID debe producir un objeto Container con todos los campos equivalentes a los originales.

**Validates: Requirements 3.9, 6.3**

---

### Property 2: Item creation round-trip

*Para cualquier* entrada válida de Item (nombre no vacío, quantity ≥ 1, container_id existente), crear el Item en SQLite y luego leerlo por su ID debe producir un objeto Item con todos los campos equivalentes a los originales.

**Validates: Requirements 7.3**

---

### Property 3: Name validation rejects empty and whitespace inputs

*Para cualquier* cadena de texto compuesta únicamente de caracteres de espacio en blanco (incluyendo la cadena vacía, espacios, tabs y saltos de línea), intentar crear un Container o Item con ese nombre debe ser rechazado por la capa de validación, y el estado de la base de datos debe permanecer sin cambios.

**Validates: Requirements 6.4, 7.4**

---

### Property 4: Migration preserves existing data

*Para cualquier* estado válido de la base de datos (conjunto arbitrario de Containers, Items, Locations y Tags), aplicar una migración de esquema incremental debe preservar todos los registros existentes con sus valores originales intactos.

**Validates: Requirements 3.8**

---

### Property 5: Cascading delete removes all descendants

*Para cualquier* árbol de Containers (con profundidad y ramificación arbitrarias) que contenga Items en cualquier nodo, eliminar el Container raíz debe resultar en que ningún Container descendiente ni ningún Item de esos descendientes permanezca en la base de datos.

**Validates: Requirements 6.8**

---

### Property 6: Recursive item count invariant

*Para cualquier* árbol de Containers con Items distribuidos arbitrariamente en sus nodos, el `countItems(rootId)` debe ser igual a la suma de todos los Items en todos los nodos del subárbol (incluyendo el nodo raíz y todos sus descendientes a cualquier profundidad).

**Validates: Requirements 6.10**

---

### Property 7: Move item updates container membership

*Para cualquier* Item en un Container origen y cualquier Container destino distinto, mover el Item debe resultar en que el Item aparezca en la lista de Items del Container destino y no aparezca en la lista del Container origen, con su `container_id` actualizado al ID del destino.

**Validates: Requirements 7.8**

---

### Property 8: Search results contain query string

*Para cualquier* consulta de búsqueda no vacía, todos los Containers e Items retornados por el Search_Engine deben tener su campo `name` o `description` conteniendo la cadena de búsqueda (comparación case-insensitive).

**Validates: Requirements 8.2**

---

### Property 9: Search results grouped — containers before items

*Para cualquier* consulta de búsqueda que retorne una mezcla de Containers e Items, todos los resultados de tipo Container deben aparecer antes que todos los resultados de tipo Item en la lista de resultados.

**Validates: Requirements 8.3**

---

### Property 10: Tag filter returns only containers with ALL selected tags

*Para cualquier* conjunto no vacío de Tags seleccionados como filtro, todos los Containers retornados por el filtro deben tener asignados TODOS los Tags del conjunto seleccionado (semántica AND). Ningún Container retornado puede carecer de alguno de los Tags del filtro.

**Validates: Requirements 9.4**

---

### Property 11: Tag assignment persists in junction table

*Para cualquier* Container o Item y cualquier Tag existente, asignar el Tag debe crear exactamente un registro en la tabla de junction correspondiente (`container_tags` o `item_tags`), y la asignación debe ser idempotente (asignar el mismo Tag dos veces no crea duplicados).

**Validates: Requirements 9.2**

---

### Property 12: Delete location nullifies container location_id

*Para cualquier* Location con cualquier número de Containers asignados, eliminar esa Location debe resultar en que todos los Containers que tenían esa `location_id` ahora tengan `location_id = NULL`, sin modificar ningún otro campo de esos Containers.

**Validates: Requirements 10.4**

---

### Property 13: Theme color contrast ratio ≥ 4.5:1

*Para cualquier* par de colores (texto, fondo) definido en el Theme_Engine, el ratio de contraste WCAG calculado debe ser mayor o igual a 4.5:1.

**Validates: Requirements 11.4, 17.4**

---

### Property 14: Image resize enforces max 1024×1024

*Para cualquier* imagen de dimensiones arbitrarias (incluyendo imágenes muy grandes, cuadradas, apaisadas y verticales), después de ser procesada por el Camera_Module, tanto el ancho como el alto de la imagen resultante deben ser menores o iguales a 1024 píxeles, preservando el aspect ratio original.

**Validates: Requirements 13.4**

---

### Property 15: Export/import round-trip

*Para cualquier* estado válido de la base de datos (conjunto arbitrario de Containers, Items, Locations, Tags y sus relaciones), exportar a JSON e importar ese JSON en una base de datos vacía debe producir un estado de base de datos equivalente al original, con todos los registros y relaciones preservados.

**Validates: Requirements 14.6**

---

### Property 16: UI primitive components are crash-safe with invalid props

*Para cualquier* combinación de props inválidos o ausentes pasados a cualquier componente primitivo del design system (`GlassCard`, `PosterCard`, `FAB`, `BottomSheet`, `SkeletonLoader`, `TagChip`, `SearchBar`, `EmptyState`, `QuantityStepper`, `HapticButton`), el componente no debe lanzar una excepción en modo producción (puede mostrar un estado degradado o vacío).

**Validates: Requirements 16.6**

---

## Error Handling

### Estrategia general

Todos los errores se propagan hacia arriba desde la capa de Repository hasta el Store, donde se capturan y se exponen como `error: string | null` en el estado. La UI reacciona al estado de error mostrando mensajes inline o toasts.

```
Repository throws → Store catches → Store sets error state → UI renders error UI
```

### Errores por capa

#### Database Layer
| Escenario | Manejo |
|---|---|
| DB no inicializada | `db.ts` lanza `DatabaseNotInitializedError`; la app muestra pantalla de error fatal |
| Fallo de migración | Se hace rollback de la migración fallida; se preservan los datos existentes; se loguea el error |
| Constraint violation (FK, UNIQUE) | El Repository captura el error SQLite y lanza un error de dominio tipado |
| Disco lleno | El Repository captura el error y lo propaga; la UI muestra alerta de espacio insuficiente |

#### Repository Layer
```typescript
// Errores de dominio tipados
class ContainerNotFoundError extends Error { constructor(id: string) { super(`Container ${id} not found`) } }
class DuplicateTagNameError extends Error { constructor(name: string) { super(`Tag "${name}" already exists`) } }
class InvalidContainerTypeError extends Error { constructor(type: string) { super(`Invalid container type: ${type}`) } }
class LocationNotFoundError extends Error { constructor(id: string) { super(`Location ${id} not found`) } }
```

#### Store Layer
- Todas las acciones async están envueltas en `try/catch`.
- Los errores se almacenan en `store.error` y se limpian al iniciar la siguiente acción.
- Los errores de validación (nombre vacío, tipo inválido) se manejan antes de llamar al Repository.

#### UI Layer
- **Errores de validación de formulario**: mensajes inline debajo del campo afectado, sin cerrar el modal.
- **Errores de operación** (fallo al guardar, fallo al borrar): Toast/Snackbar con mensaje descriptivo y opción de reintentar.
- **Errores fatales** (DB no inicializable): pantalla de error con instrucciones para el usuario.
- **Permisos denegados** (cámara, galería): Alert con enlace a configuración del dispositivo.

### Validaciones de entrada

```typescript
// utils/validation.ts
export const validateContainerName = (name: string): string | null => {
  if (!name || name.trim().length === 0) return 'El nombre es requerido';
  if (name.trim().length > 100) return 'El nombre no puede superar 100 caracteres';
  return null;
};

export const validateItemName = (name: string): string | null => {
  if (!name || name.trim().length === 0) return 'El nombre es requerido';
  if (name.trim().length > 100) return 'El nombre no puede superar 100 caracteres';
  return null;
};

export const validateQuantity = (qty: number): string | null => {
  if (!Number.isInteger(qty) || qty < 0) return 'La cantidad debe ser un número entero no negativo';
  return null;
};

export const validateExportJSON = (data: unknown): data is ExportSchema => {
  // Verifica version, arrays requeridos, tipos de campos
  // Retorna false si el formato es inválido
};
```

### Manejo de imágenes

- Si `cover_image_uri` apunta a un archivo que ya no existe en el sistema de archivos, la UI muestra un placeholder en lugar de crashear.
- Las imágenes se procesan en background; si el resize falla, se usa la imagen original sin resize.

---

## Testing Strategy

### Enfoque dual

La estrategia combina **tests de ejemplo** (casos concretos, edge cases, integración) con **tests de propiedad** (fast-check, 100+ iteraciones) para cobertura comprensiva.

### Property-Based Testing con fast-check

**Biblioteca**: `fast-check` (npm package, TypeScript nativo)
**Configuración**: mínimo 100 iteraciones por propiedad (`numRuns: 100`)
**Tag format**: `// Feature: san-alejo-app, Property {N}: {property_text}`

Cada propiedad del documento de diseño se implementa como un único test de propiedad:

```typescript
// Ejemplo de implementación — Property 1
import fc from 'fast-check';
import { ContainerRepository } from '../database/repositories/ContainerRepository';
import { getTestDb } from './testUtils';

// Feature: san-alejo-app, Property 1: Container creation round-trip
test('Container creation round-trip', async () => {
  await fc.assert(
    fc.asyncProperty(
      fc.record({
        name: fc.string({ minLength: 1 }).filter(s => s.trim().length > 0),
        type: fc.constantFrom('box', 'suitcase', 'drawer', 'shelf', 'bag', 'other'),
        description: fc.option(fc.string(), { nil: undefined }),
        color_tag: fc.option(fc.hexaString({ minLength: 6, maxLength: 6 }).map(h => `#${h}`), { nil: undefined }),
      }),
      async (input) => {
        const db = await getTestDb();
        const repo = new ContainerRepository(db);
        const created = await repo.create(input);
        const fetched = await repo.findById(created.id);
        expect(fetched).not.toBeNull();
        expect(fetched!.name).toBe(input.name.trim());
        expect(fetched!.type).toBe(input.type);
      }
    ),
    { numRuns: 100 }
  );
});
```

### Tests de ejemplo (Jest + React Native Testing Library)

**Casos cubiertos por tests de ejemplo:**
- Onboarding: primer lanzamiento muestra slides, skip/complete persiste flag
- Dashboard: empty state cuando no hay containers, FAB visible, toggle grid/list
- Container Detail: hero header, badge de item count, sub-containers visibles
- Formularios: campos pre-poblados en edición, submit con datos válidos
- Navegación: back button, tab switching, modal dismiss
- Cámara: action sheet con dos opciones, manejo de permiso denegado
- Búsqueda: empty state cuando no hay resultados, sugerencias recientes
- Exportación: share sheet se abre, error con JSON inválido

### Tests de integración

**Casos cubiertos:**
- Inicialización de DB: schema creado correctamente, seed de "Sin ubicación"
- Migración: aplicar migración sobre DB con datos existentes
- Cascading delete: verificar que sub-containers e items se eliminan
- Export/import: ciclo completo con datos reales en DB de test

### Tests de smoke (configuración)

- Verificar que todas las dependencias del stack están instaladas
- Verificar que el esquema SQLite tiene todas las tablas y columnas requeridas
- Verificar que el Theme_Engine exporta todos los tokens requeridos

### Organización de tests

```
/src
└── __tests__/
    ├── properties/          # fast-check property tests
    │   ├── container.property.test.ts
    │   ├── item.property.test.ts
    │   ├── search.property.test.ts
    │   ├── tags.property.test.ts
    │   ├── export.property.test.ts
    │   └── theme.property.test.ts
    ├── unit/                # Jest example-based tests
    │   ├── validation.test.ts
    │   ├── repositories/
    │   └── utils/
    ├── integration/         # Tests con DB real
    │   ├── database.test.ts
    │   └── export.test.ts
    └── components/          # React Native Testing Library
        ├── GlassCard.test.tsx
        ├── PosterCard.test.tsx
        └── ...
```

### Cobertura objetivo

| Capa | Tipo de test | Objetivo |
|---|---|---|
| Repositorios | Property + Integration | 90%+ |
| Stores (Zustand) | Unit (mocked repos) | 80%+ |
| Validaciones | Property | 100% |
| Componentes UI | Example (RNTL) | 70%+ |
| Export/Import | Property + Integration | 95%+ |
| Theme Engine | Property (contraste) | 100% |

### Herramientas

| Herramienta | Uso |
|---|---|
| Jest | Test runner principal |
| fast-check | Property-based testing |
| React Native Testing Library | Tests de componentes |
| expo-sqlite (in-memory) | DB de test aislada |
| jest-expo | Preset de Jest para Expo |
