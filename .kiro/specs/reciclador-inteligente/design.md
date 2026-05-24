qju# Design Document — Reciclador Inteligente

## Overview

El módulo **Reciclador Inteligente** se integra en la app San Alejo como una extensión nativa de su arquitectura existente. No introduce nuevas librerías de UI, no modifica el sistema de navegación personalizado más allá de añadir rutas y un tab, y reutiliza todos los patrones establecidos: repositorios async/await sobre expo-sqlite, stores Zustand con cache local, TabNavigator con `Animated.spring` + haptics, y el sistema de tema oscuro premium.

El color de identidad del módulo es `Colors.accent` (`#00D4AA`) — el teal ya definido en el tema — diferenciándolo visualmente del violeta `Colors.primary` del resto de la app sin romper la coherencia.

---

## Architecture

### Diagrama de capas

```
┌─────────────────────────────────────────────────────────────┐
│  UI Layer (Screens + Components)                            │
│  EcoHub · EcoClassify · EcoItemDetail · EcoHistory          │
│  + modificaciones: ContainerDetailScreen, CreateItemScreen  │
│    SearchScreen, DashboardScreen                            │
├─────────────────────────────────────────────────────────────┤
│  State Layer (Zustand)                                      │
│  useEcoStore  ←→  useItemStore (cache invalidation)         │
├─────────────────────────────────────────────────────────────┤
│  Data Layer (Repositories)                                  │
│  EcoRepository  ←→  ItemRepository (shared items table)     │
├─────────────────────────────────────────────────────────────┤
│  Database Layer (expo-sqlite)                               │
│  items (+ 4 cols eco)  ·  eco_achievements (nueva tabla)    │
└─────────────────────────────────────────────────────────────┘
```

### Principios de integración

- **Sin nuevas dependencias**: todo con `expo-sqlite`, `zustand`, `expo-haptics`, `expo-linear-gradient`, `@expo/vector-icons` (Ionicons) y `react-native` core — ya instalados.
- **Retrocompatibilidad**: los 4 campos eco en `items` son NULL por defecto; todo el código existente que lee `Item` sigue funcionando sin cambios.
- **Patrón repositorio**: `EcoRepository` opera sobre las mismas tablas que `ItemRepository` pero encapsula exclusivamente la lógica ecológica.
- **Navegación**: el sistema usa un stack de historial propio (`NavigationContext.ts`). Las nuevas rutas se añaden al union type `RouteName` y al objeto `RouteParams`, y se registran en `RootNavigator.renderScreen()` con `require()` lazy para evitar ciclos.

---

## Data Models

### 1. Extensión de tipos existentes — `src/types/Item.ts`

```typescript
// Añadir al archivo existente (no reemplazar)
export type EcoAction = 'recycle' | 'donate' | 'sell' | 'reuse' | 'repair' | 'discard';
export type EcoStatus = 'pending' | 'completed' | 'skipped';

// Extender la interfaz Item existente con campos opcionales
export interface Item {
  // ... campos existentes sin cambio ...
  eco_action?: EcoAction;
  eco_notes?: string;
  eco_completed_at?: UnixTimestamp;
  eco_status?: EcoStatus;
}

// Extender UpdateItemInput para permitir actualizaciones eco desde ItemRepository
export interface UpdateItemInput {
  // ... campos existentes sin cambio ...
  eco_action?: EcoAction | null;
  eco_notes?: string | null;
  eco_completed_at?: UnixTimestamp | null;
  eco_status?: EcoStatus | null;
}
```

### 2. Nuevas entidades — `src/types/Eco.ts` (archivo nuevo)

```typescript
import type { UnixTimestamp } from './common';

export interface EcoAchievement {
  id: string;
  type: EcoAchievementType;
  unlocked_at: UnixTimestamp;
  metadata?: string;
}

export type EcoAchievementType =
  | 'first_rescue'
  | 'guardian_verde'
  | 'eco_heroe'
  | 'maestro_reciclaje'
  | 'leyenda_sostenible'
  | 'campeon_planeta';

export interface EcoStats {
  totalPending: number;
  totalCompleted: number;
  totalSkipped: number;
  totalDiscarded: number;   // eco_action='discard' AND eco_status='completed'
  totalRescued: number;     // eco_action!='discard' AND eco_status='completed'
  ecoPoints: number;        // suma ponderada de puntos (ver tabla de puntos)
}

// Constantes de configuración del módulo
export const ECO_ACTION_LABELS: Record<EcoAction, string> = {
  recycle: 'Reciclar',
  donate: 'Donar',
  sell: 'Vender',
  reuse: 'Reutilizar',
  repair: 'Reparar',
  discard: 'Desechar',
};

export const ECO_ACTION_ICONS: Record<EcoAction, string> = {
  recycle: 'leaf-outline',
  donate: 'heart-outline',
  sell: 'pricetag-outline',
  reuse: 'refresh-outline',
  repair: 'construct-outline',
  discard: 'trash-outline',
};

export const ECO_ACTION_POINTS: Record<EcoAction, number> = {
  reuse: 10,
  repair: 10,
  donate: 8,
  recycle: 6,
  sell: 5,
  discard: 1,
};

export const ECO_ACHIEVEMENT_THRESHOLDS: Record<EcoAchievementType, number> = {
  first_rescue: 1,
  guardian_verde: 5,
  eco_heroe: 10,
  maestro_reciclaje: 25,
  leyenda_sostenible: 50,
  campeon_planeta: 100,
};

export const ECO_ACHIEVEMENT_LABELS: Record<EcoAchievementType, { name: string; description: string }> = {
  first_rescue:       { name: 'Primer Rescate',        description: 'Completaste tu primera acción ecológica.' },
  guardian_verde:     { name: 'Guardián Verde',         description: 'Rescataste 5 objetos del desecho.' },
  eco_heroe:          { name: 'Eco Héroe',              description: 'Rescataste 10 objetos del desecho.' },
  maestro_reciclaje:  { name: 'Maestro del Reciclaje',  description: 'Rescataste 25 objetos del desecho.' },
  leyenda_sostenible: { name: 'Leyenda Sostenible',     description: 'Rescataste 50 objetos del desecho.' },
  campeon_planeta:    { name: 'Campeón del Planeta',    description: 'Rescataste 100 objetos del desecho.' },
};
```

---

## Database Schema Changes

### Migration v4 — `src/database/migrations.ts`

Se añade como cuarto elemento del array `migrations[]` existente:

```typescript
{
  version: 4,
  up: async (db) => {
    // Campos ecológicos en items
    await db.execAsync(
      "ALTER TABLE items ADD COLUMN eco_action TEXT DEFAULT NULL CHECK(eco_action IN ('recycle','donate','sell','reuse','repair','discard') OR eco_action IS NULL);"
    );
    await db.execAsync(
      'ALTER TABLE items ADD COLUMN eco_notes TEXT DEFAULT NULL;'
    );
    await db.execAsync(
      'ALTER TABLE items ADD COLUMN eco_completed_at INTEGER DEFAULT NULL;'
    );
    await db.execAsync(
      "ALTER TABLE items ADD COLUMN eco_status TEXT DEFAULT NULL CHECK(eco_status IN ('pending','completed','skipped') OR eco_status IS NULL);"
    );

    // Nueva tabla de logros
    await db.execAsync(`
      CREATE TABLE IF NOT EXISTS eco_achievements (
        id           TEXT PRIMARY KEY,
        type         TEXT NOT NULL CHECK(type IN (
                       'first_rescue','guardian_verde','eco_heroe',
                       'maestro_reciclaje','leyenda_sostenible','campeon_planeta'
                     )),
        unlocked_at  INTEGER NOT NULL,
        metadata     TEXT
      );
    `);

    // Índices de rendimiento
    await db.execAsync(
      'CREATE INDEX IF NOT EXISTS idx_items_eco_action ON items(eco_action);'
    );
    await db.execAsync(
      'CREATE INDEX IF NOT EXISTS idx_items_eco_status ON items(eco_status);'
    );
  },
},
```

> **Nota de atomicidad**: expo-sqlite ejecuta cada `execAsync` de forma independiente. Para garantizar rollback, la migración debe envolverse en `db.withTransactionAsync()` en la implementación final.

### Actualización de `rowToItem` en `ItemRepository.ts`

```typescript
function rowToItem(row: Record<string, unknown>): Item {
  return {
    // ... campos existentes sin cambio ...
    eco_action:       (row.eco_action as EcoAction)    ?? undefined,
    eco_notes:        (row.eco_notes as string)         ?? undefined,
    eco_completed_at: row.eco_completed_at != null ? Number(row.eco_completed_at) : undefined,
    eco_status:       (row.eco_status as EcoStatus)    ?? undefined,
  };
}
```

---

## Components and Interfaces

### EcoRepository — `src/database/repositories/EcoRepository.ts`

```typescript
import { getDb } from '../db';
import { generateUUID } from '../../utils/uuid';
import { nowTimestamp } from '../../utils/dateUtils';
import type { Item } from '../../types/Item';
import type { EcoAction, EcoStats } from '../../types/Item';
import type { EcoAchievement, EcoAchievementType } from '../../types/Eco';
import { ECO_ACTION_POINTS } from '../../types/Eco';

// Reutiliza rowToItem de ItemRepository (importado)
import { rowToItem } from './ItemRepository'; // exportar rowToItem como named export

export const EcoRepository = {

  async updateEcoAction(itemId: string, action: EcoAction, notes?: string): Promise<void> {
    const db = getDb();
    await db.runAsync(
      'UPDATE items SET eco_action=?, eco_notes=?, eco_status=?, updated_at=? WHERE id=?;',
      [action, notes ?? null, 'pending', nowTimestamp(), itemId]
    );
  },

  async completeEcoAction(itemId: string): Promise<void> {
    const db = getDb();
    const now = nowTimestamp();
    await db.runAsync(
      'UPDATE items SET eco_status=?, eco_completed_at=?, updated_at=? WHERE id=?;',
      ['completed', now, now, itemId]
    );
  },

  async skipEcoAction(itemId: string): Promise<void> {
    const db = getDb();
    await db.runAsync(
      'UPDATE items SET eco_status=?, updated_at=? WHERE id=?;',
      ['skipped', nowTimestamp(), itemId]
    );
  },

  async findByEcoAction(action: EcoAction): Promise<Item[]> {
    const db = getDb();
    const rows = await db.getAllAsync<Record<string, unknown>>(
      'SELECT * FROM items WHERE eco_action=? ORDER BY updated_at DESC;',
      [action]
    );
    return rows.map(rowToItem);
  },

  async findPending(): Promise<Item[]> {
    const db = getDb();
    const rows = await db.getAllAsync<Record<string, unknown>>(
      "SELECT * FROM items WHERE eco_status='pending' ORDER BY updated_at DESC;"
    );
    return rows.map(rowToItem);
  },

  async findUnclassified(limitDays?: number): Promise<Item[]> {
    const db = getDb();
    const base = 'SELECT * FROM items WHERE eco_action IS NULL';
    if (limitDays !== undefined) {
      const cutoff = Date.now() - limitDays * 86_400_000;
      const rows = await db.getAllAsync<Record<string, unknown>>(
        `${base} AND updated_at < ? ORDER BY updated_at ASC;`,
        [cutoff]
      );
      return rows.map(rowToItem);
    }
    const rows = await db.getAllAsync<Record<string, unknown>>(
      `${base} ORDER BY updated_at ASC;`
    );
    return rows.map(rowToItem);
  },

  async findCompleted(): Promise<Item[]> {
    const db = getDb();
    const rows = await db.getAllAsync<Record<string, unknown>>(
      "SELECT * FROM items WHERE eco_status='completed' ORDER BY eco_completed_at DESC;"
    );
    return rows.map(rowToItem);
  },

  async getEcoStats(): Promise<EcoStats> {
    const db = getDb();
    // Una sola query con agregaciones condicionales
    const row = await db.getFirstAsync<Record<string, unknown>>(`
      SELECT
        SUM(CASE WHEN eco_status='pending'                                    THEN 1 ELSE 0 END) AS totalPending,
        SUM(CASE WHEN eco_status='completed'                                  THEN 1 ELSE 0 END) AS totalCompleted,
        SUM(CASE WHEN eco_status='skipped'                                    THEN 1 ELSE 0 END) AS totalSkipped,
        SUM(CASE WHEN eco_status='completed' AND eco_action='discard'         THEN 1 ELSE 0 END) AS totalDiscarded,
        SUM(CASE WHEN eco_status='completed' AND eco_action!='discard'        THEN 1 ELSE 0 END) AS totalRescued,
        SUM(CASE WHEN eco_status='completed' AND eco_action='reuse'    THEN 10
                 WHEN eco_status='completed' AND eco_action='repair'   THEN 10
                 WHEN eco_status='completed' AND eco_action='donate'   THEN 8
                 WHEN eco_status='completed' AND eco_action='recycle'  THEN 6
                 WHEN eco_status='completed' AND eco_action='sell'     THEN 5
                 WHEN eco_status='completed' AND eco_action='discard'  THEN 1
                 ELSE 0 END) AS ecoPoints
      FROM items;
    `);
    return {
      totalPending:   Number(row?.totalPending   ?? 0),
      totalCompleted: Number(row?.totalCompleted ?? 0),
      totalSkipped:   Number(row?.totalSkipped   ?? 0),
      totalDiscarded: Number(row?.totalDiscarded ?? 0),
      totalRescued:   Number(row?.totalRescued   ?? 0),
      ecoPoints:      Number(row?.ecoPoints      ?? 0),
    };
  },

  async saveAchievement(achievement: Omit<EcoAchievement, 'id'>): Promise<EcoAchievement> {
    const db = getDb();
    const id = generateUUID();
    await db.runAsync(
      'INSERT INTO eco_achievements (id, type, unlocked_at, metadata) VALUES (?,?,?,?);',
      [id, achievement.type, achievement.unlocked_at, achievement.metadata ?? null]
    );
    return { id, ...achievement };
  },

  async findAllAchievements(): Promise<EcoAchievement[]> {
    const db = getDb();
    const rows = await db.getAllAsync<Record<string, unknown>>(
      'SELECT * FROM eco_achievements ORDER BY unlocked_at DESC;'
    );
    return rows.map((r) => ({
      id:           r.id as string,
      type:         r.type as EcoAchievementType,
      unlocked_at:  Number(r.unlocked_at),
      metadata:     (r.metadata as string) ?? undefined,
    }));
  },
};
```

### useEcoStore — `src/store/ecoStore.ts`

```typescript
import { create } from 'zustand';
import { EcoRepository } from '../database/repositories/EcoRepository';
import type { Item } from '../types/Item';
import type { EcoAction } from '../types/Item';
import type { EcoAchievement, EcoAchievementType, EcoStats } from '../types/Eco';
import {
  ECO_ACHIEVEMENT_THRESHOLDS,
  ECO_ACHIEVEMENT_LABELS,
} from '../types/Eco';
import { nowTimestamp } from '../utils/dateUtils';

interface EcoState {
  ecoStats:          EcoStats | null;
  pendingItems:      Item[];
  completedItems:    Item[];
  unclassifiedItems: Item[];
  achievements:      EcoAchievement[];
  isLoading:         boolean;
  error:             string | null;

  // Acciones de carga (gestionan isLoading)
  loadEcoStats:           () => Promise<void>;
  loadPendingItems:       () => Promise<void>;
  loadUnclassifiedItems:  (limitDays?: number) => Promise<void>;
  loadCompletedItems:     () => Promise<void>;
  loadAchievements:       () => Promise<void>;

  // Acciones de mutación (NO gestionan isLoading)
  assignEcoAction:  (itemId: string, action: EcoAction, notes?: string) => Promise<void>;
  completeEcoAction:(itemId: string) => Promise<void>;
  skipItem:         (itemId: string) => Promise<void>;

  clearError: () => void;
}

export const useEcoStore = create<EcoState>((set, get) => ({
  ecoStats:          null,
  pendingItems:      [],
  completedItems:    [],
  unclassifiedItems: [],
  achievements:      [],
  isLoading:         false,
  error:             null,

  loadEcoStats: async () => {
    set({ isLoading: true, error: null });
    try {
      const ecoStats = await EcoRepository.getEcoStats();
      set({ ecoStats, isLoading: false });
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al cargar estadísticas', isLoading: false });
    }
  },

  loadPendingItems: async () => {
    set({ isLoading: true, error: null });
    try {
      const pendingItems = await EcoRepository.findPending();
      set({ pendingItems, isLoading: false });
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al cargar pendientes', isLoading: false });
    }
  },

  loadUnclassifiedItems: async (limitDays?: number) => {
    set({ isLoading: true, error: null });
    try {
      const unclassifiedItems = await EcoRepository.findUnclassified(limitDays);
      set({ unclassifiedItems, isLoading: false });
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al cargar sin clasificar', isLoading: false });
    }
  },

  loadCompletedItems: async () => {
    set({ isLoading: true, error: null });
    try {
      const completedItems = await EcoRepository.findCompleted();
      set({ completedItems, isLoading: false });
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al cargar historial', isLoading: false });
    }
  },

  loadAchievements: async () => {
    set({ isLoading: true, error: null });
    try {
      const achievements = await EcoRepository.findAllAchievements();
      set({ achievements, isLoading: false });
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al cargar logros', isLoading: false });
    }
  },

  assignEcoAction: async (itemId, action, notes) => {
    try {
      await EcoRepository.updateEcoAction(itemId, action, notes);
      // Mover de unclassifiedItems → pendingItems en estado local
      const { unclassifiedItems, pendingItems } = get();
      const movedItem = unclassifiedItems.find((i) => i.id === itemId);
      if (movedItem) {
        const updated = { ...movedItem, eco_action: action, eco_notes: notes, eco_status: 'pending' as const };
        set({
          unclassifiedItems: unclassifiedItems.filter((i) => i.id !== itemId),
          pendingItems: [updated, ...pendingItems],
        });
      }
      // Recargar stats
      const ecoStats = await EcoRepository.getEcoStats();
      set({ ecoStats });
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al asignar acción' });
    }
  },

  completeEcoAction: async (itemId) => {
    try {
      await EcoRepository.completeEcoAction(itemId);
      const { pendingItems, completedItems } = get();
      const completedItem = pendingItems.find((i) => i.id === itemId);
      const now = nowTimestamp();
      if (completedItem) {
        const updated = { ...completedItem, eco_status: 'completed' as const, eco_completed_at: now };
        set({
          pendingItems: pendingItems.filter((i) => i.id !== itemId),
          completedItems: [updated, ...completedItems],
        });
      }
      // Recargar stats y evaluar logros
      const ecoStats = await EcoRepository.getEcoStats();
      set({ ecoStats });
      await _checkAndUnlockAchievements(ecoStats.totalRescued, get);
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al completar acción' });
    }
  },

  skipItem: async (itemId) => {
    try {
      await EcoRepository.skipEcoAction(itemId);
      set((state) => ({
        unclassifiedItems: state.unclassifiedItems.filter((i) => i.id !== itemId),
      }));
    } catch (err) {
      set({ error: err instanceof Error ? err.message : 'Error al omitir ítem' });
    }
  },

  clearError: () => set({ error: null }),
}));

// Helper privado — evalúa y desbloquea logros
async function _checkAndUnlockAchievements(
  totalRescued: number,
  get: () => EcoState
): Promise<void> {
  const { achievements } = get();
  const unlockedTypes = new Set(achievements.map((a) => a.type));

  for (const [type, threshold] of Object.entries(ECO_ACHIEVEMENT_THRESHOLDS) as [EcoAchievementType, number][]) {
    if (totalRescued >= threshold && !unlockedTypes.has(type)) {
      const saved = await EcoRepository.saveAchievement({
        type,
        unlocked_at: nowTimestamp(),
      });
      useEcoStore.setState((state) => ({
        achievements: [saved, ...state.achievements],
        // El banner de notificación se dispara desde EcoHub observando achievements
      }));
      break; // desbloquear de uno en uno por sesión
    }
  }
}
```

---

## Navigation Changes

### 1. `src/navigation/NavigationContext.ts` — ampliar tipos

```typescript
// Añadir al union RouteName:
export type RouteName =
  | 'MainTabs'
  | 'ContainerDetail'
  | 'Search'
  | 'CreateContainer'
  | 'EditContainer'
  | 'CreateItem'
  | 'Dashboard'
  | 'Settings'
  | 'Tags'
  | 'Locations'
  | 'EcoHub'          // nuevo
  | 'EcoClassify'     // nuevo
  | 'EcoItemDetail'   // nuevo
  | 'EcoHistory';     // nuevo

// RouteParams ya tiene itemId?: string — no requiere cambio
```

### 2. `src/navigation/RootNavigator.tsx` — registrar pantallas

```typescript
// Añadir en renderScreen() con el mismo patrón lazy require:
case 'EcoHub': {
  const EcoHubScreen = require('../screens/EcoHubScreen').default;
  return <EcoHubScreen />;
}
case 'EcoClassify': {
  const EcoClassifyScreen = require('../screens/EcoClassifyScreen').default;
  return <EcoClassifyScreen />;
}
case 'EcoItemDetail': {
  const EcoItemDetailScreen = require('../screens/EcoItemDetailScreen').default;
  return <EcoItemDetailScreen />;
}
case 'EcoHistory': {
  const EcoHistoryScreen = require('../screens/EcoHistoryScreen').default;
  return <EcoHistoryScreen />;
}
```

### 3. `src/navigation/TabNavigator.tsx` — tercer tab

```typescript
// Añadir Colors.accentGlow al tema (ver sección Theme Changes)
// Ampliar el tipo local TabName:
type TabName = 'Home' | 'Dashboard' | 'Eco';

// Añadir al array TABS:
{ name: 'Eco', label: 'Eco', icon: 'leaf-outline', iconActive: 'leaf' }

// En TabItem, el color activo se determina por tab:
const activeColor = tab.name === 'Eco' ? Colors.accent : Colors.primary;
const activeGlow  = tab.name === 'Eco' ? Colors.accentGlow : Colors.primaryGlow;

// En el renderizado del contenido:
{activeTab === 'Eco' && <EcoHubScreen />}
// (import lazy con require para evitar ciclos, igual que RootNavigator)
```

### 4. `src/theme/colors.ts` — nuevo token

```typescript
// Añadir junto a primaryGlow:
accentGlow: '#003D30',   // fondo del ícono activo del tab Eco
```

---

## Screen Designs

### EcoHubScreen — `src/screens/EcoHubScreen.tsx`

**Estructura de layout** (ScrollView con ScreenContainer):

```
┌─ Header gradient (Colors.gradients.accent) ──────────────────┐
│  "Reciclador Inteligente"  [botón historial →]               │
├─ Sección: Impacto Ecológico ──────────────────────────────────┤
│  [StatCard: Rescatados] [StatCard: Completados]              │
│  [StatCard: Pendientes]  [StatCard: PuntosEco 🌿]            │
├─ Barra de progreso eco (totalRescued / total clasificados) ───┤
├─ Sección: Pendientes ─────────────────────────────────────────┤
│  Lista agrupada por eco_action con ícono + etiqueta          │
│  Cada ítem → navega a EcoItemDetail                          │
├─ Sección: Logros Recientes ───────────────────────────────────┤
│  Últimos 3 achievements (trophy-outline + nombre + fecha)    │
└───────────────────────────────────────────────────────────────┘
[FloatingActionButton: leaf-outline → EcoClassify]
```

**Lógica de montaje**:
```typescript
useEffect(() => {
  Promise.all([
    loadEcoStats(),
    loadPendingItems(),
    loadAchievements(),
  ]);
}, []);

// Detectar nuevo logro para mostrar banner
const prevAchievementsCount = useRef(achievements.length);
useEffect(() => {
  if (achievements.length > prevAchievementsCount.current) {
    const newest = achievements[0];
    showAchievementBanner(newest); // Animated banner 3s
    prevAchievementsCount.current = achievements.length;
  }
}, [achievements]);
```

**Banner de logro** (sin librerías externas):
```typescript
// Animated.Value para opacity + translateY
// Auto-dismiss a los 3000ms con Animated.timing
// Posición: top absoluto bajo el header, z-index elevado
```

---

### EcoClassifyScreen — `src/screens/EcoClassifyScreen.tsx`

**Estructura de layout** (pantalla de una sola tarjeta):

```
┌─ Header: "Ítem X de N"  [✕ cerrar] ──────────────────────────┐
├─ Tarjeta glassmorphism del ítem actual ───────────────────────┤
│  [Imagen o ícono]  Nombre  Descripción  Contenedor           │
├─ Grid de acciones ecológicas (2×3) ───────────────────────────┤
│  [🌿 Reciclar] [❤️ Donar]   [🏷️ Vender]                     │
│  [🔄 Reutilizar] [🔧 Reparar] [🗑️ Desechar]                  │
├─ Botón "Omitir" (ghost) ──────────────────────────────────────┤
└───────────────────────────────────────────────────────────────┘
```

**Estado local**:
```typescript
const [currentIndex, setCurrentIndex] = useState(0);
const [classifiedCount, setClassifiedCount] = useState(0);
const currentItem = unclassifiedItems[currentIndex];

const handleAction = async (action: EcoAction) => {
  await assignEcoAction(currentItem.id, action);
  setClassifiedCount((c) => c + 1);
  Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
  setCurrentIndex((i) => i + 1);
};

const handleSkip = async () => {
  await skipItem(currentItem.id);
  Haptics.selectionAsync();
  setCurrentIndex((i) => i + 1);
};
```

**Pantalla de resumen** (cuando `currentIndex >= unclassifiedItems.length`):
```
"¡Sesión completada!"
"Clasificaste N ítems"
[Botón "Finalizar" → goBack()]
```

---

### EcoItemDetailScreen — `src/screens/EcoItemDetailScreen.tsx`

**Estructura de layout** (ScrollView):

```
┌─ Header: [← volver]  Nombre del ítem ────────────────────────┐
├─ Imagen de portada (si existe) ───────────────────────────────┤
├─ Selector de acción ecológica (6 chips horizontales) ─────────┤
├─ PremiumInput: eco_notes (placeholder contextual) ────────────┤
├─ Botón "Guardar cambios" (Colors.accent) ─────────────────────┤
├─ Separador ───────────────────────────────────────────────────┤
├─ Botón "Marcar como completado" (solo si eco_status≠completed)┤
└───────────────────────────────────────────────────────────────┘
```

**Estado local**:
```typescript
const itemId = params?.itemId ?? '';
const [item, setItem] = useState<Item | null>(null);
const [selectedAction, setSelectedAction] = useState<EcoAction | undefined>();
const [notes, setNotes] = useState('');
const [saveConfirm, setSaveConfirm] = useState(false); // feedback visual

useEffect(() => {
  ItemRepository.findById(itemId).then((found) => {
    if (!found) return; // mostrar error state
    setItem(found);
    setSelectedAction(found.eco_action);
    setNotes(found.eco_notes ?? '');
  });
}, [itemId]);
```

**Modo solo lectura** cuando `item.eco_status === 'completed'`:
- Chips de acción deshabilitados (opacity 0.5, no interactivos)
- PremiumInput con `editable={false}`
- Mostrar fecha: `eco_completed_at` formateada como `DD/MM/YYYY`
- Ocultar botón "Marcar como completado"

---

### EcoHistoryScreen — `src/screens/EcoHistoryScreen.tsx`

**Estructura de layout** (FlatList):

```
┌─ Header: "Historial Eco" ─────────────────────────────────────┐
├─ Resumen: "Rescatados: N  |  Desechados: N" ──────────────────┤
├─ FlatList de ítems completados ───────────────────────────────┤
│  Cada fila: [ícono eco_action] Nombre  eco_notes  Fecha       │
└───────────────────────────────────────────────────────────────┘
```

Usa `useEcoStore.loadCompletedItems()` en `useEffect` al montar.

---

## Modifications to Existing Screens

### ContainerDetailScreen — `ItemCard` component

**Cambio 1**: Mostrar badge eco en el ítem.

```typescript
// En ItemCard, después del bloque de imagen/ícono existente:
{item.eco_action && (
  <View style={styles.ecoBadge}>
    <Ionicons
      name={ECO_ACTION_ICONS[item.eco_action] as any}
      size={10}
      color={Colors.accent}
    />
  </View>
)}
// ecoBadge: posición absoluta top-right sobre el thumbnail/iconWrapper
```

**Cambio 2**: Ampliar `handleLongPress` para incluir opción eco.

```typescript
const handleLongPress = () => {
  Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);
  Alert.alert(item.name, '¿Qué deseas hacer con este ítem?', [
    { text: 'Cancelar', style: 'cancel' },
    {
      text: 'Acción ecológica',
      onPress: () => navigate('EcoItemDetail', { itemId: item.id }),
    },
    { text: 'Eliminar', style: 'destructive', onPress: () => onDelete(item.id) },
  ]);
};
```

---

### CreateItemScreen — sección eco colapsable

**Nuevo estado**:
```typescript
const [ecoExpanded, setEcoExpanded] = useState(false);
const [selectedEcoAction, setSelectedEcoAction] = useState<EcoAction | undefined>();
```

**Nuevo bloque en el formulario** (al final, antes del `bottomSpacer`):
```typescript
<TouchableOpacity
  style={styles.ecoToggle}
  onPress={() => { setEcoExpanded((v) => !v); Haptics.selectionAsync(); }}
  activeOpacity={0.75}
>
  <Ionicons name="leaf-outline" size={16} color={Colors.accent} />
  <Text variant="labelMedium" color={Colors.accent} style={styles.ecoToggleLabel}>
    Acción ecológica (opcional)
  </Text>
  <Ionicons
    name={ecoExpanded ? 'chevron-up' : 'chevron-down'}
    size={14}
    color={Colors.textTertiary}
  />
</TouchableOpacity>

{ecoExpanded && (
  <View style={styles.ecoGrid}>
    {ECO_ACTIONS.map((action) => (
      <TouchableOpacity
        key={action}
        style={[styles.ecoChip, selectedEcoAction === action && styles.ecoChipActive]}
        onPress={() => { setSelectedEcoAction(action); Haptics.selectionAsync(); }}
        activeOpacity={0.75}
      >
        <Ionicons name={ECO_ACTION_ICONS[action] as any} size={14}
          color={selectedEcoAction === action ? Colors.accent : Colors.textTertiary} />
        <Text style={[styles.ecoChipLabel,
          { color: selectedEcoAction === action ? Colors.accent : Colors.textTertiary }]}>
          {ECO_ACTION_LABELS[action]}
        </Text>
      </TouchableOpacity>
    ))}
  </View>
)}
```

**En `handleSubmit`**, añadir al payload:
```typescript
await createItem({
  // ... campos existentes ...
  eco_action: selectedEcoAction ?? null,
  eco_status: selectedEcoAction ? 'pending' : null,
});
```

---

### SearchScreen — filtros eco

**Nuevo estado**:
```typescript
const [activeEcoFilter, setActiveEcoFilter] = useState<EcoAction | 'unclassified' | null>(null);
```

**Nueva fila de chips** (debajo de los filtros existentes, solo visible cuando `activeFilter === 'items'` o `'all'`):
```typescript
<ScrollView horizontal showsHorizontalScrollIndicator={false}>
  {(['unclassified', ...ECO_ACTIONS] as const).map((eco) => {
    const isActive = activeEcoFilter === eco;
    const label = eco === 'unclassified' ? 'Sin clasificar' : ECO_ACTION_LABELS[eco];
    const icon  = eco === 'unclassified' ? 'help-circle-outline' : ECO_ACTION_ICONS[eco];
    return (
      <TouchableOpacity
        key={eco}
        style={[styles.filterChip, isActive && styles.filterChipEcoActive]}
        onPress={() => {
          setActiveEcoFilter(isActive ? null : eco);
          Haptics.selectionAsync();
        }}
      >
        <Ionicons name={icon as any} size={13} color={isActive ? Colors.accent : Colors.textTertiary} />
        <Text style={{ color: isActive ? Colors.accent : Colors.textTertiary }}>{label}</Text>
      </TouchableOpacity>
    );
  })}
</ScrollView>
// filterChipEcoActive: backgroundColor: Colors.accentGlow, borderColor: Colors.accent
```

**En `runSearch`**, añadir filtro eco a los ítems:
```typescript
if (activeEcoFilter === 'unclassified') {
  newItems = newItems.filter((i) => i.eco_action == null);
} else if (activeEcoFilter) {
  newItems = newItems.filter((i) => i.eco_action === activeEcoFilter);
}
```

---

### DashboardScreen — StatCard de PuntosEco

```typescript
// Importar useEcoStore
const { ecoStats, loadEcoStats } = useEcoStore();

useEffect(() => { loadEcoStats(); }, []);

// Añadir al bloque de metricsRow (tercera StatCard, fila separada):
{ecoStats && (
  <StatCard
    label="Puntos Eco"
    value={ecoStats.ecoPoints}
    icon="leaf"
    gradient={Colors.gradients.accent}
    subtitle={`${ecoStats.totalRescued} rescatados`}
  />
)}
```

---

## File Structure

Archivos nuevos y modificados que produce esta implementación:

```
san-alejo/src/
│
├── types/
│   ├── Item.ts                          MODIFICAR — añadir EcoAction, EcoStatus, campos eco
│   └── Eco.ts                           NUEVO — EcoAchievement, EcoStats, constantes
│
├── theme/
│   └── colors.ts                        MODIFICAR — añadir accentGlow: '#003D30'
│
├── database/
│   ├── migrations.ts                    MODIFICAR — añadir migration v4
│   ├── repositories/
│   │   ├── ItemRepository.ts            MODIFICAR — exportar rowToItem, mapear campos eco
│   │   └── EcoRepository.ts             NUEVO
│
├── store/
│   └── ecoStore.ts                      NUEVO
│
├── navigation/
│   ├── NavigationContext.ts             MODIFICAR — añadir 4 rutas al RouteName union
│   ├── RootNavigator.tsx                MODIFICAR — registrar 4 pantallas en renderScreen()
│   └── TabNavigator.tsx                 MODIFICAR — añadir tab 'Eco', color accentGlow
│
├── screens/
│   ├── EcoHubScreen.tsx                 NUEVO
│   ├── EcoClassifyScreen.tsx            NUEVO
│   ├── EcoItemDetailScreen.tsx          NUEVO
│   ├── EcoHistoryScreen.tsx             NUEVO
│   ├── ContainerDetailScreen.tsx        MODIFICAR — badge eco + opción en long-press
│   ├── CreateItemScreen.tsx             MODIFICAR — sección eco colapsable
│   ├── SearchScreen.tsx                 MODIFICAR — chips de filtro eco
│   └── DashboardScreen.tsx              MODIFICAR — StatCard PuntosEco
```

**Total**: 4 archivos nuevos, 9 archivos modificados, 0 nuevas dependencias npm.

---

## Performance Considerations

- **`EcoRepository.getEcoStats()`** usa una sola query SQL con `SUM(CASE WHEN ...)` — sin N+1 queries.
- **`findUnclassified(90)`** usa el índice `idx_items_eco_action` (eco_action IS NULL) combinado con el filtro de fecha sobre `updated_at` (indexado por `idx_items_updated_at` de la migración v3).
- **`findPending()`** usa `idx_items_eco_status`.
- **EcoClassifyScreen** carga los ítems una sola vez al montar y navega por ellos con índice local — sin re-queries por ítem.
- **EcoHubScreen** usa `Promise.all` para las tres cargas iniciales en paralelo.
- **useEcoStore** no duplica datos con `useItemStore` — los ítems eco son subconjuntos de la tabla `items` accedidos por queries específicas, no cacheados globalmente.
- **TabNavigator** usa `require()` lazy para `EcoHubScreen` igual que `RootNavigator` hace con las demás pantallas, evitando ciclos de importación.

---

## Visual Identity Summary

| Elemento | Valor |
|---|---|
| Color primario eco | `Colors.accent` = `#00D4AA` |
| Fondo ícono tab activo | `Colors.accentGlow` = `#003D30` (nuevo) |
| Gradiente encabezados | `Colors.gradients.accent` = `['#00D4AA', '#6C63FF']` |
| Glassmorphism fondo | `Colors.glass` = `#1E1E2A` |
| Glassmorphism borde | `Colors.glassBorder` = `#2A2A3A` |
| Ícono tab inactivo | `leaf-outline` (Ionicons) |
| Ícono tab activo | `leaf` (Ionicons) |
| Fondo pantallas | `Colors.background` = `#0A0A0F` |
| Tarjetas | `Colors.surface` = `#16161F` |
| Nuevas dependencias npm | **ninguna** |

---

## Error Handling

| Capa | Escenario | Comportamiento |
|---|---|---|
| Migration v4 | Fallo en cualquier sentencia DDL | Envolver en `withTransactionAsync`; si falla, no registrar versión 4 en `_migrations` |
| EcoRepository | `itemId` no existe en `items` | No-op silencioso (UPDATE afecta 0 filas) — igual que `ItemRepository.update` |
| EcoRepository | Error de BD en lectura/escritura | Propagar excepción al store |
| useEcoStore | Cualquier acción falla | Almacenar mensaje en `error`, no propagar al componente |
| EcoHubScreen | Carga paralela parcialmente fallida | Mostrar banner de error no bloqueante; renderizar el contenido que sí cargó |
| EcoItemDetailScreen | `itemId` inválido o vacío | Mostrar estado de error "Ítem no encontrado" con botón de vuelta |
| EcoClassifyScreen | `unclassifiedItems` vacío al montar | Mostrar `EmptyState` con botón de vuelta a EcoHub |
| ContainerDetailScreen | `navigate('EcoItemDetail')` sin itemId | Imposible — `item.id` siempre existe en el contexto del long-press |

---

## Correctness Properties

### Property 1: Idempotencia de migración

El sistema de migraciones versionado existente garantiza que v4 solo se ejecuta una vez (check en `_migrations`). Si la versión 4 ya está registrada, `runMigrations` la omite sin error.

**Validates: Requirements 1.9**

### Property 2: Retrocompatibilidad de Item

Los 4 campos eco son opcionales en TypeScript y NULL en BD. Todo el código existente que lee `Item` sigue compilando y funcionando sin cambios de comportamiento.

**Validates: Requirements 1.8, 2.3**

### Property 3: Consistencia de estado en assignEcoAction

`assignEcoAction` mueve el ítem de `unclassifiedItems` a `pendingItems` en el estado local antes de recargar stats, evitando que el ítem aparezca en ambas listas simultáneamente.

**Validates: Requirements 4.6**

### Property 4: Unicidad de logros

`_checkAndUnlockAchievements` verifica `unlockedTypes` antes de guardar. Un logro no puede desbloquearse dos veces aunque `completeEcoAction` se llame en rápida sucesión.

**Validates: Requirements 4.10, 14.7**

### Property 5: Puntos calculados en base de datos

`ecoPoints` se calcula en `getEcoStats()` con SQL agregado, no en memoria del store. La fuente de verdad es siempre la base de datos, evitando desincronización entre sesiones.

**Validates: Requirements 2.5, 14.2**

### Property 6: rowToItem como fuente única de mapeo

`EcoRepository` importa `rowToItem` de `ItemRepository` (exportado como named export). Los campos eco se mapean en un único lugar, eliminando duplicación y posibles inconsistencias.

**Validates: Requirements 3.1, 3.2, 3.3**

---

## Testing Strategy

No se requieren tests automáticos en esta fase. Las siguientes verificaciones manuales cubren los flujos críticos:

1. **Migración**: instalar la app con datos existentes → verificar que los ítems previos conservan todos sus campos y que `eco_action`, `eco_status` son NULL.
2. **Flujo completo**: EcoClassify → asignar acción → EcoHub muestra ítem en pendientes → EcoItemDetail → marcar completado → EcoHistory muestra el ítem → Dashboard muestra PuntosEco actualizado.
3. **Logro**: completar 1 ítem rescatado → verificar banner "Primer Rescate" en EcoHub y entrada en sección "Mis Logros".
4. **Filtro eco en Search**: buscar texto + activar chip "Donar" → verificar que solo aparecen ítems con `eco_action='donate'`.
5. **Long-press en ContainerDetail**: mantener pulsado un ítem → verificar opción "Acción ecológica" → navega a EcoItemDetail con el ítem correcto.
6. **CreateItem con eco**: crear ítem con acción "Reciclar" → verificar en BD que `eco_action='recycle'` y `eco_status='pending'`.
