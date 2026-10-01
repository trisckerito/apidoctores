# 01 — ESTADO DEL PROYECTO

## 1. Qué pasó
- Todo se implementó en la copia DEV (`86748110736040`) y quedó funcionando y verificado (523 scripts compilando, sin referencias rotas).
- Después, **Roblox Studio forzó un cierre de emergencia y se perdió todo lo implementado desde el inicio de la conversación.**
- Ahora el usuario decidió **rehacerlo en el juego principal** (`85407374189603`).

## 2. Estado esperado del juego principal al empezar (pre-rework)
Lo esperable es que el principal tenga el sistema **viejo completo**. Confirmarlo en la Sección 0 con la tabla del §4:
- 8 jardines `Garden_001`…`Garden_008` en `workspace.Gardens`, cada uno con su carpeta `Parcels` (160 modelos: `BaseSoil` + `PlantPivot` + `InteractionPivot` + pointer) y un Part `Water` decorativo de 92×92 studs con un Script de animación dentro.
- Jardines con posiciones y rotaciones distintas (en el DEV: Garden_001 rotY ≈ −44°, Garden_002 ≈ −89°, Garden_005 y Garden_007 ≈ +45°).
- Sistemas viejos activos: `PlotSystem` (`PlotService`/`PlotRegistry`), `PlotPresenceSystem`, `ParcelInteractionSystem`, `ParcelGameplaySystem`, `ParcelGuiSystem` (`ParcelGuiBridge`, `ParcelUnlockHandler`, `ParcelGuiController`), `HighlightSystem`, `AbilitySystem.ParcelDetector`, `DeletePlantHandler`, botón `PARCELButton`, `ParcelGui` y `ParcelDetailsButtonGui`.
- Huevos colocados sobre `BaseSoil`/`plotId`.
- ~12.400 partes con `CanQuery = true`, ~2.370 con `CastShadow = true`, Water con scripts de servidor, `GiroScript` de sprinklers en el servidor, `FruitAnimation` del StarCube en el servidor y `RagdollTester`.

## 3. Estado FINAL que se había alcanzado (objetivo a reproducir)

**Mundo**
- `GardenFloor` (un Part por jardín) donde se planta en cualquier punto. Garden_001 con estilo propio del usuario (posición/rotación/altura/color/material ajustados a mano).
- Cuadrícula de agua decorativa (12 `Water`) en Garden_001, sin scripts dentro; un único `WaterAnimator` en el cliente las anima.
- Carpeta `Eggs` con **14** `EggPlaced` (7 a la izquierda y 7 a la derecha) con `EggAttachment` y attribute `EggSlot` 1–14.
- Carpeta `Sprinklers` por jardín (contenedor de los sprinklers colocados).
- Cartel `GardenLevel` con SurfaceGui y un Attachment `ProximityPrompt`.
- Sin `Parcels` (Garden_001 bajó de ~20.600 a **1.898** descendientes).
- El usuario borró `Garden_002`–`Garden_008` y planeaba **clonar** `Garden_001` (con IDs únicos).

**Sistemas**
- Plantado libre por XZ con `slotId` UUID y posición **local al `GardenFloor`** (las plantas aparecen en el mismo lugar relativo en cualquier jardín).
- Ciclo completo de plantas y frutas (single y multi-harvest) con timers en un BillboardGui responsive.
- Tools adaptados: regaderas por área, sprinklers como zona, pala, trowel, extractor y Plant item; amuletos adaptados pero sin implementar por el usuario.
- Huevos en soportes con animación de vibración, desarme con física y sonidos.
- Habilidades de pets adaptadas (Sweet Frog/Dog y Nebula, Lucky Block); Lunaris compatibles.
- Garden Level 1–100 + Resonancia (`GardenProgressService` / `GardenProgressConfig`).
- `GardenMigration` como ModuleScript (en el DEV quedó con `ENABLED = true`; en el principal se crea en `false` hasta la Sección 13).
- Limpieza completa de sistemas de parcelas.
- `GamepadController` **sin adaptar** (fuera de alcance).

**Pendientes que quedaron abiertos al perderse el trabajo**
- Probar en juego: extractor + Plant item, trowel, ciclo completo de huevos, pets.
- Probar que las plantas aparezcan en la misma posición relativa al tocar **otra base**.
- Pasada en móvil con el emulador.
- Probar la migración con datos reales (nunca se pudo en el DEV).
- Balance de XP del Garden Level (muy lento; ver `02c`).
- Revisar que los tiempos de crecimiento sean los de producción (en el DEV había valores de prueba; el principal debería tener los reales: **no tocarlos**).

## 4. Tabla a rellenar en la Sección 0 (auditoría de solo lectura del juego principal)
| Elemento | Estado (Pre-rework / Parcial / Ya hecho / No determinable) | Evidencia |
|---|---|---|
| Place abierto = `85407374189603` | | |
| Copia de respaldo `.rbxl` + versión en el historial | | |
| Copia DEV recuperable (sí/no) | | |
| `GardenFloor` | | |
| `Parcels` / `BaseSoil` | | |
| Sistemas de parcelas (lista del §2) | | |
| Water con scripts / `WaterAnimator` | | |
| `Eggs` / `EggPlaced` | | |
| `GardenLevel` | | |
| `Sprinklers` (carpeta) | | |
| Formato de datos de plantas (`plotSlots`, clave) | | |
| RemoteEvents duplicados en `ReplicatedStorage` | | |
| Tiempos de crecimiento e incubación actuales | | |

## 5. Búsquedas sugeridas (todos los scripts)
`BaseSoil`, `PlantPivot`, `InteractionPivot`, `Parcels`, `plotId`, `tonumber(`, `PlotService`, `PlotRegistry`, `PlotPresence`, `ParcelGameplay`, `ParcelGui`, `ParcelUnlock`, `ParcelDetector`, `HighlightBridge`, `canAccessPlot`, `getActivePlot`, `_PlotId`, `HasPlant`, `activeSoil`, `getPlotPos`, `teleportToSoilAndAct`, `WalkSpeed = 16`, `confirm_replace`, `plantReplace`.
