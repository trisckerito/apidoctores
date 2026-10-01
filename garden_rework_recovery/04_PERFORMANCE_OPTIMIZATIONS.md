# 04 — OPTIMIZACIONES DE RENDIMIENTO (todas a reaplicar)

> Regla general acordada: **decoración = `CanQuery = false` y `CastShadow = false`**. Todo lo **interactivo** conserva `CanQuery = true`.
> **Mantener `CanQuery = true` en:** `GardenFloor`, modelos de plantas (`Plant_*`), pets, NPCs, `EggPlaced`, `RainbowFruitVFX` y cualquier cosa que el jugador deba clickear o que un sistema detecte por raycast. **Antes de aplicar en masa, auditar los raycasts** del juego.

## P1 — Eliminar el polling de presencia
- **Problema:** `PlotPresenceService` hacía un raycast de 700 studs hacia abajo cada 0,12 s por jugador en su `GardenZone` (~66 raycasts/s con 8 jugadores contra 1.280 `BaseSoil`). Era el mayor consumidor de CPU.
- **Final:** borrado. Plantar y usar tools envían la posición o el `slotId` en el request (por evento).
- **No volver a hacer:** loops que calculen "dónde está parado el jugador".

## P2 — Eliminar las 1.280 parcelas
- **Antes:** `Gardens` con ~20.600 descendientes. **Después:** 1.898 en Garden_001.
- Mientras existan las parcelas: `BaseSoil.CanQuery = false`.

## P3 — `CanQuery` / `CastShadow` en el mapa
- **12.390 partes → `CanQuery = false`.** Principales: ~9.840 `Part` (baldosas/decoración) dentro de `Gardens`, 192 troncos, 192 `Union`, los Water y ~2.018 decoraciones en la raíz de workspace.
- **2.369 partes → `CastShadow = false`** (incluido `GardenFloor`).
- Motivo: el cliente hace raycasts del mouse cada frame; menos partes consultables = más FPS. Las sombras cuestan GPU cada frame.

## P4 — Clima (`ReplicatedStorage.WeatherParticles`) y efectos dinámicos
- Plantillas: **213 propiedades** corregidas. Neon Stars (64 partes), auto McLaren, volcán, arcos rainbow y lasers → `CanQuery = false` y `CastShadow = false`. Zonas base (`RainZone`, `CyberpunkZone`…) → `CastShadow = false`.
- **`EffectSystems`:** 5 sistemas que crean Parts en runtime → ahora nacen con `CanQuery = false` (nombres de los 5 **UNKNOWN**; auditar los `Instance.new("Part")` de los efectos).
- **Lava (volcán):** ya estaba bien y **no se toca**. Las rocas físicas tienen `CanQuery = false`, **`CanCollide = true`** (necesario: empujan al jugador) y `CastShadow = false`. El efecto de superficie es visual. La detección de golpe usa `Magnitude`, no raycast.
- `RainbowFruitVFX`: se mantiene `CanQuery = true`.
- Las partículas (`ParticleEmitter`) de clima solo existen en 2 NPCs → irrelevante.

## P5 — Water: de 96 scripts de servidor a 1 LocalScript
- Cada `Water` tenía un **Script de servidor** con `TweenService` animando 6 texturas (12 Water × 8 jardines = 96 scripts).
- **Final:** borrar todos los scripts de dentro de los Water y crear **un único `WaterAnimator` (LocalScript en `StarterPlayerScripts`)** que anime todas las texturas de todos los jardines. Costo de servidor: cero.

## P6 — Sprinklers: `GiroScript`
- Heartbeat de rotación en el servidor → **`Script` con `RunContext = Client`** (un LocalScript dentro de workspace no se ejecuta).

## P7 — StarCube
- `FruitAnimation` (Script de servidor con `Humanoid` + `Animator` + `Motor6D` y un `while true` por cada StarCube) → **borrado**.
- **`StarCubeAnimator`** (LocalScript, cliente) detecta los modelos con attribute **`DefinitionId == "starcube"`** (**no** por tener Humanoid: otra planta futura con Humanoid se confundiría) y los anima localmente.
- Para futuras plantas animadas: un `<Nombre>Animator` propio por `DefinitionId`.

## P-MARK — Scripts "marca" (NO borrar)
- `NightfallColorAnim` (BT Nightfall Blossom, 11), `AstralVariant` (Astral Banana 4, Astral Galaxy Mushroom 4, Astral Nabo 5, Astral Petunia 3, Astral Red Rose 3) y `RainbowPartsAnim`: son Scripts **deshabilitados** que solo **marcan** qué animar. El que anima es **`NightfallBlossomAnimator`** (cliente).
- **Deben seguir existiendo y deshabilitados.** Si se borran, se pierde el efecto; si se habilitan, el animador los ignora.
- Assets nuevos con colores animados: agregar su config en `SIGNAL_CONFIG` de `NightfallBlossomAnimator` + un Script deshabilitado con ese nombre.

## P8 — Highlight por frame
- `HighlightSystem/Init` hacía un raycast sin filtro cada frame que el mouse se movía, contra todas las partes con `CanQuery = true` → **desactivado**, y `HighlightBridge` borrado en la limpieza.

## P9 — Sistemas de parcelas en segundo plano
- Borrados: `PlotPresenceSystem`, `ParcelGameplaySystem` (callbacks de XP), `ParcelGuiBridge` (3 `task.spawn` en loop), `ParcelUnlockHandler`, `ParcelGuiController` y sus GUIs.

## P10 — Varios
- `RagdollTester`: borrado (script de pruebas).
- `CanBeDropped = false` en todos los Tools (seguridad y basura en workspace).
- Validaciones de plantado, sprinkler y regadera **sobre los datos en memoria** (`PlantGrowthService`), sin consultar workspace.
- Cooldowns anti-misclick (0,2 s / 0,1 s) → evitan requests duplicados.
- Revisado y sin problema: 47 loops por frame (todos legítimos: clima, pets, UI, tools), 17 `while true` en el servidor (schedulers necesarios), 174 RemoteEvents (altos, pero probablemente necesarios).
- Revisado: `CowRainbowHighlight` (pets) ya es LocalScript. `NightfallColorAnim`/`AstralVariant`/`RainbowPartsAnim` → ver P-MARK.

## Verificación
Contar las partes con `CanQuery` y `CastShadow` antes y después; ningún Script dentro de los Water; `GiroScript.RunContext == Client`; no existe `FruitAnimation` en el StarCube; plantar, tools, huevos, NPCs y pets siguen siendo clickeables.
