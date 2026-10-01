# 01 — ESTADO DEL PROYECTO

## 1. Limitación importante de este análisis

Este paquete se generó **sin acceso a Roblox Studio**. Solo se contaba con el historial de la conversación, y **no se pudo comparar con el proyecto actual**.
Por eso la tabla de §4 ("qué sobrevivió / qué se perdió") la debe **rellenar la nueva ventana en la Sección 0** (auditoría de solo lectura), usando la lista de §3 como referencia.

---

## 2. Estado final alcanzado en el desarrollo original (DEV `86748110736040`)

### 2.1 Resumen

El jardín de 160 parcelas fue reemplazado por un **suelo único (`GardenFloor`) de libre colocación**. Se añadió un sistema de **Garden Level (1–100) con Resonancia**. Se dejó **activa** la migración de datos viejos y se **eliminaron** las parcelas y los sistemas de parcelas.

### 2.2 Hechos confirmados del estado final

**Plantado y mundo**
- [CONFIRMADO] Las 160 parcelas de `Garden_001` fueron eliminadas. El jardín bajó de ~20.600 instancias a **1.898**.
- [CONFIRMADO] Las plantas se apoyan sobre el **`GardenFloor`** (antes en `BaseSoil`). El usuario **subió el suelo** manualmente y por eso se cambió la referencia de altura.
- [CONFIRMADO] El **tutorial** guía su rayo (beam) hacia el `GardenFloor`, no hacia una parcela.
- [CONFIRMADO] Se quitaron restos de parcelas del **sonido de los sprinklers** y de la **Sweet Frog**.
- [CONFIRMADO] Existen en el jardín las carpetas **`Eggs`** y **`Sprinklers`**, y el cartel **`GardenLevel`** (deben clonarse junto con `Garden_001`).
- [CONFIRMADO] Los **huevos** los reubica el sistema de huevos en **soportes**, conservando su incubación (no pasan por la migración).

**Sistemas eliminados**
- [CONFIRMADO] `PlotSystem` (incluye `PlotService` y `PlotRegistry`).
- [CONFIRMADO] `HighlightBridge`.
- [CONFIRMADO] `ParcelDetector`.
- [CONFIRMADO] `AccessService.canAccessPlot`.
- [CONFIRMADO] Verificación final: **523 scripts compilan sin errores** y **ningún script activo referencia lo eliminado**.

**Consola**
- [CONFIRMADO] **`GamepadController` NO fue adaptado.** Se dejó para una update de consola: no da error, pero no encuentra nada que seleccionar.

**Garden Level y Resonancia**
- [CONFIRMADO] Implementado (5 scripts que compilan). Los números están en **`GardenProgressConfig`**.

**Migración**
- [CONFIRMADO] **`GardenMigration`** (ModuleScript) con `ENABLED = true`. La llama `PlantGrowthSystem.Init` justo **antes** de cargar las plantas de cada jugador, una sola vez por jugador. Hay una protección extra en `restorePlants`.
- [CONFIRMADO] **Nunca se probó con datos reales**, porque el DEV no tenía datos con formato de parcelas.

**Pendiente al cierre del desarrollo**
- Clonar `Garden_001` a N jardines (con IDs únicos).
- Probar la migración.
- Revisar los tiempos de prueba (crecimiento de plantas; Sweet Egg ya en 5 h).
- Revisar el balance de XP.

### 2.3 Sistemas cuyo estado final NO consta en el historial

| Sistema | Qué falta saber |
|---|---|
| `PlotPresenceService` (PlotPresenceSystem) | La orden decía "desaparece". El reporte final no lo lista entre lo eliminado. **UNKNOWN**: ¿eliminado en una fase anterior o aún presente? |
| `ParcelInteractionService` | Igual que el anterior. **UNKNOWN**. |
| `ParcelGameplayService` (parcelas bloqueadas, D7) | El agente preguntó si desaparecía. Respuesta no registrada. **UNKNOWN**. |
| Botón PLOT del cliente | La orden decía "desaparece". No consta. **UNKNOWN**. |
| GUI de detalle de planta al clickear | Pedida en la orden. No consta implementación. **UNKNOWN**. |
| Sprinkler con radio real | `ParcelDetector` se eliminó, así que algo lo reemplazó. Radios por tipo **UNKNOWN**. |
| `TrowelController` (mover plantas) | Propuesta D5 (pickup → click en suelo libre → validar radio). **UNKNOWN** si quedó así. |
| Sistema de soportes de huevos | Existe en el estado final. **UNKNOWN** si se creó en este desarrollo o ya existía. |

---

## 3. Lista de referencia para auditar el proyecto actual

La nueva ventana debe buscar cada elemento (solo lectura) y clasificarlo.

### 3.1 Debe EXISTIR en el estado final
- `workspace.Gardens.Garden_001` (y/o los clones `Garden_00N`) con attribute `GardenID`.
- `GardenFloor` dentro de cada jardín, con attribute `GardenId`.
- Carpeta `Eggs` (con soportes) y carpeta `Sprinklers` en cada jardín.
- Model `GardenLevel` con `SurfaceGui` + Part que contiene un attachment/prompt (ver `03_WORLD_MANUAL_SETUP.md`).
- ModuleScript `GardenProgressConfig`.
- ModuleScript `GardenMigration` (`ENABLED = true`) y su llamada desde `PlantGrowthSystem.Init`.
- `PlantGrowthService` con clave `gardenId:slotId`.
- Modelos de planta nombrados `Plant_<gardenId>:<slotId>` con attribute `slotId` [INFERIDO a partir de la orden y de D4].
- `SeedService`, `PlantHandler`, `ToolUseHandler`, `ToolActionImplementations`, `ToolUseController`, `TrowelController` y `PlantVFXController`, adaptados a XZ/`slotId`.
- `PlantingGuard` (se preserva sin cambios).

### 3.2 Debe NO EXISTIR en el estado final
- Carpeta `Parcels` y cualquier `BaseSoil`, `PlantPivot` o `InteractionPivot`.
- `PlotSystem` (`PlotService`, `PlotRegistry`).
- `HighlightBridge`.
- `ParcelDetector`.
- `AccessService.canAccessPlot`.
- Attributes `_GardenId`, `_PlotId` y `HasPlant`.
- Cualquier referencia a `plotId` fuera de `GardenMigration`.

### 3.3 Búsquedas sugeridas (en todos los scripts)
`BaseSoil`, `PlantPivot`, `InteractionPivot`, `Parcels`, `plotId`, `PlotService`, `PlotRegistry`, `ParcelDetector`, `HighlightBridge`, `canAccessPlot`, `getActivePlot`, `_PlotId`, `HasPlant`, `GardenFloor`, `slotId`, `GardenMigration`, `GardenProgressConfig`, `gardenReworkMigrated`, `Resonat`.

---

## 4. Tabla a rellenar en la Sección 0

| Elemento | Sobrevivió / Parcial / Perdido / Versión antigua / No determinable | Evidencia |
|---|---|---|
| Place DEV `86748110736040` accesible | | |
| `GardenFloor` | | |
| `GardenLevel` (modelo manual) | | |
| `Eggs` / `Sprinklers` (carpetas) | | |
| Modelo de datos `slotId` | | |
| Plantado libre (servidor) | | |
| Plantado libre (cliente PC/Mobile) | | |
| Tools servidor (`slotId`) | | |
| Trowel | | |
| Sprinkler radio real | | |
| Huevos en soportes | | |
| Garden Level + Resonancia | | |
| `GardenMigration` final | | |
| Limpieza de parcelas/sistemas | | |
| Tutorial → `GardenFloor` | | |
| Sweet Frog / sonido sprinkler sin parcelas | | |

**Criterio de "versión antigua"**: por ejemplo, una `GardenMigration` que sea un Script en `PlayerAdded` en lugar de un ModuleScript llamado desde `PlantGrowthSystem.Init`, o que busque la semilla por un campo inexistente (ver `05_BUGS_AND_FINAL_SOLUTIONS.md` B1/B2).
