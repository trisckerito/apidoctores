# 02 — SISTEMAS: NÚCLEO (suelo, plantado, datos, plantas, frutas, timers)

> Versión FINAL de cada sistema. Las rutas son las del proyecto. **Antes de crear algo, buscarlo**: puede existir.
> Tools → `02b`. Huevos, pets, Garden Level, migración y limpieza → `02c`.

---

## SYS-01 — GardenFloor
**Propósito:** superficie única por jardín donde se planta en cualquier punto XZ, y referencia de coordenadas locales.

**Cómo se creó (por código, en el DEV)** [CONFIRMADO]:
- Mismo **CFrame, posición y tamaño que el Part `Water`** del jardín (92×92 studs; hereda la rotación del jardín).
- `Anchored = true`, `CanCollide = true`, `CanQuery = true`, `CastShadow = false`, attribute `GardenId`.
- Altura: **cara superior 0,05 studs por encima del tope de los `BaseSoil`** (si queda por debajo, el click golpea el BaseSoil). La primera versión quedó 0,15 por debajo y no se podía clickear.
- Mientras existan las parcelas: **`CanQuery = false` en todos los `BaseSoil`** (siguen con `CanCollide = true`).

**Después, a mano (usuario) en Garden_001:** lo subió un poco, lo rotó unos grados, lo elevó y le cambió color/material/transparencia. El agente copió **color, material y transparencia** de Garden_001 a los demás. Ver `03`.

**Es la referencia de coordenadas:** se usa su `CFrame` (no el del Water, que es cosmético) para convertir world↔local.

---

## SYS-02 — Plantado libre (semillas y Plant items)

**Flujo final (PC y móvil):**
1. Cliente (`ToolSystem.ToolUseController`): `getFloorHit()` hace un raycast con `FilterType.Include` limitado a los `GardenFloor` → obtiene `{floor, position}`. Un toque corto planta; un arrastre mueve la cámara (no planta).
2. Envía `PlantRequest:FireServer("plant", gardenId, x, z)`.
3. Servidor (`SeedPlacementSystem.PlantHandler`):
   - acceso al jardín (`AccessService.canAccessGarden`);
   - punto dentro de los límites del `GardenFloor` del jardín;
   - proximidad jugador–punto ≤ **30 studs** (semillas y Plant items por igual);
   - **separación mínima de 1 stud** con las plantas del jardín (decisión del usuario; error `too_close`);
   - **máximo 200 plantas** por jardín (`garden_full`);
   - genera **`slotId` (UUID)** en el servidor;
   - calcula **`localX`/`localZ`** = `GardenFloor.CFrame:PointToObjectSpace(Vector3.new(x, 0, z))`;
   - inyecta `{gardenId, slotId, x, z, localX, localZ}` en `ParcelInteractionService._activeSlot[player]` (getter `getActiveSlot(player)`);
   - semilla → `SeedService.plant()`; Plant item → `PlantFactory.fromExisting()` (conserva tamaño, variante y mejoras).
4. Respuesta: `PlantResponse:FireClient(player, "planted", worldX, worldZ)`.
5. `ToolSystem.PlantVFXController`: **28 cubos de tierra** salen disparados desde `(worldX, Y del suelo, worldZ)`.

**Decisiones finales (UX):**
- **Sin teleport, sin animación del personaje y sin bloqueo de movimiento** (opción "C" elegida por el usuario: solo partículas en el punto plantado).
- **Sin confirmación por rareza** ("Plant X here?" para Galaxy, Royal, Aurora y OdysseySecret) — eliminada.
- **Sin reemplazo de planta:** se eliminaron la acción `confirm_replace` y la función `plantReplace` del servidor. El servidor solo acepta `"plant"`.
- Se eliminó el sync del attribute `HasPlant` en `BaseSoil`.

**`SeedService`** (`SeedSystem`): usa `getActiveSlot` (no `getActivePlot`); todo `plotId` → `slotId`; guarda `x`, `z`, `localX`, `localZ` en el `plantInstance`; locks `userId:gardenId:slotId`; consume con `consumeReferenceGetEntry`; deduplicación por UUID.

**`ParcelInteractionService`:** **se conserva** solo con `_activeSlot` / `getActiveSlot`. Se eliminaron `_activePlot` y su `require` de `PlotPresenceService`.

**`PlantingGuard.isPlanting()`:** se preserva.

---

## SYS-03 — Datos de plantas (`slotId`) y restauración
**Formato viejo** (solo lo lee la migración): `plotSlots[slotIdx].gardens["main"].plants[tostring(plotId)] = { definitionId, plantedAt, properties = plantInstance }`.

**Formato final** [CONFIRMADO]:
```lua
plotSlots[slotIdx].gardens["main"].plants[slotId] = {
    definitionId = "...",
    plantedAt    = os.time(),
    properties   = plantInstance, -- incluye uuid, growthTierId, yRotation, x, z, localX, localZ, ...
}
```
(`x/z/localX/localZ` viven en `properties`/`plantInstance`; leer el código real antes de asumir otra ubicación.)

**`PlantGrowthSystem.PlantGrowthService`:**
- Clave interna `gardenId:slotId` (string). **`_parseKey` NO hace `tonumber`** (devolvía `nil` con UUID y `getGardenPlants` devolvía 0 plantas, lo que rompía la regadera y los sprinklers).
- `getPlant`, `plotHasPlant`, `getGrowthRate`, `clearGarden`, `getGardenPlants`, `maturePlant` y `createPlant` adaptados a string.
- `_resolveServices`: **sin** referencias a `ParcelGameplaySystem`.
- Scheduler de maduración de planta: `TICK_INTERVAL = 5` s (las single-harvest **no** dependen de él; ver SYS-05).
- **`restorePlants`:**
  - si la planta tiene `localX/localZ` → `world = GardenFloor(jardín actual).CFrame:PointToWorldSpace(Vector3.new(localX, 0, localZ))`, y actualiza `x/z`;
  - **auto-reparación:** si no tiene `localX/localZ` pero sí `x/z`, busca en cuál de los jardines cae ese punto, calcula la posición local respecto de ese jardín y la guarda;
  - **protección:** ignora (sin borrar) las entradas de formato viejo (clave numérica de parcela) → las procesa `GardenMigration`.
- `GardenService.releasePlayer()` (en `PlayerRemoving`) → `clearGarden(gardenId)`: limpia el runtime, no la persistencia. **Sin cambios** (ya funcionaba).

**`GardenSystem.GardenService`:** quitar la referencia a `PlotPresenceSystem`. El attribute `GardenID` del modelo es **string** ("1") en el DEV y `_gardenToPlayer` se indexa con ese valor [CONFIRMADO]. Verificar el tipo en el principal y mantener la coherencia.

---

## SYS-04 — Visual de plantas (`PlantVisualSystem.PlantVisualService`)
- Posición: `pivotCF = CFrame.new(worldX, plantY, worldZ)`; `targetCF = pivotCF * CFrame.Angles(0, math.rad(yRotation), 0) * CFrame.new(-PlantAttachment.CFrame.Position)`; `model:PivotTo(targetCF)`. El **`PlantAttachment`** del modelo sigue siendo el ancla (igual que antes con `PlantPivot`).
- **`plantY` = cara superior del `GardenFloor`** del jardín (no la del `BaseSoil`; cuando el usuario subió el suelo, las plantas quedaban hundidas).
- `yRotation` aleatoria de `PlantFactory.create()` (sin cambios).
- Nombre del modelo: **`Plant_<gardenId>:<slotId>`**.
- Attributes en **todos** los modelos de planta (single **y** multi-harvest): **`GardenId`, `SlotId`, `DefinitionId`**. Faltaba `GardenId` en las multi-harvest, por eso la pala/trowel/extractor no las detectaban.
- **Partes de la planta con `CanQuery = true`** (los tools hacen raycast a ellas). Las frutas de las multi-harvest: `CanQuery = false`.
- Las frutas llevan attributes `GardenId`, `SlotId` (de su planta) y `FruitSlot`.
- Bucle de restauración: separar la clave con `string.find(key, ":")` + `sub`, **no** con el patrón `":(%d+)$"`.
- **Listener de `PlantEffectService`** (efecto aplicado / expirado): **esperar** a que el servicio exista (máx. 30 s, con un `warn` si no aparece). Un `getInstance()` único fallaba según el orden de arranque, y entonces las regaderas no aceleraban el visual.
- `reSyncFruitTimers(gardenId, plotId)`: construir la clave con el parámetro recibido (había una referencia a `slotId` sin definir → `"1:nil"`).
- Se eliminó `_getPlotService` (código muerto).

---

## SYS-05 — Frutas y cosecha (`HarvestService`, handlers y controller)

**Tipos:**
- **Multi-harvest** (tomate, papa, avocado…): la planta crece; al madurar, cada fruta tiene `FRUIT_COOLDOWN = 60` s + su `generationTime` (240–900 s según la planta). Los prompts están en el `Attachment "ProximityPrompt"` dentro del `FruitConector` de cada fruta.
- **Single-harvest** (carrot, nabo, astral_nabo, petunia, astral_petunia, red_rose, astral_red_rose): **la fruta es la planta**. No tienen `generationTime`; su único tiempo es el `growthTime`. El `Attachment "ProximityPrompt"` está dentro de un Part cualquiera del modelo `Fruit` (a veces `Center`).

**Implementación final [CONFIRMADO]:**
- **Single-harvest:** el runtime de la fruta se crea **al plantar** (`_connectCreated`) con `vStart = now`, `rAt = now + growthTime`, y se programa con `_scheduleFruit` (`task.delay` exacto) → `_notifyFruitVisual` (timer) → a los `growthTime` s exactos `_notifyFruitReady` → `_completeFruitVisual` (prompt de cosecha). Es el mismo mecanismo que las multi-harvest. No depender del scheduler de 5 s ni de `maturePlant`.
- **Multi-harvest:** `initializeFruits` al madurar la planta → `_scheduleFruit` con `task.delay(FRUIT_COOLDOWN)` y `task.delay(FRUIT_COOLDOWN + generationTime)`.
- **Boost:** loop de `HarvestService` con `TICK_INTERVAL = 1` s: para plantas con efecto activo, adelanta `regrowAt` según la velocidad y **reprograma** el `task.delay`.
- **La madurez la decide solo el servidor:** una fruta solo aparece con prompt de cosecha si el servidor ya la marcó lista (`regrowAt = nil`). Progreso visual = `generationTotal − (regrowAt − now)`, **sin volver a multiplicar por la velocidad** (el doble conteo hacía que con la Cosmic la fruta pareciera madura con 282 s restantes).
- Sin ningún `tonumber(plotId)`: `_scheduleFruit` (notificaciones), `restoreFruits` (no saltar UUID), `getGrowthRate(gardenId, slotId)`, sync de mutaciones y `_mutationDelays`.
- `canAccessPlot` → **`canAccessGarden`**.
- Se eliminaron las llamadas a `ParcelGameplayService` (`addHarvestXP` y `parcelStars` en `initializeFruits` y en el respawn) → en su lugar, `GardenProgressService:addXP` (ver `02c`).
- `FruitHarvestHandler` y `FruitSkipHandler`: sin `type(plotId) ~= "number"` (aceptan UUID).
- `FruitHarvestController` (cliente): lee el attribute **`SlotId`** (no `PlotId`).

**Producto Robux "madurar fruta al instante"** (25 Robux, id `3711141516`): ya no hay forma de abrirlo (el prompt de timer no es interactivo). `FruitSkipHandler` se conserva intacto para usarlo desde otra UI en el futuro (decisión pendiente del usuario).

**MutationGUI de frutas/parcelas:** el usuario dijo "podemos obviarla" (estaba ligada a parcelas). **UNKNOWN** su estado final: auditar y preguntar.

---

## SYS-06 — Timers (BillboardGui, cliente `PlantVisualController`)
- Los **prompts de timer** (`IsFruitTimer = true`) quedan **deshabilitados** (`Enabled = false`, `ClickablePrompt = false`): solo sirven como marca interna. La E aparece **solo** en las frutas maduras cosechables.
- El cliente dibuja **un único** texto flotante (BillboardGui, fuente **Fredoka**, texto blanco con borde negro grueso):
  - **Frutas:** `Adornee` = `Attachment "ProximityPrompt"` de la fruta; `StudsOffset = (0, 1.2, 0)`.
  - **Plantas en crecimiento** (multi-harvest): **3 studs** sobre el `PlantAttachment`; muestra el tiempo de crecimiento de la planta. Las single-harvest usan su timer de fruta (no se duplica).
  - Solo se muestra **el más cercano** (planta o fruta), en pantalla y a ≤ **`TIMER_RANGE = 12`** studs.
  - **Responsive:** tamaño del texto = **4,8 % de la altura de la pantalla**, mínimo **28** y máximo **56** (≈ 28 en móvil horizontal, 38 en tablet, 51 a 1080p, 56 a 1440p+). Se recalcula al cambiar la ventana o rotar el móvil.
  - Se acelera en vivo con regaderas y sprinklers, y vuelve a la normalidad al expirar el efecto.
- **No** tocar los timers de huevos ni los de la PowerStation con este sistema (los huevos tienen su propio prompt; ver `02c`).
