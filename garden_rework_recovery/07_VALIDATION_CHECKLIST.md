# 07 — CHECKLIST DE VALIDACIÓN

> Marcar cada punto al cerrar la sección. Si un punto falla → **no avanzar**, reportarlo.
> Checks comunes a **todas** las secciones (aplicar siempre):
> - [ ] Los scripts tocados compilan.
> - [ ] Output sin errores ni warnings nuevos al iniciar Play.
> - [ ] Sin referencias rotas (`FindFirstChild`/`WaitForChild` a objetos inexistentes).
> - [ ] Solo se tocó lo de esta sección.
> - [ ] No se reintrodujo ningún bug de `05`.
> - [ ] Recomendado (`gosa`): cerrar y reabrir Studio y hacer un Play limpio.

---

## S0 — Auditoría
- [ ] Place destino identificado y confirmado por el usuario.
- [ ] Tabla de `01` §4 rellenada con evidencia.
- [ ] Lista de UNKNOWN resueltos y abiertos entregada.
- [ ] **Ningún cambio en el proyecto.**

## S1 — `GardenFloor`
- [ ] `Garden_001.GardenFloor` existe y es un `Part`.
- [ ] `Anchored = true`, `CanCollide = true`, `CanQuery = true`.
- [ ] Attribute `GardenId` = `GardenID` del modelo.
- [ ] Cubre el área de plantado.

## S2 — Datos `slotId`
- [ ] Al plantar, se guarda `plants[slotId] = {definitionId, plantedAt, x, z, properties}`.
- [ ] `PlantGrowthService` indexa por `gardenId:slotId`.
- [ ] Rejoin: las plantas reaparecen en la misma X/Z con la misma `yRotation` y el mismo estado de crecimiento.
- [ ] `restorePlants` ignora entradas de formato viejo sin error.

## S3 — Plantado libre (servidor)
- [ ] Plantar fuera del `GardenFloor` → rechazado.
- [ ] Plantar a < 5 studs de otra planta → rechazado.
- [ ] Planta número 201 → rechazada.
- [ ] Plantar en el jardín de otro jugador sin acceso → rechazado.
- [ ] Plantar lejos del jugador (proximidad) → rechazado.
- [ ] Doble request simultáneo → una sola planta y una sola semilla consumida.
- [ ] Modelo `Plant_<gardenId>:<slotId>` con attribute `slotId`, apoyado sobre la cara superior de `GardenFloor` (sin hundirse ni flotar).
- [ ] Plantar un `Plant` item conserva su estado.
- [ ] Ningún uso de `getActivePlot` en el flujo.

## S4 — Cliente PC/Mobile (plantado)
- [ ] PC: click en el suelo → la planta aparece en el punto clickeado.
- [ ] Mobile (emulador): tap → igual.
- [ ] El cliente envía `(gardenId, x, z)`. No lee `_PlotId`.
- [ ] El raycast ignora el personaje y solo acepta `GardenFloor`.

## S5 — Tools, Trowel y VFX
- [ ] Click sobre una planta con un tool → el servidor recibe el `slotId` correcto.
- [ ] `DeletePlant`, `ExtractPlant`, `WaterPlant` y `MovePlant` funcionan por `slotId`.
- [ ] Trowel: mover a un punto libre válido → OK. A < 5 studs → rechazado.
- [ ] VFX de plantado sin errores (no lee `activeSoil`).
- [ ] Tool sobre una planta de otro jardín sin acceso → rechazado.

## S6 — Sprinkler
- [ ] Se coloca en un XZ libre.
- [ ] Afecta exactamente a las plantas dentro del radio circular de su tipo.
- [ ] Radios provenientes de config o definiciones (no codificados a mano en la lógica).
- [ ] `ParcelDetector` no se usa.
- [ ] Sonido sin errores.

## S7 — Huevos
- [ ] Los huevos aparecen en los soportes de `Eggs`.
- [ ] La incubación se conserva tras reubicarlos o hacer rejoin.
- [ ] Sweet Egg = valor confirmado por el usuario.

## S8 — Garden Level + Resonancia
- [ ] `GardenProgressConfig` existe. MaxLevel = **100**. Curva `6N² + 44`. Coste 1.000.000. Bonus 0,05 %. Tope 50 %.
- [ ] Cosechar una fruta Common → +1 XP. Una OdysseySecret → +8 XP (según la tabla de rarezas existente).
- [ ] El cartel muestra el título, "Lv. X", la barra y "a / b EXP", centrados. Se actualiza al cosechar, sin loop.
- [ ] Un visitante ve el mismo valor.
- [ ] En el nivel 100: barra dorada y "Lv. 100 MAX". Prompt "Resonate (1,000,000)" visible **solo para el dueño**, HoldDuration 1 s.
- [ ] Sin dinero suficiente → aviso y no cobra.
- [ ] Con dinero suficiente → pide confirmación → cobra → nivel 1, 0 XP → título "Garden Level 1".
- [ ] Una fruta cosechada con resonancia R se vende con +0,05 % × R (tope 50 %). El inventario muestra el precio con bonus.
- [ ] Persistencia: rejoin conserva nivel, XP y resonancia.

## S9 — Migración
- [ ] `GardenMigration` es un ModuleScript con `ENABLED = true`.
- [ ] Se llama desde `PlantGrowthSystem.Init` **antes** de `restorePlants`. No es un Script en `PlayerAdded`.
- [ ] Planta vieja madura y extraíble → `Plant` item con su tamaño, variante y mejoras.
- [ ] Planta vieja en crecimiento o no extraíble → semilla correcta (no se pierde: B1).
- [ ] Frutas maduras → inventario.
- [ ] Inventario lleno → **no** marca `gardenReworkMigrated` y no duplica en el reintento.
- [ ] Tras completarse: `gardenReworkMigrated = true` y en el siguiente join no vuelve a correr.
- [ ] Los huevos no se ven afectados.

## S10 — Limpieza
- [ ] No existen `Parcels`, `BaseSoil`, `PlantPivot` ni `InteractionPivot`.
- [ ] No existen `PlotSystem`, `HighlightBridge`, `ParcelDetector` ni `canAccessPlot`.
- [ ] `PlotPresenceService` y `ParcelInteractionService` eliminados (o con una decisión documentada).
- [ ] Búsqueda de `01` §3.3 → 0 referencias activas (excepto `plotId` dentro de `GardenMigration`).
- [ ] Tutorial: el beam apunta a `GardenFloor`.
- [ ] Sweet Frog y sonido de sprinklers sin errores.
- [ ] `GamepadController` intacto (inerte, sin errores).
- [ ] Todos los scripts compilan. `Garden_001` ≈ 1.898 instancias.

## S11 — Multi-jardín
- [ ] Cada `Garden_00k` tiene un nombre único, `GardenID = k` y `GardenFloor.GardenId = k`.
- [ ] Cada uno contiene `GardenFloor`, `Eggs`, `Sprinklers` y `GardenLevel`.
- [ ] Max Players = número de jardines.
- [ ] Con 2+ jugadores: cada uno recibe su jardín y planta solo en el suyo.
- [ ] Valores de prueba revisados (MaxLevel, tiempos de crecimiento, Sweet Egg).

## S12 — Prueba con datos reales
- [ ] El usuario confirmó los riesgos por escrito.
- [ ] API Services activado. Sin sesión abierta en el juego publicado.
- [ ] Las plantas viejas llegan al inventario según las reglas.
- [ ] Flag `gardenReworkMigrated` presente.
