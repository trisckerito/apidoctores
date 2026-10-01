# 02 — SISTEMAS (ESTADO FINAL)

> Cada sistema describe **la versión final a implementar**. La historia solo aparece cuando evita repetir un error.
> Etiquetas: **[CONFIRMADO]**, **[INFERIDO]**, **UNKNOWN / REQUIERE VERIFICACIÓN** (ver `00_MASTER_PLAN.md` §6).
> Rutas de scripts: las de la orden original. **Antes de crear algo, buscarlo en el proyecto**; puede existir ya en su versión final.

---

## SYS-01 — Modelo de datos de plantas (`slotId`)

**Propósito:** persistir las plantas de cada jardín con clave `slotId` (UUID) y posición XZ libre, en lugar de `plotId` 1..160.

**Estado final:** IMPLEMENTAR la versión del esquema D1 (propuesta del agente aprobada con "puedes continuar").

**Formato VIEJO** [CONFIRMADO, leído por el agente en `PlayerDataService`] — solo lo lee la migración:
```lua
plotSlots[slotIdx].gardens["main"].plants[tostring(plotId)] = {
    definitionId = "...",
    plantedAt    = os.time(),
    properties   = plantInstance, -- uuid, growthTierId, yRotation, etc.
}
```

**Formato FINAL** [INFERIDO: propuesta D1 aceptada; verificar en el proyecto]:
```lua
plotSlots[slotIdx].gardens["main"].plants[slotId] = {
    definitionId = "avocado",
    plantedAt    = os.time(),
    x            = 10.5,   -- world X
    z            = -44.2,  -- world Z
    properties   = plantInstance,
}
```

**Scripts:**
- `ServerScriptService.PlantGrowthSystem.PlantGrowthService`: es el dueño del estado de todas las plantas. Clave interna **`gardenId:slotId`** (antes `gardenId:plotId`). Métodos existentes: `createPlant`, `plotHasPlant` (adaptar o renombrar según convención; UNKNOWN), `onPlantCreated`, `onPlantRemoved`. Contiene `restorePlants` (carga al entrar el jugador).
- `ServerScriptService.PlantGrowthSystem.PlantFactory`: crea `PlantInstance`. `create()` genera `yRotation` aleatorio, **se mantiene igual**. `fromExisting()` reconstruye desde el estado guardado y lo usan la migración y los `Plant` items.
- `PlayerDataService`: persistencia (namespace exacto UNKNOWN; el agente lo encontró bajo `plotSlots`).

**Decisiones finales:**
- `slotId`: UUID string generado **en el servidor** al plantar. [INFERIDO: la orden dice "generado al plantar"; el servidor es autoritativo según `gosa`.]
- UNKNOWN: ¿el `slotId` coincide con `properties.uuid` del `plantInstance`? D1 decía "la migración convierte `plants[tostring(plotId)]` → `plants[uuid]` (usando el uuid del plantInstance)". Pero la migración final **no reubica plantas**, las convierte en items (ver SYS-10). Verificar en el proyecto.
- Los locks de `SeedService` pasan de `userId:gardenId:plotId` a **`userId:gardenId:slotId`** [CONFIRMADO como plan].

**Validaciones:** ver `07_VALIDATION_CHECKLIST.md` §S2.

---

## SYS-02 — `GardenFloor` y colocación libre (servidor)

**Propósito:** plantar en cualquier punto XZ del suelo del jardín.

**Objetos requeridos (MANUALES, ver `03`):** `GardenFloor` por jardín, con attribute `GardenId`.

**Flujo final (PC/Mobile):**
1. El cliente hace click/tap → raycast → impacta `GardenFloor` → obtiene `(x, z)` y el `GardenId` del suelo.
2. El cliente envía **`(gardenId, x, z)`** en lugar de `(gardenId, plotId)`.
3. Servidor (`PlantHandler` → `SeedService`):
   - `AccessService.canAccessGarden(player, gardenId)` (ya **no** existe `canAccessPlot`).
   - Valida que `(x, z)` esté **dentro de los límites del `GardenFloor`** de ese `gardenId` [INFERIDO: necesario para seguridad; implementación UNKNOWN].
   - Valida la **proximidad** del jugador al punto. Antes era un máximo de 20 studs al `BaseSoil`. Distancia final UNKNOWN; usar 20 si no hay otra, y confirmar con el usuario.
   - Valida el **radio mínimo de 5 studs** contra todas las plantas activas del jardín, y el **límite de 200 plantas** por jardín (valores de la orden; UNKNOWN si cambiaron, ver `GardenProgressConfig` u otra config).
   - Genera `slotId`, consume la semilla (`consumeReferenceGetEntry`), crea la planta con `PlantFactory` y la registra en `PlantGrowthService`.
4. `PlantVisualService` crea el modelo **`Plant_<gardenId>:<slotId>`** en `(x, Ytop(GardenFloor) + offset, z)` con la `yRotation` del `plantInstance`, y le pone el attribute `slotId` [INFERIDO].

**Scripts:**
- `ServerScriptService.SeedPlacementSystem.PlantHandler`: recibe `PlantRequest`. Antes inyectaba `activePlot` en `ParcelInteractionService` y mantenía `HasPlant` en el `BaseSoil`. **Eso desaparece.** Maneja semillas (`SeedService.plant()`) y `Plant` items (`PlantFactory.fromExisting()`).
- `ServerScriptService.SeedSystem.SeedService`: deja de llamar `getActivePlot(player)`. Recibe la posición/slot directamente. Conserva los locks y la deduplicación por UUID.
- `PlantVisualService`: antes usaba `plot.plantPivot`. Ahora posiciona por XZ.
- `ServerScriptService.AccessSystem.AccessService`: solo `canAccessGarden`.
- `PlantingGuard.isPlanting()`: **se preserva igual**.

**Bug histórico que NO debe repetirse:** apoyar la planta en la altura de `BaseSoil`. Al subir el usuario el suelo, las plantas quedaron mal posicionadas. **Final: la altura sale del `GardenFloor`** (cara superior: `Position.Y + Size.Y/2`). Ver `05` B3.

**RemoteEvents:** `PlantRequest` (existente). Firma final UNKNOWN: se infiere `(action, gardenId, x, z)`. Verificar en el proyecto antes de cambiar nada.

---

## SYS-03 — Detección de presencia / plot activo (ELIMINADO → basado en eventos)

**Estado final:** sin polling. El cliente manda la posición (plantar) o el `slotId` (tools) en el propio request, y el servidor valida. No existe "plot activo" por jugador.

**Se elimina:**
- `ServerScriptService.PlotPresenceSystem.PlotPresenceService` (raycast cada 0,12 s desde la cabeza del jugador + `GardenZone` Touched/TouchEnded).
- `ServerScriptService.ParcelInteractionSystem.ParcelInteractionService` (`_activePlot[userId]`).
- **UNKNOWN** si ambos se borraron físicamente en el desarrollo original (no figuran en la lista final de eliminados). Al implementar: eliminar solo en la sección de limpieza, después de comprobar que nadie los referencia.

**Se conserva:** `GardenZone` (zona de entrada de cada jardín) [CONFIRMADO "puede reutilizarse"]. Uso final concreto UNKNOWN.

---

## SYS-04 — Tools (servidor)

**Scripts:**
- `ServerScriptService.ToolUseHandler`: recibe `ToolUseRequest`. Final: **`(gardenId, slotId)`** en lugar de `(gardenId, plotId)`. Ya no inyecta `activePlot`. Valida el acceso al jardín, que el `slotId` exista en ese jardín y la proximidad del jugador a la planta. Resuelve el UUID del item equipado (sin cambios).
- `ServerScriptService.ToolSystem.ToolActionImplementations`: `PlacePlant`, `DeletePlant`, `ExtractPlant`, `MovePlant`, `WaterPlant` y `PlaceSprinkler` reciben un destino nuevo en lugar de `plot = {gardenId, plotId}`:
  - Acciones sobre una planta existente → `{ gardenId, slotId }`.
  - Acciones que colocan algo nuevo (`PlacePlant`, `MovePlant` destino, `PlaceSprinkler`) → `{ gardenId, x, z }`, con la misma validación que SYS-02.
  - UNKNOWN: la forma exacta de la tabla. Leer el proyecto.

---

## SYS-05 — Cliente PC / Mobile

**Scripts (`StarterPlayer.StarterPlayerScripts.ToolSystem`):**
- `ToolUseController`:
  - Semillas y `Plant` items: raycast al **`GardenFloor`** (en lugar de `getSoilUnderMouse`/`getSoilFromPlantModel`) → envía `(gardenId, x, z)`.
  - Tools: raycast al **modelo de la planta** → lee el attribute `slotId` del modelo (subiendo por los ancestros hasta el `Model` `Plant_*`) → envía `(gardenId, slotId)`. Sustituye a `handlePlotToolClick`.
  - Ya no usa `_GardenId` / `_PlotId`.
- `TrowelController` (mover plantas): **propuesta D5** [INFERIDO como final]: recoger la planta → click en un punto libre del `GardenFloor` → el servidor valida el radio mínimo → la mueve. UNKNOWN si quedó exactamente así.
- `PlantVFXController`: antes leía `EquipState.activeSoil` (`BaseSoil`). Final: usa la posición/planta objetivo. Detalle UNKNOWN.
- **GUI de planta**: al clickear o tocar una planta se muestran los detalles de esa planta (pedido en la orden). **UNKNOWN** si se implementó. Preguntar al usuario.
- **Botón PLOT**: debe desaparecer (orden). **UNKNOWN** si se quitó.

**Restricción de raycast** [INFERIDO]: para clicks con un tool, el raycast debe ignorar el personaje y, al plantar, solo aceptar `GardenFloor`. Ver `04` P6.

---

## SYS-06 — Sprinkler con radio real

**Estado final:** `ParcelDetector` (BFS en grid de `BaseSoil`) **eliminado** [CONFIRMADO]. Se reemplaza por un **radio circular en XZ**: distancia euclidiana `sqrt((px-sx)² + (pz-sz)²) <= radio` sobre las posiciones `(x, z)` de las plantas activas del jardín.

**Objetos:** carpeta `Sprinklers` en cada jardín [CONFIRMADO que existe]. **UNKNOWN** si la creó el usuario o el código.

**Datos:**
- Radio por tipo (basic / rare / ultrarare / cosmic): **UNKNOWN** (pregunta D6 sin respuesta registrada). Buscarlo en la config o definición de sprinklers del proyecto. Si no existe → **preguntar al usuario** (es gameplay).
- [CONFIRMADO] Se limpiaron referencias a parcelas en el **sonido** de los sprinklers.

**Colocación:** `PlaceSprinkler` en una posición XZ libre [INFERIDO de D6].

---

## SYS-07 — Huevos (soportes)

**Estado final** [CONFIRMADO]: "el sistema de huevos ya los reubica solos en **soportes**, conservando su incubación". Hay una carpeta **`Eggs`** por jardín. **Sweet Egg** = 5 horas de incubación.

**UNKNOWN / REQUIERE VERIFICACIÓN:**
- Si este sistema de soportes ya existía antes del rework o se creó en él.
- Estructura de la carpeta `Eggs` (número de soportes, nombres, attributes).
- Qué pasó con `EggPlantRequest` (antes usaba `BaseSoil`/`plotId`).

**Acción:** auditar en la Sección 0. Si no existe, **detenerse y preguntar** antes de diseñar nada.

---

## SYS-08 — Consola (`GamepadController`) — FUERA DE ALCANCE

**Estado final** [CONFIRMADO]: no se adaptó. Queda inerte (no da error y no encuentra `BaseSoil`).
**NO HACER:** reescribirlo en esta recuperación. Si el usuario lo pide más adelante, el diseño de la orden era: el joystick derecho navega entre **modelos de planta** activos, `ButtonA` selecciona y el highlight va sobre el modelo. **No** volver a recolectar todo cada 5 s (ver `04` P5).

---

## SYS-09 — Garden Level y Resonancia

**Propósito:** progresión por jardín. Cada fruta cosechada da XP, el nivel llega a 100 y en el 100 se puede "resonar" (prestigio).

**Scripts** [CONFIRMADO: 5 scripts; nombre conocido solo `GardenProgressConfig`]. Nombres del resto UNKNOWN. Estructura sugerida si no existen (siguiendo `gosa`): un servicio de servidor dueño del estado y la lógica, un controlador o script de cliente para el cartel y la confirmación, y la config.

**Config (`GardenProgressConfig`)** [CONFIRMADO: todos los números están aquí]:
| Parámetro | Valor final |
|---|---|
| XP por fruta | = número de rareza de la fruta (Common = 1 … OdysseySecret = 8). La tabla de rarezas ya existe en el proyecto; **no duplicarla**, leer su número. |
| Nivel mínimo / máximo | 1 / **100** (⚠ ver nota de nivel de prueba) |
| XP para pasar de N a N+1 | **6 × N² + 44** (la misma curva que los pets) |
| XP total 1 → 100 | 1.974.456 (verificación: Σ_{N=1}^{99}(6N²+44)) |
| Coste de resonancia | **1.000.000 Odyssey Coins** |
| Bonus por resonancia | **+0,05 %** del valor de venta de la fruta por nivel de resonancia |
| Tope del bonus | **50 %** (= 1.000 niveles de resonancia) |
| HoldDuration del prompt | **1 s** |

**Flujo final:**
1. Al cosechar una fruta → +XP (su rareza) al jardín del **dueño**. Al cruzar el umbral se sube de nivel (puede subir varios niveles de una vez [INFERIDO]).
2. En el nivel 100 → la barra queda llena y **dorada**, con el texto **"Lv. 100 MAX"**. La XP sobrante no se acumula [INFERIDO; verificar].
3. En el nivel 100, **solo para el dueño**, se habilita el `ProximityPrompt` "**Resonate (1,000,000)**" (mantener 1 s).
4. Al activarlo → **pide confirmación** → cobra 1.000.000 → **vuelve a nivel 1 con 0 XP** → resonancia +1. Si no alcanza el dinero, avisa.
5. **Cada fruta guarda el nivel de resonancia en el momento de la cosecha**, y al venderse aplica `+0,05 % × resonancia` (tope 50 %).

**Cartel (`SurfaceGui` en `GardenLevel`)**, todo centrado:
- Título: **"Garden Level"** si la resonancia es 0, y **"Garden Level N"** si es N > 0.
- **"Lv. X"**.
- Barra de progreso.
- Texto **"340 / 1,200 EXP"** (formato con separador de miles).
- Se actualiza **en vivo al cosechar**, y los visitantes lo ven igual.

**Discrepancia con la petición del usuario:** el usuario escribió "volverá a nivel 0 1/x". El agente implementó "vuelve a nivel 1 con 0 XP". **Implementar nivel 1 con 0 XP** (versión final reportada sin objeción del usuario), pero mencionarlo en el reporte de la sección.

**Limitaciones conocidas (aceptadas):**
- El precio que muestra el prompt de una fruta **antes** de cosecharla **no** incluye el bonus de resonancia (se aplica al cosechar). En el inventario y al vender sí aparece.
- Balance: llegar a 100 requiere ~2 M de XP con 1–8 XP por fruta, lo que es muy lento. El agente recomendó multiplicar la XP por fruta (×10 o ×50) o ajustar la curva. **Decisión del usuario: UNKNOWN.** No cambiarlo sin preguntar.

**Datos persistentes** [INFERIDO]: nivel, XP y resonancia por jugador/jardín, y el nivel de resonancia por fruta. Nombres de campos y namespace: **UNKNOWN**.

**Remotes:** para la confirmación de resonancia hace falta uno cliente→servidor (o uso de `PromptTriggered` en el servidor + remote de confirmación). Nombre UNKNOWN.

**Objetos manuales:** `GardenLevel` (ver `03` §3.4).

**⚠ Nivel de prueba:** el agente ofreció poner temporalmente el nivel máximo en 2 para probar. **UNKNOWN** si se hizo. Validar que el valor final sea **100**.

---

## SYS-10 — Migración de datos (`GardenMigration`)

**Estado final** [CONFIRMADO]:
- **ModuleScript** `GardenMigration`, con constante **`ENABLED = true`**.
- Lo llama **`PlantGrowthSystem.Init`** justo **antes** de cargar (`restorePlants`) las plantas de cada jugador. Corre **una sola vez por jugador**.
- Marca en el perfil: **`gardenReworkMigrated = true`**.
- **Protección en `restorePlants`**: no debe intentar restaurar plantas en formato viejo (`plotId`) como si fueran nuevas [INFERIDO sobre el detalle; la existencia de la protección está CONFIRMADA].

**Reglas finales:**
| Caso | Resultado |
|---|---|
| Planta **madura y extraíble** | → **`Plant` item** en el inventario (vía `PlantFactory.fromExisting()`), conservando tamaño, variante y mejoras |
| Planta **en crecimiento** (sea o no extraíble) **o no extraíble** | → **su semilla** al inventario |
| **Frutas maduras** de esas plantas | → al inventario **como frutas** |
| Inventario sin espacio | **NO** se marca como migrado; se reintenta en la próxima entrada |
| Huevos | **No los toca la migración.** El sistema de huevos los reubica en soportes conservando la incubación |

**Datos:** las propiedades de cada planta están en `properties` (es el `plantInstance`) [CONFIRMADO].

**⚠ Diferencia con la orden original** (implementar la FINAL, pero mencionarlo al usuario en el reporte):
- La orden decía que una planta extraíble *en crecimiento* se convertía en `Plant` item. La final la convierte en **semilla**.
- La orden decía que una `noExtract` madura se cosechaba y se eliminaba. La final da **semilla + frutas maduras**.
- UNKNOWN: si una `noExtract` madura da semilla **y** frutas, o solo frutas. Preguntar.

**Versiones descartadas:** ver `05` B1 y B2.

**Prueba en producción** [CONFIRMADO, respuesta del agente]: ver `06` Sección 12.

---

## SYS-11 — Residuos de parcelas en otros sistemas

[CONFIRMADO] Ajustados:
- **Tutorial**: el beam apunta al `GardenFloor`.
- **Sonido de sprinklers**: sin referencias a parcelas.
- **Sweet Frog**: sin referencias a parcelas.
- `ParcelGameplayService` (parcelas bloqueadas): **UNKNOWN** (D7).

---

## SYS-12 — Multi-jardín y capacidad del servidor

[CONFIRMADO]
- `GardenService` asigna jardines a jugadores (`getPlayerGarden`, `getGardenPlayer`). Sin cambios conocidos.
- Cada clon de `Garden_001` necesita:
  - un nombre único: `Garden_002`, `Garden_003`, …
  - attribute **`GardenID`** único en el **modelo** (2, 3, …).
  - attribute **`GardenId`** con el mismo número en su **`GardenFloor`**.
  - ⚠ las mayúsculas difieren (`GardenID` vs `GardenId`) según el reporte. Verificar la grafía exacta en el proyecto.
- **Max Players del place = número de jardines** (ej. 6 bases → 6 jugadores). Si hay más jugadores que jardines, el excedente entra sin jardín.
- Huevos, sprinklers, cartel y migración no dependen del número de bases.
