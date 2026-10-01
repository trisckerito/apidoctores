# 05 — BUGS Y SOLUCIONES FINALES

> Solo se incluyen bugs y riesgos que **constan en el historial**. Cada uno indica qué implementar y qué **NO** repetir.

---

## B1 — La migración perdía todas las plantas en crecimiento
- **BUG:** la primera versión de la migración buscaba la semilla de la planta por un **campo que no existe**. Resultado: no encontraba la semilla y las plantas en crecimiento se habrían perdido sin compensación.
- **CAUSA:** se asumió un nombre de campo sin leer la estructura real. Los datos de la planta están en `properties` (el `plantInstance`).
- **SOLUCIÓN FINAL:** resolver la semilla desde la definición real de la planta (`definitionId` / datos de `properties`), leyendo cómo el proyecto relaciona planta ↔ semilla.
- **IMPLEMENTACIÓN FINAL:** antes de escribir la migración, **leer** con el MCP la definición de plantas y semillas y una entrada real de `plants[...]`. Si la búsqueda de la semilla devuelve `nil` → **no** marcar como migrado; registrar un warn y conservar la planta en los datos viejos.
- **NO HACER:** suponer nombres de campo (`seedId`, `seed`, etc.) sin verificarlos. Tampoco descartar silenciosamente una planta cuya semilla no se encuentra.

## B2 — La migración podía correr después de cargar el jardín
- **BUG:** en la versión anterior (Script en `PlayerAdded`), la migración podía ejecutarse **después** de que `PlantGrowthSystem` restaurara las plantas. Eso es una condición de carrera: datos viejos interpretados como nuevos, o plantas duplicadas o perdidas.
- **SOLUCIÓN FINAL:** `GardenMigration` es un **ModuleScript** llamado **síncronamente desde `PlantGrowthSystem.Init`**, justo **antes** de `restorePlants` de cada jugador. Además, una **protección dentro de `restorePlants`** ignora entradas con formato viejo.
- **NO HACER:** un Script independiente conectado a `PlayerAdded`, ni `task.spawn`/`task.defer` para la migración.

## B3 — Plantas a la altura equivocada tras subir el suelo
- **BUG:** las plantas se apoyaban en el `BaseSoil`. Cuando el usuario subió el `GardenFloor`, quedaron hundidas o a mala altura.
- **SOLUCIÓN FINAL:** la altura de la planta = cara superior del **`GardenFloor`** del jardín (+ el offset del modelo).
- **NO HACER:** usar `BaseSoil`, `PlantPivot` o una Y fija codificada a mano.

## B4 — Inventario lleno durante la migración
- **RIESGO resuelto en el diseño final:** si los items no caben en el inventario, se perderían.
- **SOLUCIÓN FINAL:** si algo no entra, **no** se marca `gardenReworkMigrated`, y se reintenta en la siguiente entrada.
- **NO HACER:** marcar como migrado tras una entrega parcial. Tampoco descartar el excedente.
- ⚠ UNKNOWN: cómo se evita entregar dos veces lo que **sí** entró en un intento parcial. Comprobar en el código existente. Si no está resuelto, **reportarlo al usuario** antes de activar la migración: o se hace atómica (comprobar el espacio total antes de entregar nada) o se marcan las entradas ya migradas una por una.

## B5 — Restos de parcelas en otros sistemas tras borrar `Parcels`
- **BUG:** el tutorial (beam hacia una parcela), el **sonido de los sprinklers** y la **Sweet Frog** referenciaban parcelas o `BaseSoil`.
- **SOLUCIÓN FINAL:** tutorial → `GardenFloor`. Sonido de sprinklers y Sweet Frog sin referencias a parcelas.
- **NO HACER:** borrar `Parcels` sin buscar antes referencias en **todos** los scripts (lista de búsqueda en `01` §3.3).

## B6 — `GamepadController` inerte
- **ESTADO:** no se adaptó. No da error y no encuentra nada.
- **DECISIÓN FINAL:** se deja así (update de consola futura).
- **NO HACER:** reescribirlo dentro de esta recuperación. Tampoco "arreglarlo" volviendo a crear `BaseSoil`.

## B7 — Jardines clonados con el mismo ID
- **RIESGO:** al duplicar `Garden_001`, todas las copias heredan `GardenID = 1` y `GardenId = 1`, y el sistema confunde las bases.
- **SOLUCIÓN FINAL:** cada copia con nombre único, `GardenID` del modelo único y el `GardenId` de su `GardenFloor` igual al del modelo.
- **NO HACER:** dar por buenos los clones sin verificar los IDs.

## B8 — Más jugadores que jardines
- **RIESGO:** con Max Players mayor que el número de jardines, el jugador extra entra sin jardín.
- **SOLUCIÓN FINAL:** Max Players = número de jardines.

## B9 — Valores de prueba que pueden llegar a producción
- Nivel máximo de Garden Level: el agente ofreció ponerlo temporalmente en **2**. El valor final debe ser **100**.
- Tiempos de crecimiento de plantas: anotados como **valores de prueba**; confirmar con el usuario.
- Sweet Egg: 5 h (dado como valor actual; confirmar que es el de producción).
- **NO HACER:** publicar sin revisar `GardenProgressConfig` y los tiempos.

## B10 — Precio previo a la cosecha sin bonus de resonancia (limitación aceptada)
- El prompt de la fruta en la planta muestra el precio **sin** el bonus, que se aplica al cosechar. En el inventario y al vender sí aparece.
- **No es un bug a corregir** salvo que el usuario lo pida.

## B11 — Balance de XP muy lento (pendiente de decisión)
- 1 → 100 = 1.974.456 XP con 1–8 XP por fruta, es decir, cientos de miles de cosechas por resonancia.
- **NO cambiar** sin decisión del usuario. Preguntar si quiere un multiplicador (×10, ×50) o una curva distinta.

## B12 — Prueba de la migración en el place original (datos reales)
- **RIESGOS** [CONFIRMADOS por el agente]:
  - Sin "Enable Studio Access to API Services", Studio carga un perfil vacío y la prueba no sirve.
  - La migración **modifica y guarda** el perfil real. Es irreversible para esa cuenta.
  - Para repetirla hay que borrar a mano `gardenReworkMigrated` del perfil.
  - Si la misma cuenta está conectada al juego publicado, el perfil puede estar **bloqueado** por esa sesión. Hay que salir antes.
- **NO HACER:** activar la prueba sin la confirmación explícita del usuario.

## B13 — Migración nunca probada con datos reales
- El DEV no tenía datos con formato de parcelas, así que la migración final **no se probó**.
- **Recomendación final del agente:** verificar con datos de formato viejo **antes** de publicar.
- **Ahora (juego principal):** los datos viejos reales existen. La prueba se hace en la Sección 12 con la cuenta del usuario, tras su confirmación. Antes de activarla, revisar el código de la migración con una entrada real leída (solo lectura) del perfil.

---

## Resumen: implementaciones históricas que NO deben volver

| No usar | Usar en su lugar |
|---|---|
| Migración como Script en `PlayerAdded` | ModuleScript llamado desde `PlantGrowthSystem.Init` antes de `restorePlants` |
| Buscar la semilla por un campo supuesto | Leer la relación real planta → semilla desde las definiciones |
| Altura desde `BaseSoil` | Cara superior de `GardenFloor` |
| `PlotPresenceService` (raycast 0,12 s) | Validación en el request |
| `ParcelInteractionService` / `getActivePlot` | Posición o `slotId` en el request |
| `ParcelDetector` (BFS) | Radio euclidiano XZ |
| `canAccessPlot` | `canAccessGarden` |
| `plotId` numérico | `slotId` UUID + `x`, `z` |
| Reescribir `GamepadController` | Dejarlo inerte (fuera de alcance) |
