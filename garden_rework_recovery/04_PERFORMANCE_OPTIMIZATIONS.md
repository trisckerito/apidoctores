# 04 — OPTIMIZACIONES DE RENDIMIENTO

> Implementar **directamente la versión final**. Cada optimización tiene cómo verificar que sigue presente.
> El historial **no registró** ajustes concretos de `CanQuery` / `CanTouch` / `CanCollide` en masa, de frecuencias de UI ni de replicación más allá de lo que se lista aquí. No inventar otros.

---

## P1 — Eliminar el polling de presencia (`PlotPresenceService`)

- **Problema original:** raycast cada **0,12 s** desde la cabeza de cada jugador hacia `BaseSoil`, más `GardenZone` Touched/TouchEnded. Era el **mayor consumidor de CPU del jardín** [CONFIRMADO, orden original].
- **Qué se hizo:** reemplazo por un modelo basado en eventos. El cliente envía la posición o el `slotId` en el request, y el servidor valida solo cuando hay una acción.
- **Implementación final:** sin bucles por jugador. `PlantHandler` y `ToolUseHandler` validan en el momento del request (acceso, límites, proximidad, radio).
- **NO volver a implementar:** ningún `while`/`Heartbeat`/`task.wait(0.12)` que calcule "en qué parcela o planta está el jugador". Nada de `ParcelInteractionService` ni de "active plot".
- **Dependencias:** SYS-02, SYS-03, SYS-04.
- **Verificar:** buscar `PlotPresence`, `getActivePlot`, `0.12` y `RaycastParams` en loops del servidor → 0 resultados relevantes.

## P2 — Eliminar 160 parcelas por jardín

- **Problema original:** `Garden_001` tenía ~**20.600** instancias (160 `Model` × `BaseSoil` + `PlantPivot` + `InteractionPivot` + decoración).
- **Qué se hizo:** un solo `GardenFloor` y borrado de la carpeta `Parcels`.
- **Implementación final:** `Garden_001` ≈ **1.898** instancias [CONFIRMADO].
- **NO volver a implementar:** slots físicos invisibles, grids de Parts ni "marcadores" por posición posible.
- **Verificar:** `#Garden_001:GetDescendants()` ≈ 1.898 (orden de magnitud). Ningún `BaseSoil`.

## P3 — Radio de sprinkler euclidiano (en lugar de BFS)

- **Problema original:** `ParcelDetector` hacía un BFS sobre posiciones de `BaseSoil` vecinos en grid.
- **Qué se hizo:** distancia euclidiana XZ sobre la lista de plantas activas del jardín (datos en memoria de `PlantGrowthService`, sin consultar Workspace).
- **Implementación final:** `dx*dx + dz*dz <= r*r` (sin `sqrt`), iterando solo las plantas **del jardín del sprinkler**.
- **NO volver a implementar:** `ParcelDetector`, búsquedas espaciales en Workspace (`GetPartBoundsInRadius` sobre todo el jardín) ni BFS por vecinos.
- **Verificar:** `ParcelDetector` no existe. El código del sprinkler itera los datos de plantas, no instancias.

## P4 — Validación de colocación sobre datos, no sobre Workspace

- **Problema:** con libre colocación hay que comprobar el radio mínimo de 5 studs y el límite de 200 plantas.
- **Implementación final** [INFERIDO como buena práctica coherente con P3]: el servidor itera las plantas del jardín en memoria (`PlantGrowthService`), comparando con la distancia al cuadrado. El límite de 200 se comprueba primero (corte barato).
- **NO hacer:** raycasts o `GetPartBoundsInRadius` para detectar plantas vecinas. Tampoco confiar en una pre-validación del cliente (puede existir como ayuda visual, pero el servidor decide).
- **Verificar:** la validación vive en el servidor y no consulta Workspace.

## P5 — Consola: no reintroducir la recolección periódica

- **Problema original:** `GamepadController.collectSoils()` volvía a recolectar todos los `BaseSoil` **cada 5 s**.
- **Estado final:** consola fuera de alcance (inerte).
- **NO hacer:** si en el futuro se adapta, no recolectar todas las plantas cada N segundos. Usar `ChildAdded`/`ChildRemoved` de la carpeta de plantas o un evento del servidor.
- **Verificar:** no hay nuevos loops periódicos en `GamepadController`.

## P6 — Raycast de cliente filtrado

- [INFERIDO] Al plantar, aceptar solo impactos sobre `GardenFloor` (filtro por instancia o por `CanQuery`). Para tools, aceptar solo modelos `Plant_*`. Excluir siempre el personaje.
- **NO hacer:** aceptar cualquier Part y luego buscar con `FindFirstAncestor` en bucles grandes.
- **Verificar:** el `RaycastParams` del `ToolUseController` usa una lista de filtro.

## P7 — Cartel de Garden Level dirigido por eventos

- **Implementación final** [CONFIRMADO]: el cartel "se actualiza en vivo al cosechar". Sin loop de refresco.
- **NO hacer:** `while true do ... task.wait()` para refrescar el `SurfaceGui`.
- **Verificar:** la actualización la dispara la cosecha (evento o atributo), no un temporizador.

## P8 — Migración solo una vez

- **Implementación final:** `gardenReworkMigrated = true` en el perfil evita volver a ejecutarla [CONFIRMADO].
- **NO hacer:** escanear el formato viejo en cada entrada una vez migrado.
- **Verificar:** con el flag a `true`, `GardenMigration` sale inmediatamente.

## P9 — Locks por slot

- **Implementación final:** locks en `SeedService` con clave `userId:gardenId:slotId` + deduplicación por UUID del item [CONFIRMADO como plan]. Es tanto corrección como rendimiento: evita trabajo duplicado por doble click.
- **Verificar:** dos requests idénticos simultáneos crean una sola planta.

---

### Propiedades físicas
`CanQuery` / `CanTouch` / `CanCollide` / `Anchored` de objetos concretos: solo constan las de `GardenFloor` (`Anchored = true`, `CanCollide = true`; `CanQuery = true` inferido). Cualquier otra optimización de propiedades: **UNKNOWN**, no aplicar sin evidencia.

---

## P10 — Optimizaciones FUERA del rework (mapa, iluminación, clima) — UNKNOWN / REQUIERE VERIFICACIÓN

- **Qué se sabe** [CONFIRMADO por el usuario, sin detalle]: durante el desarrollo se hicieron optimizaciones de lag fuera del alcance del rework. Por ejemplo, Parts con `CastShadow`, `CanQuery`, etc., y ajustes en sistemas de clima.
- **Qué NO se sabe:** qué objetos exactos, qué propiedades, qué valores, si fueron manuales o por script, y en qué sistemas de clima.
- **Acción obligatoria de la nueva ventana:**
  1. En el arranque (checklist `03` §0), **preguntar al usuario** por estas optimizaciones.
  2. Si la copia DEV (`86748110736040`) sigue existiendo → comparar (solo lectura) las propiedades de rendimiento (`CastShadow`, `CanQuery`, `CanTouch`, `CanCollide`, `Anchored`, iluminación, `Lighting`/clima) entre el DEV y el principal, y presentar las diferencias al usuario **antes** de aplicar nada.
  3. **No aplicar optimizaciones masivas por cuenta propia** (cambiar `CastShadow`/`CanQuery` en masa puede romper raycasts, el plantado o la estética).
- **Verificar:** lista de cambios aprobada por el usuario y aplicada. Raycast de plantado y tools sigue funcionando.
