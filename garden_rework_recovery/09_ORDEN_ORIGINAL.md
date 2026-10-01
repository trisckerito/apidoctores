# 09 — ORDEN ORIGINAL (texto histórico, copiado tal cual)

> ⚠ **Nota para la nueva ventana:** esta es la orden que se le dio al agente anterior. Se conserva **como referencia**.
> **Prevalecen** la "NOTA DE CAMBIOS" de `08_PROMPT_DE_ARRANQUE.md` y la guía `00`–`07`:
> - Ya **NO** se trabaja en una copia DEV: se implementa en el **juego principal** (ver `00_MASTER_PLAN.md` §1).
> - La **consola** (`GamepadController`) quedó **fuera de alcance**.
> - Las reglas finales de la **migración** difieren de las de esta orden (ver `02_SYSTEMS.md` SYS-10). Además, la migración final es un ModuleScript llamado desde `PlantGrowthSystem.Init`, **no** un parche en `PlayerAdded` (ver `05` B2).
> - Durante el desarrollo se añadió **Garden Level + Resonancia** (`02` SYS-09), que no aparece aquí.

---

Garden Odyssey — Rework: 160 Parcelas → Jardín de Libre Colocación

Carga los skills gosa y go-worker-contract antes de comenzar.

## Contexto del proyecto

Garden Odyssey es un juego Roblox (placeId: 85407374189603, Studio ID: cd0b5411-5667-4d94-b775-19170f9f529d). Cada jugador tiene un jardín con actualmente 160 parcelas físicas (Model con BaseSoil + PlantPivot + InteractionPivot en la carpeta Parcels de cada jardín en workspace.Gardens). El objetivo es reemplazarlas por una sola superficie grande (GardenFloor) donde el jugador coloca plantas libremente en cualquier posición XZ, sin slots predefinidos.

## IMPORTANTE — Versión de prueba

Este rework se implementa en una copia DEV del juego, NO en el original. El juego original sigue en producción sin tocar. Al terminar todo el rework y probarlo, los cambios se portarán manualmente al original. Por eso al implementar no hay que preocuparse por no romper el juego en vivo — trabajamos con total libertad en el DEV.

## Sistemas actuales que debes investigar a fondo ANTES de tocar nada

### Cliente
- StarterPlayer.StarterPlayerScripts.ToolSystem.ToolUseController — maneja PC y Mobile. Para semillas: detecta BaseSoil bajo el mouse/touch con getSoilUnderMouse y getSoilFromPlantModel. Para tools: handlePlotToolClick. Para Plant items: getSoilUnderMouse. Usa _GardenId y _PlotId attributes del BaseSoil.
- StarterPlayer.StarterPlayerScripts.GamepadController — consola Xbox/PS. Tiene un sistema completo de targeting de parcelas con joystick derecho: collectSoils() recopila todos los BaseSoil del jardín del jugador, soilInDirection() navega entre ellos, setSelectedSoil() muestra SelectionBox. ButtonA confirma la acción. Re-colecta soils cada 5 segundos. Todo esto desaparece y debe rediseñarse para libre colocación.
- StarterPlayer.StarterPlayerScripts.ToolSystem.TrowelController — mover plantas entre parcelas (drag & drop). Leer completo.
- StarterPlayer.StarterPlayerScripts.ToolSystem.PlantVFXController — VFX de plantado, lee BaseSoil.

### Servidor — Plantado
- ServerScriptService.SeedPlacementSystem.PlantHandler — recibe PlantRequest con (action, clientGardenId, clientPlotId). Inyecta activePlot en ParcelInteractionService para que SeedService lo resuelva. Valida proximidad al BaseSoil (max 20 studs). Maneja tanto semillas (SeedService.plant()) como Plant items (PlantFactory.fromExisting()). Mantiene atributo HasPlant en cada BaseSoil.
- ServerScriptService.SeedSystem.SeedService — orquestador del plantado. Usa _getParcelInteraction():getActivePlot(player) para saber en qué plot plantar. Tiene locks de plot y UUID para evitar duplicados. Consume semilla con consumeReferenceGetEntry, crea planta con PlantFactory.
- ServerScriptService.PlantGrowthSystem.PlantFactory — crea PlantInstance. Leer completo.
- ServerScriptService.PlantGrowthSystem.PlantGrowthService — dueño del estado de todas las plantas. Clave: gardenId:plotId. Métodos: createPlant, plotHasPlant, onPlantCreated, onPlantRemoved.

### Servidor — Detección de presencia (DESAPARECE)
- ServerScriptService.PlotPresenceSystem.PlotPresenceService — polling raycast cada 0.12s desde la cabeza del jugador hacia BaseSoil. Usa GardenZone Touched/TouchEnded. Mayor consumidor de CPU del jardín.
- ServerScriptService.ParcelInteractionSystem.ParcelInteractionService — rastrea qué plot tiene activo cada jugador. SeedService lo consulta para saber dónde plantar.
- ServerScriptService.AbilitySystem.ParcelDetector — detecta plots en radio para sprinklers. Busca BaseSoil vecinos en grid.

### Servidor — Tools
- ServerScriptService.ToolSystem.ToolActionImplementations — PlacePlant, DeletePlant, ExtractPlant, MovePlant, PlaceSprinkler. Todos reciben plot = { gardenId, plotId }.
- ServerScriptService.ToolUseHandler — recibe ToolUseRequest con (clientGardenId, clientPlotId). Inyecta activePlot en ParcelInteractionService. Resuelve UUID del item equipado.

### Servidor — Plot/Garden
- ServerScriptService.PlotSystem.PlotService + PlotRegistry — descubren parcelas desde los modelos en workspace.Gardens. PlotRegistry busca PlantPivot y InteractionPivot en cada BaseSoil.
- ServerScriptService.GardenSystem.GardenService — asigna jardines a jugadores. Leer getPlayerGarden, getGardenPlayer.
- ServerScriptService.AccessSystem.AccessService — canAccessGarden, canAccessPlot.

### Datos
- PlayerDataService — leer el formato exacto de cómo se guardan las plantas. Namespace "inventory", "plotSlots" o similar. Las plantas están indexadas por plotId numérico 1-160.

## Lo que desaparece
- Carpeta Parcels con 160 modelos por jardín
- BaseSoil, PlantPivot, InteractionPivot por parcela
- PlotPresenceService — polling raycast
- ParcelInteractionService — active plot tracking
- ParcelDetector — grid-based radius
- PlotService / PlotRegistry
- Botón PLOT en cliente
- plotId: number como clave de datos
- _GardenId / _PlotId attributes en BaseSoil
- HasPlant attribute en BaseSoil

## Lo que lo reemplaza

GardenFloor — un solo Part plano, grande, anclado, CanCollide = true, con attribute GardenId.

slotId: string — UUID generado al plantar. Es la nueva clave de posición. Cada planta lleva { slotId, x, z, definitionId, ... }.

Plantar (PC/Mobile): click/tap en GardenFloor → raycast → obtiene posición XZ → valida radio mínimo entre plantas (5 studs) y límite máximo (200 plantas) → genera slotId → crea planta en esa posición. El cliente manda (gardenId, x, z) en vez de (gardenId, plotId).

Usar tools (PC/Mobile): click/tap en el modelo 3D de la planta en el mundo → cliente detecta el slotId (attribute en el modelo) → manda (gardenId, slotId).

Consola: joystick derecho navega entre plantas activas en vez de entre BaseSoil. ButtonA selecciona. El highlight pasa al modelo de la planta.

Sprinkler: radio circular real en XZ (distancia euclidiana sobre posiciones (x, z) de plantas activas).

GUI de planta: al clickear/tocar una planta → muestra GUI con detalles de esa planta específica (no de la parcela).

## Migración de datos — CRÍTICA, va junto con el update

Parche que corre en PlayerAdded, permanente, con migrated = true en el perfil:
- Planta extractable (madura o en crecimiento) → PlantFactory.fromExisting() → dar como Plant item al inventario con estado completo preservado
- Planta noExtract madura → cosechar fruta → dar al inventario → eliminar planta
- Planta noExtract en crecimiento → devolver semilla correspondiente → eliminar planta
- Al terminar: migrated = true → parche no vuelve a correr para ese jugador

## Instrucciones de implementación
1. INVESTIGAR PRIMERO — leer todos los sistemas mencionados antes de escribir una sola línea. Entender el flujo completo de plantado en PC, mobile y consola.
2. Implementar en este orden:
   - Investigación y documentación del sistema actual
   - Parche de migración de datos (sin tocar nada más)
   - Nuevo modelo de datos (slotId)
   - GardenFloor y sistema de posicionamiento libre
   - Reemplazar PlotPresence + ParcelInteraction por event-driven
   - Adaptar SeedService y PlantHandler al nuevo sistema
   - Adaptar tools servidor (ToolActionImplementations)
   - Adaptar cliente PC y Mobile (ToolUseController)
   - Adaptar consola (GamepadController)
   - Sprinkler con radio real (reemplazar ParcelDetector)
   - Eliminar sistemas y UI obsoletos
3. No eliminar sistemas viejos hasta que el nuevo esté probado y funcionando
4. Reportar al final de cada parte qué se hizo, qué falta y qué decisiones de diseño necesitan confirmación antes de continuar
