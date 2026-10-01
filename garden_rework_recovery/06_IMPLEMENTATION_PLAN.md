# 06 — RUTA DE IMPLEMENTACIÓN (GUÍA DE PROCESO)

> Destino: **juego principal**. Control total con las paradas de `00` §2. **Guardar al cerrar cada fase** (el trabajo anterior se perdió por un cierre forzado de Studio).
> Cada fase: leer la spec → auditar → implementar la versión final → releer las ediciones → compilar → validar con `07` → reportar.

| # | Fase | Spec | Parada manual |
|---|---|---|---|
| 0 | Arranque, respaldo y auditoría | `01`, `03` §0 | **Sí** |
| 1 | `GardenFloor` + `BaseSoil.CanQuery = false` | `02` SYS-01 | Después: el usuario estiliza el suelo |
| 2 | Plantado libre + datos + coordenadas locales + visual | `02` SYS-02/03/04 | No |
| 3 | Frutas, cosecha y timers | `02` SYS-05/06 | No |
| 4 | Borrar los sistemas de parcelas en segundo plano | `02c` CLEAN-01 (parte 1) | Confirmar (destructivo) |
| 5 | Optimizaciones globales | `04` | Confirmar antes de aplicar en masa |
| 6 | Tools | `02b` | No |
| 7 | Huevos en soportes | `02c` EGG-01 | **Sí** (`Eggs`/`EggPlaced`, assets) |
| 8 | Pets | `02c` PET-01..03 | No |
| 9 | Garden Level + Resonancia | `02c` LVL-01 | **Sí** (`GardenLevel`) |
| 10 | `GardenMigration` (con `ENABLED = false`) | `02c` MIG-01 | Confirmar las reglas |
| 11 | Limpieza final (parcelas, `PlotSystem`…) | `02c` CLEAN-01 | Confirmar (destructivo) |
| 12 | Clonado de jardines, IDs y valores | `03` §2 | **Sí** |
| 13 | Activar la migración, probar con datos reales y publicar | `02c` MIG-01, `05` H4 | **Sí** |
| — | Consola | Fuera de alcance | — |

## Fase 0 — Arranque
1. Cargar `gosa` y `go-worker-contract`. Confirmar con el MCP que el Studio abierto es `85407374189603`.
2. **PARADA:** checklist de `03` §0 + respaldo (`.rbxl` + versión) + ¿existe todavía el DEV? + ¿API Services desactivado durante el desarrollo?
3. Auditoría de solo lectura (`01` §4 y §5). Leer: `PlayerDataService` (formato), `PlantHandler`, `SeedService`, `PlantGrowthService`, `PlantVisualService`, `HarvestService`, `ToolUseHandler`, `ToolActionImplementations`, `ToolUseController`, `TrowelController`, los VFX controllers, el sistema de huevos, `PetSystem` y las definiciones (rarezas, `growthTime`/`generationTime`, `noExtract`, relación planta → semilla, curva de XP de pets).
4. Reportar y continuar.

## Fase 1 — GardenFloor
Crear un `GardenFloor` en **cada** jardín por código (para que cualquier jardín asignado funcione durante las pruebas): CFrame y tamaño del `Water`, cara superior +0,05 sobre el tope de los `BaseSoil`, propiedades de `03` §1.1 y `GardenId`. `BaseSoil.CanQuery = false`. **Avisar al usuario** que puede estilizar el de Garden_001 cuando quiera (después se copian color/material/transparencia, o se clona el jardín al final).

## Fase 2 — Plantado
Implementar SYS-02/03/04 completos de una vez: `getFloorHit`, `PlantRequest (x, z)`, validaciones (1 stud, 200 plantas, 30 studs, límites del suelo), `slotId`, `localX/Z`, `_activeSlot`, `SeedService`, `PlantGrowthService` (sin `tonumber`, restore + auto-reparación + protección del formato viejo), `PlantVisualService` (Y del suelo, attributes, `CanQuery = true`), VFX de tierra (opción C), sin confirmación por rareza y sin `plantReplace`. Validar con `07` §F2 (incluido el cambio de jardín).

## Fase 3 — Frutas, cosecha y timers
SYS-05/06: single-harvest con runtime al plantar, madurez autoritativa del servidor, sin `tonumber`, handlers con UUID, listener de efectos con espera, BillboardGui del timer (frutas y plantas, el más cercano, responsive). Quitar las llamadas a `ParcelGameplayService` de `HarvestService` (dejar el gancho para `GardenProgressService`).

## Fase 4 — Borrar los sistemas de parcelas en segundo plano (confirmar)
Borrar `PlotPresenceSystem`, `ParcelGameplaySystem`, `ParcelGuiBridge`, `ParcelUnlockHandler`, `ParcelGuiController` y las GUIs/botón de parcela. Desactivar `HighlightSystem/Init`. Corregir de inmediato las referencias (`GardenService`, `PlantGrowthService._resolveServices`, `ParcelInteractionService`). **No** tocar todavía `PlotSystem` ni las parcelas (los huevos aún las usan hasta la Fase 7). Validar que las plantas se restauran.

## Fase 5 — Optimizaciones (confirmar)
Auditar los raycasts → aplicar `04` P3–P10 (CanQuery/CastShadow, WeatherParticles, EffectSystems, `WaterAnimator`, `GiroScript`, `StarCubeAnimator`, `RagdollTester`, `CanBeDropped`). Respetar P-MARK. Validar que se puede clickear todo lo interactivo.

## Fase 6 — Tools
`02b` completo: detección del modelo, cooldowns, regaderas por área, sprinklers como zona (sin combo secreto ni confirmación), pala, trowel, extractor, Plant item, amuletos (solo detección). Revisar los remotes duplicados (`WaterComplete`). Borrar `DeletePlantHandler`.

## Fase 7 — Huevos
**PARADA:** `Eggs/EggPlaced ×14` con `EggAttachment` y el Attachment del prompt en los assets. Después, todo `02c` EGG-01 (numeración `EggSlot`, módulo `EggSlots`, colocación, límite de 8, aviso, reubicación de huevos viejos, x4, prompt, hatch con validaciones y cooldown, `EggAnimator` con sonidos). **No** cambiar los tiempos de incubación del principal.

## Fase 8 — Pets
PET-01..03 + los remotes desde el arranque.

## Fase 9 — Garden Level
**PARADA:** `GardenLevel`. Implementar LVL-01. `MaxLevel = 100`. Preguntar por el balance (multiplicador de XP).

## Fase 10 — Migración
MIG-01 con **`ENABLED = false`**. Mientras esté desactivada, `restorePlants` ignora (sin borrar) las plantas viejas. Revisar el código contra una entrada real (solo lectura) y confirmar las reglas con el usuario.

## Fase 11 — Limpieza final (confirmar)
Borrar `Parcels` (todas), `PlotSystem`, `HighlightBridge`, `ParcelDetector` y `canAccessPlot`. Ajustar el tutorial (beam → `GardenFloor`), el sonido de los sprinklers y la Sweet Frog. Buscar referencias (`01` §5) → 0. Compilar **todos** los scripts. No tocar `GamepadController`.

## Fase 12 — Jardines
**PARADA:** el usuario clona y reposiciona. Verificar IDs, `EggSlot`, `Sprinklers`, `GardenLevel`, que no haya `Parcels` y Max Players. Revisar los valores de prueba (MaxLevel, incubaciones, crecimiento).

## Fase 13 — Migración real y publicación (confirmar)
Explicar los riesgos (`05` H4) → API Services ON, salir del juego publicado → `ENABLED = true` → Play con la cuenta del usuario → verificar el inventario y el flag → **publicar solo con autorización**.
