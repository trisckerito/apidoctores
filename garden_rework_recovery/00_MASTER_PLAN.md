# 00 — GUÍA MAESTRA DE PROCESO
## Garden Odyssey — Rework "160 Parcelas → Jardín Libre" + Garden Level / Resonancia + Optimizaciones

> **Qué es este paquete:** una **guía de proceso** construida a partir de la conversación completa (~2 días) con el agente anterior.
> En aquel desarrollo todo quedó funcionando en una copia DEV, pero **un cierre forzado de Roblox Studio borró todo el trabajo**.
> Este paquete permite **rehacerlo yendo directo a la versión final**, sin recorrer los ~30 bugs que ya se resolvieron.
>
> **No contiene el código fuente**: el agente anterior editaba los scripts directamente en Studio y el código nunca quedó en el chat.
> Contiene la **especificación final** de cada sistema, valores exactos, decisiones del usuario, bugs ya resueltos y cómo verificar.

---

## 1. Cambio fundamental respecto a la orden original (`09_ORDEN_ORIGINAL.md`)

| Orden original | **AHORA (prevalece)** |
|---|---|
| Trabajar en una copia DEV y portar después | Implementar **directamente en el JUEGO PRINCIPAL** (placeId `85407374189603`) |
| Radio mínimo entre plantas: 5 studs | **1 stud** (decisión del usuario) |
| Consola dentro del alcance | **Fuera de alcance** (update aparte al final) |
| Migración como parche en `PlayerAdded` | **ModuleScript llamado desde `PlantGrowthSystem.Init`** (ver `05` B-MIG) |
| Sin Garden Level | **Garden Level 1–100 + Resonancia** (reemplaza la XP por parcela) |
| Sin huevos en el alcance | **Huevos en soportes `EggPlaced`** (rework incluido) |
| — | **Optimizaciones globales de lag** (CanQuery, CastShadow, clima, agua, StarCube…) |

**Precauciones obligatorias por trabajar en el juego principal:**
1. **Antes de tocar nada**, el usuario guarda una copia: *File → Save to File As* (`.rbxl` local) **y** confirma que existe una versión en el historial de versiones del place. Con esto se evita que un cierre forzado de Studio vuelva a destruir el trabajo.
2. **Guardar a menudo** durante la implementación (*Ctrl+S* / *Save to Roblox*), y nunca editar scripts con el juego en **Play** (los cambios hechos en Play no se guardan).
3. **No publicar** hasta terminar y validar todo, y solo con la autorización del usuario.
4. **Datos reales:** con *Enable Studio Access to API Services* activado, cada Play usa el DataStore de producción. `GardenMigration` se crea con **`ENABLED = false`** y solo se activa cuando el usuario lo autorice (Sección 13 de `06`). Preguntar si quiere desactivar API Services mientras se desarrolla.
5. En el juego principal están **los 8 jardines originales con parcelas**. Toda la parte manual del DEV (estilo del suelo, soportes de huevos, cartel, agua) **hay que rehacerla** (`03`).

---

## 2. Modo de trabajo: CONTROL TOTAL con paradas fijas

La nueva ventana **toma el control**: decide el orden fino, los nombres internos y la forma del código siguiendo `gosa`, `go-worker-contract` y las convenciones del proyecto. `06_IMPLEMENTATION_PLAN.md` es la **ruta recomendada**, no un guion rígido.

**Paradas OBLIGATORIAS (únicas):**
1. **Al inicio:** presentar el checklist de `03_WORLD_MANUAL_SETUP.md` §0 (qué hizo el usuario y **cómo** lo hizo antes) y preguntar si ya está. Si responde que sí → verificar con el MCP. Si responde que no → dar instrucciones.
2. Cuando una fase necesite un objeto manual que todavía no existe.
3. **Decisiones de gameplay** que no estén en una config.
4. **Acciones destructivas** (borrar parcelas, sistemas, scripts u objetos) → confirmar.
5. **Activar la migración** o cualquier prueba con datos reales.
6. **Publicar.**

Fuera de esas paradas: avanzar, validar cada fase con `07` y reportar en bloques.

---

## 3. Reglas de ejecución aprendidas (OBLIGATORIAS — causaron la mitad de los bugs)

1. **Verificar cada edición después de hacerla.** En el desarrollo anterior, varios reemplazos con `gsub` / *MultiEdit* **fallaron en silencio** (saltos de línea dobles, caracteres especiales). Se dieron por hechos y rompieron cosas días después. Después de cada cambio: **releer el fragmento editado** y confirmar que el texto nuevo está.
2. **Chequeo de sintaxis** (compilar con `loadstring`) de cada script modificado antes de dar por cerrada una tarea. Al final, chequear **todos** los scripts del juego.
3. **Nunca editar con el juego en Play.** Detener el Play antes de escribir código.
4. **Al renombrar `plotId` → `slotId`**, buscar **todas** las referencias al nombre viejo en ese scope (varias quedaron como `nil`).
5. **Prohibido `tonumber()` sobre IDs de planta.** Los `slotId` son UUID string; `tonumber` devuelve `nil`. Fue la causa de al menos 8 bugs.
6. **Eliminar, no deshabilitar.** El usuario quiere el código muerto **borrado** del DataModel, no con `Disabled = true`. Excepción: los scripts "marca" deshabilitados (`NightfallColorAnim`, `AstralVariant`, `RainbowPartsAnim`) **deben seguir existiendo y deshabilitados** (ver `04` P-MARK).
7. **Después de borrar algo**, buscar referencias en todos los scripts. Un `pcall(function() return game.ServerScriptService.X end)` **no** protege si el índice falla antes; usar `FindFirstChild`.
8. **Quitar las trazas de debug** (`print` de diagnóstico) al terminar cada arreglo.
9. **No duplicar RemoteEvents.** Antes de crear uno, buscar si existe. Al final, revisar que no haya nombres repetidos en `ReplicatedStorage`.
10. **Remotes que el cliente espera al arrancar** deben existir desde el inicio (creados en Studio o por el servidor antes de que el cliente agote su `WaitForChild`).
11. **El servidor es autoritativo:** la madurez, la distancia, la posición y el consumo de items los decide el servidor.

---

## 4. Archivos del paquete

| Archivo | Uso |
|---|---|
| `08_PROMPT_DE_ARRANQUE.md` | Cómo entregar el paquete a la nueva ventana |
| `00_MASTER_PLAN.md` | Este documento |
| `01_PROJECT_STATE.md` | Estado final alcanzado + qué esperar en el juego principal |
| `02_SYSTEMS.md` | Núcleo: suelo, plantado, datos, plantas, frutas, timers |
| `02b_SYSTEMS_TOOLS.md` | Regaderas, sprinklers, pala, trowel, extractor, Plant item, amuletos, cooldowns |
| `02c_SYSTEMS_EGGS_PETS_LEVEL_MIGRATION.md` | Huevos, pets, Garden Level/Resonancia, migración, limpieza |
| `03_WORLD_MANUAL_SETUP.md` | Parte manual del usuario + checklist de inicio |
| `04_PERFORMANCE_OPTIMIZATIONS.md` | Todas las optimizaciones a reaplicar |
| `05_BUGS_AND_FINAL_SOLUTIONS.md` | Bugs resueltos: qué NO repetir |
| `06_IMPLEMENTATION_PLAN.md` | Ruta por fases |
| `07_VALIDATION_CHECKLIST.md` | Verificación por fase |
| `09_ORDEN_ORIGINAL.md` | Orden original (histórica; prevalece esta guía) |

## 5. Places
| Place | Dato |
|---|---|
| **Juego principal (DESTINO)** | placeId `85407374189603`, Studio ID `cd0b5411-5667-4d94-b775-19170f9f529d` |
| Copia DEV anterior (perdida) | `09282026_3`, placeId `86748110736040`. Comprobar si alguna versión guardada sobrevivió: si existe, los objetos y scripts pueden **copiarse** desde ahí en lugar de rehacerse |

## 6. Etiquetas
- **[CONFIRMADO]** — dicho explícitamente en la conversación.
- **[INFERIDO]** — deducido; verificar en el proyecto.
- **UNKNOWN / REQUIERE VERIFICACIÓN** — no consta; leer el proyecto o preguntar.
