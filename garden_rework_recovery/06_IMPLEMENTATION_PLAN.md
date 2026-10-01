# 06 — PLAN DE IMPLEMENTACIÓN (DOCUMENTO PRINCIPAL)

> **Una sección a la vez.** Al terminar cada una: verificar con `07`, reportar con el formato de `go-worker-contract` y **esperar confirmación**.
> Protocolo de cada sección: `00_MASTER_PLAN.md` §2.3.
> **Si la auditoría (Sección 0) muestra que una sección ya está en su versión final → solo verificarla y reportarla. No reescribir.**

## Orden (derivado de las dependencias)

| # | Sección | Depende de | Parada manual |
|---|---|---|---|
| 0 | Arranque y auditoría (solo lectura) | — | Sí (decidir el place destino) |
| 1 | Mundo base: `GardenFloor` | 0 | **Sí** |
| 2 | Modelo de datos `slotId` | 1 | No |
| 3 | Plantado libre (servidor) | 2 | No |
| 4 | Plantado libre (cliente PC/Mobile) | 3 | No |
| 5 | Tools servidor + cliente (`slotId`), Trowel, VFX | 3, 4 | No |
| 6 | Sprinkler con radio real | 5 | Posible (carpeta `Sprinklers`, radios) |
| 7 | Huevos en soportes | 2 | **Sí** (carpeta `Eggs`) |
| 8 | Garden Level + Resonancia | 2 (cosecha) | **Sí** (`GardenLevel`) |
| 9 | Migración (`GardenMigration`) | 2, 3 | No (pero confirmar las reglas) |
| 10 | Limpieza de sistemas viejos y residuos | 1–9 validadas | Confirmar (destructivo) |
| 11 | Multi-jardín y configuración del place | 10 | **Sí** (clonado manual) |
| 12 | Prueba de migración con datos reales | 9, 11 | **Sí** (riesgo irreversible) |

Fuera de alcance: **consola / `GamepadController`** (ver `02` SYS-08).

---

## SECCIÓN 0 — Arranque y auditoría (SOLO LECTURA)

1. Cargar los skills `gosa` y `go-worker-contract`.
2. Listar los Studios abiertos (MCP: *List Roblox Studios* / *Get Studio State*). Identificar el place: original `85407374189603` o DEV `86748110736040`.
3. **Preguntar al usuario:** ¿en qué place se implementa esta recuperación?
   - DEV existente (`86748110736040`), si sigue vivo.
   - Una nueva copia DEV del original.
   - Directamente en el original (desaconsejado: la orden exigía trabajar en una copia).
4. Recorrer el place y rellenar la tabla de `01_PROJECT_STATE.md` §4, con las búsquedas de §3.3.
5. Para cada `UNKNOWN` de `02`, intentar resolverlo leyendo el proyecto. Prioridades:
   - firma actual de `PlantRequest` y `ToolUseRequest`,
   - nombres de campos en `PlayerDataService`,
   - relación planta → semilla,
   - config de sprinklers (radios),
   - estructura de `Eggs`, `Sprinklers` y `GardenLevel`,
   - existencia de `ParcelGameplayService`, `PlotPresenceService`, `ParcelInteractionService` y el botón PLOT,
   - GUI de detalle de planta.
6. **Reportar:** la tabla rellenada, los UNKNOWN resueltos y los que siguen abiertos, y qué secciones parecen ya hechas.
7. **DETENERSE.** No modificar nada.

**Salida esperada:** lista de secciones a implementar, verificar o saltar.

---

## SECCIÓN 1 — Mundo base: `GardenFloor`

1. Comprobar que existen `workspace.Gardens.Garden_001` y su `GardenFloor` (`03` §3.1).
2. **PARADA MANUAL** (plantilla `00` §4) si falta `GardenFloor` o su attribute `GardenId`, o si `Garden_001` no tiene `GardenID`.
3. Si el usuario dice SÍ → verificar: tipo `Part`, `Anchored`, `CanCollide`, `CanQuery`, `GardenId` = `GardenID` del modelo.
4. **No borrar todavía la carpeta `Parcels`** si existe (eso es la Sección 10).
5. Verificar con `07` §S1 → reportar → detenerse.

---

## SECCIÓN 2 — Modelo de datos `slotId`

1. Leer `PlayerDataService`, `PlantGrowthService` (incluido `restorePlants`) y `PlantFactory`.
2. Implementar el esquema final (`02` SYS-01): `plants[slotId] = { definitionId, plantedAt, x, z, properties }`.
3. Cambiar la clave interna de `PlantGrowthService` a `gardenId:slotId`. Adaptar `createPlant`, `onPlantCreated`, `onPlantRemoved` y la consulta de existencia.
4. Dejar `restorePlants` con la **protección** contra entradas de formato viejo (`02` SYS-10, `05` B2). Por ahora solo omitirlas; la migración llega en la Sección 9.
5. **No** tocar todavía el cliente ni los tools.
6. Verificar con `07` §S2 → reportar → detenerse.

---

## SECCIÓN 3 — Plantado libre (servidor)

1. Leer `PlantHandler`, `SeedService`, `PlantVisualService`, `AccessService` y `PlantingGuard`.
2. Implementar el flujo de `02` SYS-02:
   - `PlantRequest` recibe `(gardenId, x, z)`. Comprobar antes la firma real (Sección 0).
   - Validaciones: `canAccessGarden`, dentro de `GardenFloor`, proximidad, radio mínimo de 5, máximo de 200 (P4: sobre datos).
   - `slotId` generado en el servidor. Locks `userId:gardenId:slotId`.
   - `PlantVisualService`: modelo `Plant_<gardenId>:<slotId>`, attribute `slotId`, Y = cara superior de `GardenFloor` (`05` B3).
3. Mantener `Plant` items vía `PlantFactory.fromExisting()`.
4. **No** eliminar todavía `PlotPresenceService`, `ParcelInteractionService` ni `PlotSystem`: solo dejar de depender de ellos en este flujo.
5. Si 5 studs, 200 plantas o la distancia de proximidad no están en una config → **preguntar** si se crean como config (data-driven, `gosa`).
6. Verificar con `07` §S3 → reportar → detenerse.

---

## SECCIÓN 4 — Plantado libre (cliente PC/Mobile)

1. Leer `ToolUseController` completo.
2. Semillas y `Plant` items: raycast filtrado a `GardenFloor` (`04` P6) → enviar `(gardenId, x, z)`.
3. Quitar el uso de `_GardenId`, `_PlotId`, `getSoilUnderMouse` y `getSoilFromPlantModel` en ese flujo.
4. Botón PLOT: si sigue existiendo → **preguntar** si se elimina ahora o en la Sección 10.
5. Verificar con `07` §S4 (prueba en Play: plantar en PC y en el emulador móvil) → reportar → detenerse.

---

## SECCIÓN 5 — Tools (servidor + cliente), Trowel y VFX

1. Servidor: `ToolUseHandler` → `(gardenId, slotId)`. `ToolActionImplementations` → `{gardenId, slotId}` o `{gardenId, x, z}` según la acción (`02` SYS-04).
2. Cliente: `ToolUseController` → raycast al modelo `Plant_*` → attribute `slotId` (sustituye a `handlePlotToolClick`).
3. `TrowelController`: recoger la planta → click en un punto libre del `GardenFloor` → servidor (`MovePlant`) valida el radio mínimo. Si la Sección 0 muestra otra implementación final → respetarla.
4. `PlantVFXController`: dejar de leer `EquipState.activeSoil`; usar la planta o posición objetivo.
5. GUI de detalle de planta: si no existe → **preguntar** al usuario si forma parte de esta recuperación (no consta como implementada).
6. Verificar con `07` §S5 → reportar → detenerse.

---

## SECCIÓN 6 — Sprinkler con radio real

1. Leer las definiciones o la config de sprinklers y la acción `PlaceSprinkler`.
2. Si no existen radios por tipo → **PARADA (gameplay)**: preguntar los radios de basic, rare, ultrarare y cosmic.
3. Comprobar la carpeta `Sprinklers` del jardín. Si falta → **PARADA MANUAL** (o preguntar si la crea el código).
4. Implementar el radio euclidiano XZ sobre los datos de plantas (`04` P3). Colocación en un XZ libre.
5. Mantener el sonido de sprinklers sin referencias a parcelas.
6. Verificar con `07` §S6 → reportar → detenerse.

---

## SECCIÓN 7 — Huevos en soportes

1. Auditar el sistema de huevos, `EggPlantRequest` y la carpeta `Eggs`.
2. Si el sistema de soportes ya existe y funciona → solo verificar.
3. Si no existe → **DETENERSE Y PREGUNTAR**: el historial no describe su diseño (número de soportes, ubicación, flujo). No diseñarlo por cuenta propia.
4. Requisito final conocido: los huevos se reubican en soportes **conservando la incubación**. Sweet Egg = 5 h (confirmar).
5. Verificar con `07` §S7 → reportar → detenerse.

---

## SECCIÓN 8 — Garden Level + Resonancia

1. **PARADA MANUAL:** el modelo `GardenLevel` con `SurfaceGui` y el prompt (`03` §3.4). Resolver la ambigüedad del `Attachment` llamado "ProximityPrompt".
2. Leer:
   - la tabla de rarezas (números 1..8: Common = 1 … OdysseySecret = 8),
   - la curva de XP de pets (`6 × N² + 44`),
   - el flujo de cosecha,
   - el cálculo del precio de venta (PriceCalculator o equivalente),
   - la moneda Odyssey Coins.
3. Crear o verificar `GardenProgressConfig` con los valores de `02` SYS-09. **MaxLevel = 100.**
4. Servidor:
   - XP por cosecha al jardín del dueño,
   - subida de nivel,
   - estado persistente (nivel, XP, resonancia),
   - prompt habilitado solo en nivel 100 y solo para el dueño,
   - confirmación → cobro de 1.000.000 → nivel 1 con 0 XP y resonancia +1. Si no hay dinero suficiente, avisar.
5. Fruta: guardar el nivel de resonancia al cosechar. Precio de venta × (1 + min(0,05 % × res, 50 %)). Según `gosa`: inyectar el bonus **solo para la operación**, nunca como registro global.
6. Cartel: título "Garden Level" / "Garden Level N", "Lv. X", barra, "a / b EXP", "Lv. 100 MAX" en dorado. Actualización por evento (`04` P7). Visible para los visitantes.
7. **No cambiar el balance.** Recordar al usuario B11 y preguntar por el multiplicador.
8. Verificar con `07` §S8 → reportar → detenerse.

---

## SECCIÓN 9 — Migración (`GardenMigration`)

1. **Confirmar con el usuario** las reglas finales (`02` SYS-10), incluidas las diferencias con la orden original y el caso `noExtract` maduro.
2. Leer una entrada real de formato viejo (o crear una de prueba en el DEV) y la relación planta → semilla (`05` B1).
3. Implementar el **ModuleScript** `GardenMigration` (`ENABLED` como constante), llamado desde `PlantGrowthSystem.Init` **antes** de `restorePlants` (`05` B2).
4. Reglas:
   - madura y extraíble → `Plant` item,
   - en crecimiento o no extraíble → semilla,
   - frutas maduras → frutas,
   - inventario lleno → no marcar y reintentar (`05` B4: resolver la entrega parcial),
   - huevos → no tocarlos.
5. Flag `gardenReworkMigrated = true` al completar.
6. Probar en el DEV con una planta vieja falsa (`05` B13).
7. Verificar con `07` §S9 → reportar → detenerse.

---

## SECCIÓN 10 — Limpieza (DESTRUCTIVA → confirmar antes)

1. Buscar referencias (`01` §3.3) en **todos** los scripts. Corregir los residuos: tutorial → `GardenFloor`, sonido de sprinklers, Sweet Frog y cualquier otro encontrado.
2. **Pedir confirmación** y entonces eliminar:
   - carpeta `Parcels` (160 modelos) de cada jardín,
   - `PlotSystem` (`PlotService`, `PlotRegistry`),
   - `HighlightBridge`,
   - `ParcelDetector`,
   - `AccessService.canAccessPlot`,
   - `PlotPresenceService`, `ParcelInteractionService`,
   - botón PLOT,
   - attributes `_GardenId`, `_PlotId` y `HasPlant`.
3. `ParcelGameplayService`: **preguntar** (D7, sin respuesta registrada).
4. **No tocar `GamepadController`.**
5. Verificar: todos los scripts compilan (referencia: 523 en el DEV original), 0 referencias activas a lo eliminado y `Garden_001` ≈ 1.898 instancias.
6. Reportar → detenerse.

---

## SECCIÓN 11 — Multi-jardín y configuración del place

1. **PARADA MANUAL:** el usuario clona `Garden_001` a N jardines (`03` §4) y ajusta Max Players.
2. Verificar nombres, `GardenID` y `GardenId` únicos y coincidentes, presencia de `GardenFloor`, `Eggs`, `Sprinklers` y `GardenLevel` en cada uno, y que no haya `Parcels`.
3. Revisar los valores de prueba (`05` B9): MaxLevel = 100, tiempos de crecimiento, Sweet Egg.
4. Reportar → detenerse.

---

## SECCIÓN 12 — Prueba de migración con datos reales (opcional; la decide el usuario)

1. Explicarle al usuario los riesgos de `05` B12 y obtener una confirmación explícita.
2. Preparación (manual): portar los cambios al Studio del original, activar "Enable Studio Access to API Services" y salir del juego publicado.
3. Play → comprobar que las plantas viejas llegan al inventario según las reglas y que `gardenReworkMigrated = true`.
4. Reportar. Indicar cómo repetir la prueba (borrar el flag a mano).

---

## Cuándo DETENERSE y preguntar (resumen)

- Falta un objeto de `03`.
- Un valor de gameplay no está en ninguna config (radios, distancias, balance).
- Hay ambigüedad entre la orden original y la versión final (migración, nivel 0 o 1, GUI de planta, botón PLOT, `ParcelGameplayService`).
- Antes de cualquier borrado o de activar la migración sobre datos reales.
- Si la auditoría encuentra una implementación distinta de la documentada → reportarla y preguntar cuál prevalece.
