# 02b — SISTEMAS: TOOLS

## Base común
**Cliente — `ToolUseController`:**
- `getPlantModelUnderMouse(inputPos)`: raycast desde `cam:ScreenPointToRay(inputPos)` (mouse o toque). Desde la parte impactada, **sube por los ancestros hasta el Model que tenga los attributes `GardenId` y `SlotId`** (el `Plant_*`). No depende del `PrimaryPart`.
  - Las frutas también tienen esos attributes (con el `SlotId` de su planta) → hacer click en una fruta apunta a su planta. Comportamiento vigente; el usuario no pidió cambiarlo.
- `handlePlantToolClick(model, def)`: lee `GardenId` y `SlotId`, setea `EquipState.activePlant = model` y envía `ToolUseRequest:FireServer(gardenId, slotId)`. Sustituye a `handlePlotToolClick`. **Sin teleport ni bloqueo de movimiento.**
- Herramientas **de área** (regaderas, sprinklers): usan el mismo raycast al `GardenFloor` que las semillas (atraviesa las plantas) y envían `(gardenId, x, z)`.
- **Cooldown anti-misclick:** `TOOL_COOLDOWN_DEFAULT = 0.2` s para todas las herramientas, y una tabla `TOOL_COOLDOWN` de excepciones con **`magic_wand = 0.1`**. El chequeo va **antes** de marcar cualquier estado de "esperando respuesta" (si va después, deja el tool bloqueado).
- Rechazos del servidor: mostrar el aviso al jugador (había una llamada a una función eliminada en la línea del manejo de errores que rompía el cliente).

**Servidor — `ToolUseHandler`:**
- Acepta `(gardenId, slotId)`, o `(gardenId, x, z)` para las herramientas de área.
- **Valida:** acceso al jardín, que el `slotId` exista (`invalid_slot` si no) y la **distancia horizontal**: `plantPos = Vector3.new(px, hrp.Position.Y, pz)` (solo cuentan X/Z, medido a la base de la planta).
- Rango por defecto **25 studs**; **pala, trowel y extractor: 9 studs**; aviso "Get closer to the plant!".
- Inyecta `_activeSlot`. `resolveTarget` devuelve `{gardenId, plotId = slotId}` (compatibilidad con las acciones).

**Servidor — `ToolActionImplementations`:** cada acción recibe `plot.plotId = slotId` (string). Sin `plotId` numérico ni `BaseSoil`.

**Inventario de tools (28):** 21 interactúan con el jardín (4 regaderas, trowel, extractor, pala, 10 amuletos, 4 sprinklers + Plant items). Los 7 restantes (linterna, varita mágica, bat, premium bat, blessed restoration scroll, 3 galaxy chests) no tocan el jardín, pero también les aplica el cooldown. El cofre se abre por `ToolUseController`; su ruleta (`SpaceChestController`) no cambia.

---

## TOOL-01 — Regaderas (4 tiers) — riego por ÁREA
- **Diseño final (idea del usuario):** se apunta a cualquier punto del jardín (raycast al `GardenFloor`, como las semillas), el agua cae ahí y **todas las plantas dentro del radio** reciben el boost. Efecto **instantáneo** sobre las plantas que están en ese momento (a diferencia del sprinkler).
- **Radios** (`WATER_RADIUS` en `ToolActionImplementations`, valor por defecto 2,5):
  | Regadera | Radio |
  |---|---|
  | watering_can | 2,5 |
  | rare | 3,75 |
  | ultrarare | 5 |
  | cosmic | 6,25 |
- El servidor valida que el punto esté en el jardín y a ≤ 25 studs, busca las plantas en el radio con `getGardenPlants` y **envía el radio al cliente**.
- **Consumo:** 1 uso por riego, sin importar cuántas plantas toque. Sin plantas en el radio → **no consume** y muestra "There are no plants in that area to water!".
- **Cliente — `WaterCanVFXController`:** al recibir `GrowthSpeed` reproduce el sonido, la **animación del personaje regando** y el VFX de agua repartido sobre el área (diámetro = 2 × radio). `spawnWaterEffect` va dentro de un **`pcall`** (un error visual no debe bloquear el consumo). Al terminar envía **`WaterComplete`** → el servidor aplica `AbilityRegistry.GrowthSpeedSingle.applyEffect` y consume el uso.
- **Bug crítico ya resuelto:** existían **dos RemoteEvents `WaterComplete`** en `ReplicatedStorage`; cliente y servidor usaban uno distinto cada uno y el riego nunca se consumía. Debe haber **uno solo**.

## TOOL-02 — Sprinklers (4 tiers) — ZONA activa
- Se colocan en **cualquier punto del suelo** (no hace falta que haya una planta). **Sin confirmación** (se eliminaron también los textos de confirmación en `ItemDefinition`). **Sin restricción de distancia entre sprinklers** (se pueden apilar).
- **Radios** (`RADIUS_BY_DEF` en `PlaceSprinkler`): Basic **7**, Rare **14**, UltraRare **21**, Cosmic **28** studs (equivalen a 1–4 parcelas de 7 studs centro a centro).
- **Duración:** 300 s los cuatro (`duration` en `PlantEffectRegistry`). **Velocidad:** +2 / +4 / +9 / +17.
- **Apilado:** tiers **distintos se suman** (los 4 → x33); el **mismo tier no suma**, solo renueva la duración. El tamaño de las frutas usa las probabilidades del **tier más alto** que cubre la planta.
- `SprinklerService` guarda cada sprinkler activo `{position, radius, effect, expiresAt}`:
  - al colocarlo aplica el efecto a las plantas del radio (`applyGrowthEffect`);
  - **las plantas que se planten o se muevan con el trowel dentro de la zona** reciben el efecto por el tiempo restante (`applyEffectWithRemaining`);
  - al expirar destruye el modelo y limpia los efectos.
- Los modelos colocados van en la carpeta **`Sprinklers`** de cada jardín (el servidor la crea si falta, pero el sonido de ese jardín no se engancha hasta que el jugador reconecta → debe existir en cada jardín). El script de **sonido** (cliente) observa esa carpeta y suena si estás dentro del jardín. La limpieza al salir del jugador también usa esa carpeta.
- **Efecto secreto (combo de 4 tiers): ELIMINADO** por decisión del usuario (no queda ninguna referencia).
- **`GiroScript`** (rotación visual): `Script` con **`RunContext = Client`** (un `LocalScript` dentro de workspace no se ejecuta). Era un Heartbeat en el servidor.
- `ParcelDetector` no se usa (eliminado en la limpieza).

## TOOL-03 — Pala (shovel)
- Click en el modelo de la planta (cualquier parte visible). Rango de **9 studs horizontales** a la base.
- Confirmación de borrado (existente) → animación de cavar **en loop durante 2 s** (`totalDur = 2` en `ShovelVFXController`; antes 6 loops × 1,5 s = 9 s) → `ShovelComplete` → el servidor borra la planta y sus frutas.
- Se eliminó el "restaurar movimiento" (`WalkSpeed = 16`, `JumpPower = 50`): era un resto del teleport y pisaba los boosts del jugador.
- Se eliminaron **`DeletePlantHandler`** y sus 2 remotes (sistema viejo de la GUI de parcelas que solo aceptaba IDs numéricos).

## TOOL-04 — Trowel (mover planta)
- **Levantar:** click en una planta **madura** a ≤ 9 studs → confirmación "Move X somewhere else? This will use 1 Trowel." → la planta sigue al cursor, la original queda semitransparente y se rota con las flechas (botones en pantalla en móvil). El jugador se mueve con normalidad.
- **Colocar:** click en **cualquier punto de su jardín**, sin límite de distancia. El raycast atraviesa las demás plantas y la que se lleva en la mano.
  - Destino válido → se mueve, recibe un **nuevo `slotId`** con `localX/localZ` recalculados, y **recién entonces se consume** el trowel.
  - Destino inválido (fuera del jardín o a < 1 stud de otra planta) → aviso y la planta sigue en la mano.
  - **Cancelar:** E o cambiar de herramienta → vuelve a su lugar y no se consume. La marca de "esperando respuesta" se limpia también al cancelar.
- Servidor `TrowelPlace(destGardenId, destWorldX, destWorldZ, yRotation)`.
- Conserva tamaño, variante, mejoras y rotación elegida. Las **frutas se reinician** al mover (igual que en el sistema viejo; se aceptó).
- **Single-harvest NO se pueden mover** (`noExtract`) → `no_plant_item`. Decisión: dejarlo así. (Opcional, no decidido: mostrar "This plant can't be moved. Harvest it instead!".)
- Se quitaron 3 restos del teleport que forzaban la velocidad.

## TOOL-05 — Plant extractor
- Igual que el trowel al levantar: planta madura a ≤ 9 studs → animación de **0,4 s** → la planta va al inventario como `Plant` item con todo su estado.
- El aviso final al servidor se envía **siempre** (antes dependía de que cargara la animación y, si no cargaba, la extracción nunca ocurría).
- Single-harvest no se pueden extraer (`noExtract`). Sin restauración de velocidad.

## TOOL-06 — Plantar un Plant item
- Igual que una semilla (SYS-02 en `02`): click en el suelo, 30 studs, 1 stud de separación, límite de 200. Se quitó la excepción de distancia que tenía por el teleport viejo.

## TOOL-07 — Amuletos `magic_amulet_4..13` (10)
- `ApplyAmulet` funciona sin cambios porque recibe el `slotId` vía `resolveTarget`. **El usuario aún no los implementó.** No tocar más allá de la detección.

## TOOL-08 — Seguridad de tools
- **`CanBeDropped = false` en todos los Tools** (50 revisados; solo `BlessedRestorationScroll` en `ServerStorage` estaba en `true`). Riesgo de duplicación con exploits si se dejan en `true`.
