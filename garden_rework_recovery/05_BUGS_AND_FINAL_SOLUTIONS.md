# 05 — BUGS RESUELTOS: QUÉ NO REPETIR

> Formato: **BUG** → **CAUSA** → **SOLUCIÓN FINAL** → **NO HACER**. Implementar directamente la solución final.

## A. Proceso de edición (raíz de muchos bugs)
| ID | Bug | Causa | Solución final / NO hacer |
|---|---|---|---|
| A1 | Cambios "hechos" que no estaban en el código (`handlePlantToolClick` inexistente → `attempt to call a nil value` en `ToolUseController:288` y `:368`; `localX/localZ` nunca calculados; la conversión local→world nunca aplicada) | `gsub`/MultiEdit fallaron en silencio (archivos con `\n` doble, caracteres especiales) | **Releer después de cada edición.** No dar por hecho un reemplazo sin verificarlo |
| A2 | Errores de sintaxis (`end)` huérfano en PlantHandler, `endend` en ToolUseController, `end` sobrante en PlantVisualService) | Reemplazos de bloques largos | Editar en bloques pequeños + compilar cada script modificado |
| A3 | Código escrito durante Play que se perdió | Studio no guarda lo editado en Play | Detener el Play antes de editar |
| A4 | Trazas de debug olvidadas | — | Quitarlas al cerrar cada arreglo |

## B. `plotId` → `slotId` (UUID)
| ID | Bug | Solución final |
|---|---|---|
| B1 | Modelo de planta bugueado (sin crecimiento, prompt ni GUI) | `_makeKey(gardenId, plotId)` usaba el parámetro renombrado (nil) → usar `slotId`. Restore: parsear la clave con `find(":")`, no con `":(%d+)$"` |
| B2 | Las frutas nunca aparecían | `HarvestService._scheduleFruit` enviaba `tonumber(plotId) or 0` → clave `gardenId:0`. Pasar el slotId tal cual |
| B3 | Frutas no restauradas al reconectar | `restoreFruits` con `if not numPlotId then continue end` → aceptar el string |
| B4 | Sin boost de crecimiento | `getGrowthRate(gardenId, tonumber(plotId) or 0)` → pasar el slotId |
| B5 | Cosecha bloqueada | `type(plotId) ~= "number"` en `FruitHarvestHandler`/`FruitSkipHandler`/`FruitHarvestController`; el controller leía `PlotId` → leer `SlotId` |
| B6 | `canAccessPlot` fallaba con UUID | Usar `canAccessGarden` |
| B7 | Regadera/sprinkler sin plantas en el radio | `PlantGrowthService._parseKey` hacía `tonumber` → `getGardenPlants` devolvía 0. Sin `tonumber` |
| B8 | El timer de las frutas no se aceleraba en vivo | `reSyncFruitTimers` armaba la clave con `slotId` inexistente (`"1:nil"`) |
| B9 | Sync de frutas al entrar un jugador | 4 `tonumber` en `PlantVisualService` → eliminados |
| B10 | Pets Sweet Frog/Dog/Nebula no aplicaban el efecto | Usaban `plantState.plotId` + `getPlotPos` → `slotId` + posición del modelo o `instance.x/z` |
| **Regla** | **Prohibido `tonumber` sobre IDs de planta** | Buscar `tonumber(` en todos los servicios del flujo |

## C. Mundo y coordenadas
| ID | Bug | Solución final |
|---|---|---|
| C1 | No se podía clickear el `GardenFloor` | Quedaba bajo los `BaseSoil` → cara superior +0,05 sobre su tope y `BaseSoil.CanQuery = false` |
| C2 | Al reconectar en otro jardín, las plantas aparecían en el jardín anterior | Guardar `localX/localZ` relativos al `CFrame` del **`GardenFloor`** (no del Water, que es cosmético) y convertir al restaurar; auto-reparar las plantas sin posición local |
| C3 | Plantas hundidas | Apoyarlas en la cara superior del `GardenFloor`, no en el `BaseSoil` |
| C4 | Agua replicada por código, orientada al revés en Garden_003/005 | Rotaciones distintas entre jardines (−44° vs +45°). **No replicar decoración por código**: el usuario clona `Garden_001` a mano |
| C5 | No se podía cambiar el `MeshId` del Water en runtime | Clonar el Part de referencia |
| C6 | Clones con el mismo `GardenID` | IDs únicos en el modelo y en el suelo |
| C7 | Más jugadores que jardines | Max Players = número de jardines |

## D. Frutas y timers
| ID | Bug | Solución final |
|---|---|---|
| D1 | Single-harvest: el prompt de cosecha solo aparecía al reconectar; después, el timer se quedaba en "0m 0s" | **No** parchear con `task.delay(maturePlant)` ni saltar el timer. **Crear el runtime de la fruta al plantar** con `rAt = now + growthTime` y `_scheduleFruit` (idéntico a las multi-harvest). Se descartaron 4 intentos fallidos y su código muerto (`isInstant` reveal-retry, atributos duplicados en `_createPlantVisual`, retry de `_completeFruitVisual`) |
| D2 | Con la regadera Cosmic, la fruta mostraba peso y precio pero no se podía cosechar (faltaban 282 s) | Progreso visual con doble conteo de la velocidad. **La madurez la decide solo el servidor** (`regrowAt = nil`); progreso = total − restante |
| D3 | Dos regaderas seguidas congelaban el timer y la animación | Mismo doble conteo en el aviso de cambio de velocidad → calcular desde el `regrowAt` del servidor |
| D4 | Las regaderas a veces no aceleraban el visual | El listener de `PlantEffectService` se registraba una sola vez sin esperar → **esperar hasta 30 s** |
| D5 | El prompt de timer tenía una E que abría la compra por Robux | Primero se quitó la tecla, y el prompt desapareció (un ProximityPrompt necesita tecla para mostrarse). **Final:** prompt deshabilitado + BillboardGui propio |
| D6 | Varios timers a la vez = desorden visual | Mostrar solo el más cercano a ≤ 12 studs |
| D7 | El texto era enorme en móvil | Tamaño = 4,8 % del alto de la pantalla, entre 28 y 56 |
| D8 | El log `[ParcelGameplayService] addHarvestXP … callbacks: 0` | Llamadas muertas → eliminadas (reemplazadas por Garden Level) |

## E. Tools
| ID | Bug | Solución final |
|---|---|---|
| E1 | La regadera nunca se consumía | **RemoteEvent `WaterComplete` duplicado** → borrar el duplicado y revisar que no haya nombres repetidos |
| E2 | El VFX del agua usaba un Model como Part | Ya no aplica: VFX de área desde el punto, dentro de un `pcall` |
| E3 | Los tools no detectaban las plantas | Las partes de las plantas tenían `CanQuery = false` (por la optimización masiva) → `true` en las plantas |
| E4 | Pala, trowel y extractor no detectaban las multi-harvest | Faltaba el attribute `GardenId` en esos modelos |
| E5 | Sprinkler: las plantas nuevas no recibían el efecto | Se aplicaba una sola vez → zona activa con `applyEffectWithRemaining` |
| E6 | Sonido y limpieza de sprinklers buscaban en el `BaseSoil` | Carpeta `Sprinklers` por jardín |
| E7 | El sprinkler no giraba | Un LocalScript en workspace no corre → `Script` con `RunContext = Client` |
| E8 | La pala tardaba 9 s y pisaba la velocidad del jugador | 2 s en loop; sin `WalkSpeed = 16` / `JumpPower = 50` |
| E9 | Trowel: el click de destino fallaba cerca de otras plantas; se podían apilar plantas; soltaba la planta sin explicación | El raycast excluye las plantas; validar jardín + 1 stud; si falla, mantener la planta en la mano |
| E10 | Trowel: cancelar mientras esperaba respuesta bloqueaba el siguiente uso | Limpiar la marca al cancelar |
| E11 | El extractor no extraía si no cargaba la animación | Enviar siempre el aviso final |
| E12 | Misclicks gastaban usos | Cooldown de 0,2 s (varita 0,1 s), comprobado **antes** de cualquier estado de espera |

## F. Huevos y pets
| ID | Bug | Solución final |
|---|---|---|
| F1 | El hatch se podía hacer desde cualquier distancia | El servidor valida 9 studs horizontales (+1) |
| F2 | El aviso de rechazo nunca se mostraba (`reason:nil`, ×18 rechazos) | Orden de argumentos `uuid, ok, animIndex, soilCF, reason` + cooldown de 0,2 s antes del flag |
| F3 | El skip dejaba un tamaño final incorrecto | Calcular desde la escala inicial |
| F4 | El prompt no respetaba la posición movida en el asset | Engancharlo al `Attachment "ProximityPrompt"` del asset |
| F5 | El timer de incubación se veía desde 25 studs | 9 studs |
| F6 | El aviso "red stands" salía a cada rato | El click del OK atravesaba la ventana → bloquear clicks mientras está abierto + 0,3 s después |
| F7 | Lucky Block: dos huevos superpuestos | Si el soporte está ocupado, usar el primer soporte libre |
| F8 | El highlight de los pets no se veía (el anuncio sí) | El cliente esperaba los remotes 10 s y el servidor los creaba después → los remotes existen desde el arranque (highlight, sonido de habilidad, Lucky Block, Bunny) |
| F9 | "El tiempo del huevo sube de 2 s a 1 h" (Lunaris) | **No era un bug**: era el timer de otro huevo cercano. No tocar la lógica de Lunaris |
| F10 | Dos soportes con el mismo `EggSlot` al duplicar | Renumerar después de cualquier cambio de soportes |

## G. Limpieza y referencias
| ID | Bug | Solución final |
|---|---|---|
| G1 | Las plantas desaparecían después de borrar sistemas | `GardenService` (línea ~84) referenciaba `PlotPresenceSystem`, y `PlantGrowthService._resolveServices` referenciaba `ParcelGameplaySystem` (el `pcall` no protegía el índice) → quitar las referencias y usar `FindFirstChild` |
| G2 | `ParcelInteractionService` hacía `require` de `PlotPresenceService` borrado | Quitar `_activePlot` y la dependencia |
| G3 | Usuario: "¿desactivaste o eliminaste?" | Borrar del DataModel (los scripts de servidor deshabilitados no son explotables, pero son código muerto) |
| G4 | `DeletePlantHandler` viejo | Borrado junto con sus 2 remotes |

## H. Migración
| ID | Bug | Solución final |
|---|---|---|
| H1 | La semilla se buscaba por un campo que no existe → se perdían las plantas en crecimiento | Relación real planta → semilla desde las definiciones; si es `nil`, no marcar como migrado |
| H2 | La migración podía correr después de cargar el jardín | ModuleScript llamado desde `PlantGrowthSystem.Init` antes de `restorePlants` |
| H3 | Inventario lleno | No marcar como migrado; reintentar en la próxima entrada |
| H4 | Prueba con datos reales | Irreversible para esa cuenta; para repetirla, borrar `gardenReworkMigrated` a mano. Requiere API Services y no estar conectado al juego publicado |

## I. Decisiones que NO son bugs (no "arreglar")
- Las single-harvest no se pueden mover ni extraer.
- Las frutas se reinician al mover un árbol con el trowel.
- El precio del prompt antes de cosechar no incluye la resonancia.
- La compra de "madurar fruta al instante" quedó sin acceso (el handler se conserva).
- `GamepadController` inerte.
- 2 sprinklers del mismo tier no suman velocidad (solo renuevan la duración).
