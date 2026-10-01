# 02c — SISTEMAS: HUEVOS, PETS, GARDEN LEVEL, MIGRACIÓN, LIMPIEZA

## EGG-01 — Huevos en soportes `EggPlaced`
**Objetos manuales (usuario):** `Garden_001/Eggs/EggPlaced` ×**14** (7 a la izquierda, 7 a la derecha; eran 13 y el usuario añadió uno), cada uno con un `Attachment "EggAttachment"` donde va el huevo. Soportes **rojos**. Ver `03`.

**Preparación por código:**
- Attribute **`EggSlot`** 1–14, numerado por posición (1–7 en un lado y 8–14 en el otro, en orden estable) para que los clones hereden la misma numeración.
- `CanQuery = true` en los `EggPlaced` (deben ser clickeables).
- Si el usuario **duplica un soporte**, la copia hereda el mismo `EggSlot` → hay que **renumerar** (dos soportes con el mismo número se confunden).

**Módulo nuevo `EggSlots`:** devuelve el soporte N de un jardín y la posición de su `EggAttachment`. Lo usan todos los scripts de huevos.

**Adaptados:** `EggVisualService`, `EggPlacementService`, `Init` (restauración e inicio del hatch), `EggService`, `LuckyBlockService` y la colocación de huevos en `ToolUseController`. Sin ninguna referencia a parcelas.

**Colocar:** con el huevo equipado, click/toque en un `EggPlaced` libre → el huevo aparece en su `EggAttachment`.
- **Límite:** `EggPlacementConfig.maxEggsPerGarden = 8` (cuenta el total, no qué soportes). El jugador elige cualquier soporte, sin orden. Para el futuro: un pase de Robux subirá el límite a 13/14 (solo cambia ese valor según el pase).
- Soporte ocupado → "This egg stand already has an egg!".
- Click en el jardín pero no en un soporte → aviso con OK **"Eggs go on the red stands around your garden!"**, una vez **por cada intento**. Mientras está abierto no se cuentan clicks, y durante 0,3 s después del OK el mismo click no lo reabre. Fuera del jardín o sin huevo equipado no sale nunca.

**Datos:** el namespace `"eggs"` antes guardaba `{gardenId, plotId, relativePosition}`; ahora `plotId` = número de soporte + una marca del sistema nuevo. **Huevos viejos** (por parcela, sin marca): al restaurar se **reubican solos en soportes libres en orden** (1, 2, 3…), **conservando la incubación**. **No** pasan por `GardenMigration` (decisión: mandarlos al inventario haría perder el progreso).

**Crecimiento:** escala **x4** desde la escala inicial (antes x1,4). Medidas: 1,8 × 2,4 al colocarlo → 7,2 × 9,6 al madurar. Está en dos lugares de `EggVisualService` (crecimiento normal y skip por Robux). El skip calcula el tamaño final **desde la escala inicial** (no desde la actual). Con x4 los huevos vecinos quedan casi tocándose (soportes de 5,5 de ancho a ~8 studs entre centros): revisarlo visualmente.

**Prompt:** se engancha al **`Attachment "ProximityPrompt"` del asset del huevo** (el usuario lo movió en los assets), con la punta del huevo como alternativa si no existe. El Attachment escala con el huevo.
- Timer de incubación y prompt de hatch: **`MaxActivationDistance = 9`** los dos (el del timer estaba en 25).

**Hatch:**
- El servidor valida **9 studs horizontales al soporte (+1 de tolerancia)** (antes no validaba nada).
- Rechazos con aviso: "Get closer to the egg!" / "This egg isn't ready to hatch yet!". Orden correcto de los argumentos de la respuesta: **`uuid, ok, animIndex, soilCF, reason`** (5 rechazos los enviaban corridos y `reason` llegaba `nil`).
- **Cooldown de 0,2 s** entre intentos en el cliente, comprobado **antes** de marcar "hatch en curso" (si no, se bloquea el huevo hasta reconectar). Sin el cooldown, mantener la E disparaba unos 18 rechazos seguidos.
- Sin teleport durante el hatch. El huevo viejo **se renombra** al empezar a desarmarse, para que el nuevo pueda aparecer en el mismo soporte.

**`EggAnimator` (LocalScript en `StarterPlayerScripts`, solo cliente):**
- **Vibración al madurar** (detecta sola el estado "listo"): ~0,8 s inclinándose hasta 7° sobre la base, con amortiguación; reposo aleatorio de 1,2 a 2,4 s; en loop y cambiando de dirección. Constantes: `WOBBLE_ANGLE`, `WOBBLE_TIME`, `REST_MIN`, `REST_MAX`.
- **Sonido de vibración** `rbxassetid://9113994447` al inicio de cada vibración: caída **lineal**, volumen completo a 2 studs y silencio total a **9 studs** (radio de hatch). `WOBBLE_SOUND_VOLUME`.
- **Desarme al hatch:** el servidor avisa a **todos** los clientes antes de destruir el modelo. Cada cliente clona el huevo, oculta el original y suelta sus **16 piezas con física real** (salen girando hacia afuera y arriba, rebotan y se desvanecen en ~2 s).
- **Sonido de hatch** `rbxassetid://9113959343` en 3D: volumen completo a 10 studs y silencio a 80. `HATCH_SOUND_VOLUME`.
- Los sonidos respetan el silencio de efectos del jugador.

**Tiempos:** Sweet Egg = **18000 s (5 h)** en `EggIncubationRegistry` (se puso a 60 s para probar y se restauró). Premium Sweet Egg y Nebula Sweet Egg estaban en **30 s** en el DEV: **UNKNOWN** si es el valor de producción → comparar con el principal y **no cambiar** sin preguntar.

**Visuales de huevos/cofres:** todo en el cliente (Nebula en mano: su propio LocalScript; colocado: `NightfallBlossomAnimator`; partículas: nativas; Galaxy Chests: `SpaceChestController`). Sin cambios.

## PET-01 — Habilidades sobre plantas
- **Sweet Frog / Nebula Sweet Frog:** caminan hasta la planta más cercana, animación y boost de crecimiento. **Sweet Dog / Nebula Sweet Dog:** igual, pero buscan una planta de papa.
- Posición: el **modelo de la planta**, o `plantState.instance.x/z` si el modelo no cargó. **Sin `getPlotPos`.**
- Usar `slotId` (las cuatro usaban `plantState.plotId`, que no existe → nunca aplicaban el efecto).
- **Highlight** de la planta afectada durante 3 s (vía `PetAbilityHighlight`).
- La Sweet Frog tenía restos de parcelas (limpiados en la fase final).

## PET-02 — Remotes de pets disponibles desde el arranque
`PetAbilityHighlight`, el remote de **sonido de habilidades**, el de **sonido de éxito del Lucky Block** y el de **sonido de éxito del Bunny**: deben **existir desde el inicio**. El cliente los esperaba 10 s y el servidor los creaba más tarde, así que fallaban según la sesión. Los scripts que los crean deben buscarlos antes y **no duplicarlos**.

## PET-03 — Pets de huevos
- **Lunaris / Astral Lunaris:** compatibles **sin cambios** (usan la lista de incubación del jugador). El "salto de 2 s a 1 h" que se vio era el **timer de otro huevo cercano**, no un bug (comprobado con un diagnóstico que nunca detectó subidas de tiempo). Opcional, no decidido: mostrar solo el timer del huevo más cercano.
- **Lucky Block:** vuelve a poner el huevo en el **mismo soporte**. Si está ocupado (el hatch dio otro huevo), en el **primer soporte libre**; si no hay ninguno, no lo pone.

## LVL-01 — Garden Level + Resonancia
**Diseño final del usuario:**
- Nivel del jardín **1–100**. Sube con XP por **cada fruta cosechada**: **XP = número de rareza** de la fruta (tabla de rarezas existente: Common = 1 … OdysseySecret = 8). No duplicar la tabla.
- **Curva de pets:** XP para pasar de N a N+1 = **6 × N² + 44**. Total de 1 a 100 = 1.974.456 XP.
- El nivel 1–100 **no da bonus**: solo habilita la resonancia.
- En el nivel 100: barra llena y **dorada**, con "Lv. 100 MAX". Se habilita, **solo para el dueño**, el `ProximityPrompt` **"Resonate (1,000,000)"** (mantener 1 s) en el Attachment `ProximityPrompt` del cartel → **pide confirmación** → cobra **1.000.000 Odyssey Coins** → vuelve a **nivel 1 con 0 XP** → **+1 resonancia**. Si no alcanzan las monedas, avisa.
- **Bonus:** **+0,05 %** al precio de venta por nivel de resonancia, con **tope del 50 %** (1000 resonancias). Cada fruta **guarda la resonancia al cosecharse** (sello antes de entrar al inventario) y se vende con ese bonus. Inyectar el bonus solo para la operación de venta (regla de `gosa`), nunca de forma global.
  - Valores anteriores **descartados**: 5000 niveles al 0,01 %.
- **Cartel:** el `SurfaceGui` del modelo `GardenLevel`, todo centrado: título **"Garden Level"** (o "Garden Level N" con N resonancias), **"Lv. X"**, barra y **"340 / 1,200 EXP"**. Se actualiza **en vivo** al cosechar (por evento) y los visitantes ven lo mismo.
- **Archivos:** `GardenProgressService` (servidor) + `GardenProgressConfig` (todos los números) + otros 3 (5 en total; nombres de los otros 3 **UNKNOWN**). Datos: nivel, XP y resonancia por jugador en un namespace propio (nombre **UNKNOWN**).
- `HarvestService` llama a `GardenProgressService:addXP(...)` donde antes estaba `ParcelGameplayService:addHarvestXP`.
- **Limitación aceptada:** el precio que se ve en el prompt de la fruta antes de cosecharla no incluye el bonus (en el inventario y al vender sí).
- **Balance pendiente:** muy lento (cientos de miles de cosechas por resonancia). Propuesta del agente: un multiplicador de XP (×10, ×50) o ajustar la curva. **Lo decide el usuario**; no cambiarlo sin preguntar. Para probar la resonancia se puede bajar temporalmente el nivel máximo a 2 y luego **volver a 100**.

## MIG-01 — `GardenMigration` (versión final)
- **ModuleScript** con constante **`ENABLED`** (en el juego principal **`false`** hasta la Sección 13 de `06`).
- Lo llama **`PlantGrowthSystem.Init` justo antes de `restorePlants`** de cada jugador, **una sola vez** por jugador. Flag en el perfil: **`gardenReworkMigrated = true`**.
- Detecta las plantas de formato viejo (clave numérica de parcela). Los datos están en `properties` (el `plantInstance`).
- **Reglas:**
  | Caso | Resultado |
  |---|---|
  | Planta **madura y extraíble** | **Plant item** vía `PlantFactory.fromExisting()` (tamaño, variante y mejoras) |
  | Planta **en crecimiento** o **no extraíble** | **Su semilla**, buscada por la relación real planta → semilla de las definiciones (`plantDefinitionId`; verificar el campo real) |
  | **Frutas maduras** de esas plantas | Al inventario como frutas |
  | El inventario no tiene espacio | **No** se marca como migrado; se reintenta en la próxima entrada |
  | Huevos | No los toca (el sistema de huevos los reubica) |
- Al terminar: limpia las plantas viejas del perfil, pone el flag y guarda.
- **Versiones descartadas:** (1) un Script deshabilitado conectado a `PlayerAdded` (podía correr **después** de cargar el jardín); (2) buscaba la semilla por un **campo que no existe** (se habrían perdido todas las plantas en crecimiento).
- ⚠ Verificar que un reintento tras un inventario lleno **no duplique** lo que sí se entregó.

## CLEAN-01 — Limpieza final
**Eliminado (borrado, no deshabilitado):**
- **Sistemas:** `PlotPresenceSystem`, `ParcelGameplaySystem`, `ParcelGuiSystem/ParcelGuiBridge`, `ParcelGuiSystem/ParcelUnlockHandler`, `PlotSystem` (`PlotService` + `PlotRegistry`), `HighlightBridge`, `AbilitySystem.ParcelDetector`, `AccessService.canAccessPlot`, `DeletePlantHandler` (+ 2 remotes), `RagdollTester`.
- **Cliente:** `ParcelGuiController`; `ParcelDetailsButtonGui`, `ParcelGui` y `TopNavGui/NavBar/PARCELButton` (en el DEV se desactivaron / se ocultaron; según la preferencia del usuario, **borrarlos**, verificando antes que nadie los referencia).
- **`HighlightSystem/Init`:** desactivado (raycast por frame sin filtro sobre `BaseSoil`). `HighlightService` es un ModuleScript que no corre si nadie lo `require`. Los highlights de planta y fruta se iban a adaptar: **UNKNOWN** si se adaptaron → auditar.
- **Las 160 parcelas** de cada jardín (`Parcels`), con todo su contenido.
**Ajustado:** tutorial (el beam apunta al `GardenFloor`), sonido de los sprinklers, Sweet Frog.
**Se conserva:** `ParcelInteractionService` (solo `_activeSlot`), `GardenZone`, `PlantingGuard`, `GardenService`.
**Fuera de alcance:** `GamepadController` (inerte: no da error y no encuentra nada).
