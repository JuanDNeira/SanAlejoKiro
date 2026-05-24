# Implementation Plan: Reciclador Inteligente

## Overview

Plan de implementación incremental del módulo **Reciclador Inteligente** dentro de la app San Alejo. Las tareas siguen un orden de dependencias estricto: primero la capa de datos (migración, tipos, repositorio), luego el estado (store), después la navegación, y finalmente las pantallas y las integraciones con pantallas existentes. Cada tarea es autónoma y verificable antes de pasar a la siguiente.

**Stack**: expo-sqlite · Zustand · React Native Animated · Ionicons · expo-haptics · expo-linear-gradient — todos ya instalados, sin nuevas dependencias.

---

## Task Dependency Graph

```json
{
  "waves": [
    { "wave": 1, "tasks": ["1"] },
    { "wave": 2, "tasks": ["2"] },
    { "wave": 3, "tasks": ["3"] },
    { "wave": 4, "tasks": ["4"] },
    { "wave": 5, "tasks": ["5"] },
    { "wave": 6, "tasks": ["6", "7", "8", "9"] },
    { "wave": 7, "tasks": ["10", "11", "12", "13"] },
    { "wave": 8, "tasks": ["14"] }
  ]
}
```

Las tareas 6–9 pueden ejecutarse en paralelo una vez completada la tarea 5.
Las tareas 10–13 pueden ejecutarse en paralelo una vez completadas las tareas 5–9.
La tarea 14 requiere que todas las anteriores estén completas.

---

## Notes

- **Sin nuevas dependencias npm**: todo el módulo usa librerías ya instaladas (`expo-sqlite`, `zustand`, `expo-haptics`, `expo-linear-gradient`, `@expo/vector-icons`).
- **Retrocompatibilidad garantizada**: los 4 campos eco en `items` son NULL por defecto; el código existente que lee `Item` no requiere cambios.
- **Patrón `require()` lazy**: las nuevas pantallas eco deben registrarse en `RootNavigator.renderScreen()` y en `TabNavigator` usando `require()` en lugar de imports estáticos, igual que las pantallas existentes, para evitar ciclos de importación.
- **`rowToItem` como named export**: la tarea 2 debe exportar `rowToItem` de `ItemRepository.ts` como named export para que `EcoRepository` (tarea 3) pueda importarlo sin duplicar la lógica de mapeo.
- **Token `accentGlow`**: se añade en la tarea 2 (junto con la migración) porque es necesario para el tab Eco (tarea 5) y los chips activos en pantallas eco (tareas 6–12).
- **Orden de verificación en tarea 14**: ejecutar primero el flujo de migración (datos existentes), luego el flujo completo de clasificación, y finalmente las integraciones en pantallas existentes.

---

## Tasks

- [ ] 1. Tipos TypeScript del módulo ecológico
  - Añadir `EcoAction` y `EcoStatus` como tipos exportados en `src/types/Item.ts`
  - Extender la interfaz `Item` con los campos opcionales: `eco_action?: EcoAction`, `eco_notes?: string`, `eco_completed_at?: UnixTimestamp`, `eco_status?: EcoStatus`
  - Extender `UpdateItemInput` con los mismos 4 campos opcionales (aceptando `null` para borrar)
  - Crear `src/types/Eco.ts` con: interfaz `EcoAchievement`, tipo `EcoAchievementType`, interfaz `EcoStats`, y las constantes `ECO_ACTION_LABELS`, `ECO_ACTION_ICONS`, `ECO_ACTION_POINTS`, `ECO_ACHIEVEMENT_THRESHOLDS`, `ECO_ACHIEVEMENT_LABELS`
  - Verificar que el código existente que usa `Item` sigue compilando sin errores TypeScript
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ] 2. Migración de base de datos v4
  - Añadir el nuevo token de color `accentGlow: '#003D30'` en `src/theme/colors.ts` junto a `primaryGlow`
  - Añadir la migration v4 al array `migrations` en `src/database/migrations.ts` con `version: 4`
  - La migración debe ejecutarse dentro de `db.withTransactionAsync()` para garantizar atomicidad
  - Añadir las 4 columnas eco a `items`: `eco_action` (TEXT, CHECK constraint), `eco_notes` (TEXT), `eco_completed_at` (INTEGER), `eco_status` (TEXT, CHECK constraint)
  - Crear la tabla `eco_achievements` con columnas `id`, `type` (CHECK constraint con los 6 tipos), `unlocked_at`, `metadata`
  - Crear los índices `idx_items_eco_action` e `idx_items_eco_status`
  - Actualizar `rowToItem` en `ItemRepository.ts` para mapear los 4 nuevos campos y exportarlo como named export
  - Verificar manualmente que la app arranca sin errores y que los ítems existentes conservan sus datos
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 1.8, 1.9, 1.10_

- [ ] 3. EcoRepository
  - Crear `src/database/repositories/EcoRepository.ts` importando `rowToItem` desde `ItemRepository`
  - Implementar `updateEcoAction(itemId, action, notes?)`: actualiza `eco_action`, `eco_notes` (NULL si notes es undefined), `eco_status='pending'` y `updated_at`
  - Implementar `completeEcoAction(itemId)`: establece `eco_status='completed'`, `eco_completed_at=now`, `updated_at=now`
  - Implementar `skipEcoAction(itemId)`: establece `eco_status='skipped'`, `updated_at=now`
  - Implementar `findByEcoAction(action)`: SELECT con `eco_action=?` ORDER BY `updated_at DESC`
  - Implementar `findPending()`: SELECT con `eco_status='pending'` ORDER BY `updated_at DESC`
  - Implementar `findUnclassified(limitDays?)`: SELECT con `eco_action IS NULL`; si `limitDays` se provee, añadir filtro `updated_at < cutoff`
  - Implementar `findCompleted()`: SELECT con `eco_status='completed'` ORDER BY `eco_completed_at DESC`
  - Implementar `getEcoStats()`: una sola query SQL con `SUM(CASE WHEN ...)` para calcular los 6 campos de `EcoStats` incluyendo `ecoPoints` con la tabla de puntos
  - Implementar `saveAchievement(achievement)`: INSERT en `eco_achievements` con UUID generado, retorna el registro completo
  - Implementar `findAllAchievements()`: SELECT ORDER BY `unlocked_at DESC`
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10, 3.11, 3.12_

- [ ] 4. useEcoStore
  - Crear `src/store/ecoStore.ts` con el store Zustand siguiendo el patrón de `itemStore.ts` y `containerStore.ts`
  - Definir el estado: `ecoStats`, `pendingItems`, `completedItems`, `unclassifiedItems`, `achievements`, `isLoading`, `error`, `clearError`
  - Implementar las 5 acciones de carga (`loadEcoStats`, `loadPendingItems`, `loadUnclassifiedItems`, `loadCompletedItems`, `loadAchievements`): cada una establece `isLoading: true` al inicio y `isLoading: false` al finalizar
  - Implementar `assignEcoAction`: llama a `EcoRepository.updateEcoAction`, mueve el ítem de `unclassifiedItems` a `pendingItems` en estado local, recarga `ecoStats`
  - Implementar `completeEcoAction`: llama a `EcoRepository.completeEcoAction`, mueve el ítem de `pendingItems` a `completedItems`, recarga `ecoStats`, llama al helper de logros
  - Implementar `skipItem`: llama a `EcoRepository.skipEcoAction`, elimina el ítem de `unclassifiedItems`
  - Implementar el helper privado `_checkAndUnlockAchievements(totalRescued, get)`: itera `ECO_ACHIEVEMENT_THRESHOLDS`, verifica que el tipo no esté ya en `achievements`, guarda y añade al estado si corresponde
  - Las acciones de mutación (`assignEcoAction`, `completeEcoAction`, `skipItem`) NO modifican `isLoading`; los errores se almacenan en `error`
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9, 4.10, 4.11, 4.12, 4.13_

- [ ] 5. Navegación — nuevas rutas y tab Eco
  - Añadir las 4 nuevas rutas al union `RouteName` en `src/navigation/NavigationContext.ts`: `'EcoHub'`, `'EcoClassify'`, `'EcoItemDetail'`, `'EcoHistory'`
  - Verificar que `RouteParams` ya tiene `itemId?: string` (no requiere cambio)
  - Registrar las 4 pantallas en `renderScreen()` de `RootNavigator.tsx` usando el patrón `require()` lazy existente
  - Añadir el tab `'Eco'` al array `TABS` en `TabNavigator.tsx` con `icon: 'leaf-outline'`, `iconActive: 'leaf'`, `label: 'Eco'`
  - Actualizar el tipo local `TabName` en `TabNavigator.tsx` para incluir `'Eco'`
  - En `TabItem`, determinar el color activo y el glow según el tab: `Colors.accent` / `Colors.accentGlow` para `'Eco'`, `Colors.primary` / `Colors.primaryGlow` para los demás
  - Añadir el bloque de renderizado `{activeTab === 'Eco' && <EcoHubScreen />}` en el área de contenido del `TabNavigator` (con `require()` lazy)
  - Verificar que los tabs "Inicio" y "Panel" siguen funcionando sin cambios
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 6.1, 6.2, 6.3_

- [ ] 6. EcoHubScreen — pantalla principal del módulo
  - Crear `src/screens/EcoHubScreen.tsx`
  - Al montar, llamar `Promise.all([loadEcoStats(), loadPendingItems(), loadAchievements()])`
  - Mientras `isLoading`, mostrar `SkeletonLoader` en el área de contenido
  - Sección "Impacto Ecológico": cuatro `StatCard` con `totalRescued`, `totalCompleted`, `totalPending`, `ecoPoints`; usar `Colors.accent` como color de acento
  - Barra de progreso animada: valor `totalRescued / (totalRescued + totalDiscarded)` cuando el denominador > 0, si no mostrar 0%; usar `Animated.timing` con `useNativeDriver: false`
  - Sección "Pendientes": lista de `pendingItems` agrupados por `eco_action` con ícono y etiqueta; al pulsar un ítem navegar a `EcoItemDetail` con `itemId`
  - Sección "Logros Recientes": mostrar los últimos 3 `achievements` (o menos si no hay suficientes) con ícono `trophy-outline`, nombre y fecha `DD/MM/YYYY`
  - Cuando `pendingItems` y `unclassifiedItems` están vacíos, mostrar `EmptyState` con el mensaje "Todo en orden. No tienes ítems pendientes de clasificar."
  - `FloatingActionButton` con ícono `leaf-outline` que navega a `EcoClassify`
  - Banner de logro: detectar cuando `achievements.length` aumenta usando `useRef` para el conteo previo; mostrar banner animado (fade + slide) con nombre del logro durante 3 segundos
  - Aplicar gradiente `Colors.gradients.accent` en el encabezado; usar `ScreenContainer` y `SectionHeader`
  - Si alguna carga falla, mostrar el `error` del store en un banner no bloqueante
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8, 7.9, 7.10, 7.11, 7.12, 14.4, 14.5, 15.1, 15.2_

- [ ] 7. EcoClassifyScreen — flujo de clasificación
  - Crear `src/screens/EcoClassifyScreen.tsx`
  - Al montar, llamar `loadUnclassifiedItems(90)` para cargar candidatos a desuso
  - Si `unclassifiedItems` está vacío, mostrar `EmptyState` con "No hay ítems candidatos a clasificar en este momento." y botón "Volver" que llama `goBack()`
  - Mantener estado local `currentIndex` (número) y `classifiedCount` (número)
  - Mostrar en el encabezado "Ítem X de N" donde X = `currentIndex + 1` y N = total de candidatos cargados al montar
  - Tarjeta del ítem actual con glassmorphism (`Colors.glass` / `Colors.glassBorder`): imagen de portada (si existe), nombre, descripción (si existe), nombre del contenedor (cargar con `ContainerRepository.findById`)
  - Grid 2×3 de botones de acción ecológica: cada botón muestra el ícono de `ECO_ACTION_ICONS` y la etiqueta de `ECO_ACTION_LABELS`
  - Al seleccionar una acción: llamar `assignEcoAction(item.id, action)`, incrementar `classifiedCount`, avanzar `currentIndex`, feedback háptico `NotificationFeedbackType.Success`
  - Botón "Omitir": llamar `skipItem(item.id)`, avanzar `currentIndex`, feedback háptico `selectionAsync`
  - Cuando `currentIndex >= unclassifiedItems.length`: mostrar pantalla de resumen con "Clasificaste N ítems" y botón "Finalizar" que llama `goBack()`
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9_

- [ ] 8. EcoItemDetailScreen — detalle y edición de acción ecológica
  - Crear `src/screens/EcoItemDetailScreen.tsx`
  - Leer `itemId` desde `params?.itemId`; si está vacío o el ítem no existe, mostrar estado de error "Ítem no encontrado" con botón "Volver a Eco" que navega a `EcoHub`
  - Al montar, cargar el ítem con `ItemRepository.findById(itemId)` e inicializar estado local `selectedAction` y `notes`
  - Mostrar imagen de portada (si existe), nombre y descripción del ítem
  - Selector de acción: 6 chips con ícono + etiqueta; el chip activo usa `Colors.accent` como color y `Colors.accentGlow` como fondo
  - `PremiumInput` para `eco_notes` con placeholder contextual según la acción seleccionada: "Precio estimado (EUR)" para `sell`, "Punto de donación" para `donate`, "Descripción de la reparación" para `repair`, "Notas adicionales" para el resto
  - Botón "Guardar cambios": llama `assignEcoAction(itemId, selectedAction, notes)`; feedback visual cambiando el color del botón a `Colors.success` durante 1,5 segundos con `setTimeout`
  - Botón "Marcar como completado" (solo visible cuando `eco_status !== 'completed'`): llama `completeEcoAction(itemId)`, actualiza el estado local, muestra animación de escala pulsante en el ícono de acción durante 600ms
  - Modo solo lectura cuando `eco_status === 'completed'`: chips y `PremiumInput` deshabilitados, mostrar fecha de completado formateada como `DD/MM/YYYY`
  - Ícono de la acción actual con `Colors.accent`; si `eco_action` es null, mostrar `help-circle-outline` con `Colors.textTertiary`
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7, 9.8_

- [ ] 9. EcoHistoryScreen — historial de acciones completadas
  - Crear `src/screens/EcoHistoryScreen.tsx`
  - Al montar, llamar `loadCompletedItems()`
  - Mientras `isLoading`, mostrar `SkeletonLoader`
  - Encabezado con dos contadores: "Rescatados: N" (items con `eco_action !== 'discard'`) y "Desechados: N" (items con `eco_action === 'discard'`)
  - `FlatList` de `completedItems`: cada fila muestra ícono de `eco_action` (color `Colors.accent` si no es `discard`, `Colors.textTertiary` si es `discard`), nombre del ítem, etiqueta de la acción en español, `eco_notes` (si existe), fecha `DD/MM/YYYY` derivada de `eco_completed_at`
  - Cuando `completedItems` está vacío, mostrar `EmptyState` con "Aún no has completado ninguna acción ecológica."
  - Botón de retroceso en el encabezado que llama `goBack()`
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

- [ ] 10. Integración en ContainerDetailScreen
  - En el componente `ItemCard` de `ContainerDetailScreen.tsx`, añadir un badge eco: ícono de `ECO_ACTION_ICONS[item.eco_action]` con `Colors.accent`, posicionado en la esquina superior derecha del thumbnail/iconWrapper, visible solo cuando `item.eco_action !== null`
  - Ampliar `handleLongPress` en `ItemCard` para incluir la opción "Acción ecológica" en el `Alert.alert`, que al pulsarse navega a `'EcoItemDetail'` con `{ itemId: item.id }`
  - Mantener la opción "Eliminar" existente en el mismo menú contextual
  - Verificar que el long-press sigue funcionando con el mismo delay de 400ms y haptic `ImpactFeedbackStyle.Medium`
  - _Requirements: 11.1, 11.2, 11.3_

- [ ] 11. Integración en CreateItemScreen
  - Añadir estado local `ecoExpanded: boolean` (false por defecto) y `selectedEcoAction: EcoAction | undefined` en `CreateItemScreen.tsx`
  - Añadir al final del formulario (antes del `bottomSpacer`) un `TouchableOpacity` colapsable con ícono `leaf-outline`, etiqueta "Acción ecológica (opcional)" y chevron up/down; al pulsar, alternar `ecoExpanded` con haptic `selectionAsync`
  - Cuando `ecoExpanded === true`, mostrar un grid de 6 chips de acción ecológica (ícono + etiqueta); el chip seleccionado usa `Colors.accent` y `Colors.accentGlow`
  - En `handleSubmit`, incluir `eco_action: selectedEcoAction ?? null` y `eco_status: selectedEcoAction ? 'pending' : null` en el payload de `createItem`
  - Verificar que el formulario existente (nombre, descripción, cantidad, tags) sigue funcionando sin cambios
  - _Requirements: 12.1, 12.2, 12.3_

- [ ] 12. Integración en SearchScreen
  - Añadir estado local `activeEcoFilter: EcoAction | 'unclassified' | null` (null por defecto) en `SearchScreen.tsx`
  - Añadir una segunda fila de chips de filtro debajo de los filtros existentes, visible cuando `activeFilter === 'all'` o `activeFilter === 'items'`
  - Los chips eco incluyen: "Sin clasificar" (ícono `help-circle-outline`) + los 6 valores de `EcoAction` con sus íconos y etiquetas en español
  - El chip activo usa `Colors.accent` como color de texto/ícono y `Colors.accentGlow` como fondo; el chip inactivo usa `Colors.textTertiary`
  - Al pulsar un chip activo de nuevo, desactivarlo (toggle)
  - En `runSearch`, después de obtener `newItems`, aplicar el filtro eco: si `activeEcoFilter === 'unclassified'` filtrar `eco_action == null`; si es un `EcoAction`, filtrar `eco_action === activeEcoFilter`
  - El filtro eco se combina con el texto de búsqueda existente (AND lógico)
  - Cuando no hay resultados con los filtros combinados, mostrar el `EmptyState` existente
  - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.5, 13.6_

- [ ] 13. Integración en DashboardScreen
  - Importar `useEcoStore` en `DashboardScreen.tsx`
  - Llamar `loadEcoStats()` en el `useEffect` de carga inicial (en paralelo con las cargas existentes)
  - Añadir una `StatCard` de "Puntos Eco" con `value={ecoStats?.ecoPoints ?? 0}`, `icon="leaf"`, `gradient={Colors.gradients.accent}`, `subtitle` con el número de ítems rescatados
  - Mostrar la `StatCard` eco solo cuando `ecoStats !== null`
  - Verificar que las `StatCard` existentes ("Contenedores" e "Ítems totales") no se ven afectadas
  - _Requirements: 14.6_

- [ ] 14. Verificación final de integración
  - Arrancar la app y verificar que los 3 tabs ("Inicio", "Panel", "Eco") se muestran correctamente con sus colores diferenciados
  - Verificar el flujo completo: EcoClassify → asignar acción → EcoHub muestra ítem en pendientes → EcoItemDetail → marcar completado → EcoHistory muestra el ítem → Dashboard muestra PuntosEco actualizado
  - Verificar que completar 1 ítem rescatado muestra el banner "Primer Rescate" en EcoHub y aparece en "Mis Logros"
  - Verificar que el filtro eco en SearchScreen funciona combinado con búsqueda por texto
  - Verificar que el long-press en ContainerDetailScreen muestra la opción "Acción ecológica"
  - Verificar que crear un ítem con acción eco desde CreateItemScreen lo registra con `eco_status='pending'`
  - Verificar que la migración v4 no rompe datos existentes (ítems previos tienen `eco_action=null`)
  - _Requirements: 1.8, 5.2, 7.10, 8.1, 9.5, 11.2, 12.2, 14.4_
