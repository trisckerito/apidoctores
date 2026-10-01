# 00 — GUÍA MAESTRA DE PROCESO
## Garden Odyssey — Rework "160 Parcelas → Jardín de Libre Colocación" + Garden Level / Resonancia

> **Qué es este paquete:** una **GUÍA DE PROCESO** construida a partir de ~2 días de desarrollo previo con otro agente.
> Recoge cómo terminó cada sistema (versión final), qué errores se cometieron y cómo se resolvieron, qué optimizaciones quedaron y qué hizo el usuario a mano en el mapa.
>
> **Qué NO es:** no es código. El agente anterior escribía los scripts directamente en Studio, así que el código no quedó en el historial. La nueva ventana **implementa por su cuenta**, usando esta guía como referencia para llegar directo a la versión final sin repetir errores.

---

## 1. Cambio fundamental respecto a la orden original

| Orden original | **AHORA (prevalece)** |
|---|---|
| Implementar en una **copia DEV**; "no preocuparse por romper el juego en vivo" | Implementar **directamente en el JUEGO PRINCIPAL** (placeId `85407374189603`) |
| Portar luego manualmente al original | No hay porte: se trabaja en el principal |

**Consecuencias obligatorias de trabajar en el juego principal:**
1. **Antes de empezar**, pedir al usuario que guarde o publique una versión de respaldo. Roblox conserva el historial de versiones del place (Creator Hub → place → Version History), y así siempre se puede volver atrás.
2. **No publicar** el place hasta terminar y validar todo. Trabajar en Studio, guardar con normalidad y publicar solo con la autorización del usuario.
3. **Migración y datos reales:** si Studio tiene activado *Enable Studio Access to API Services*, cada Play en Studio usa el **DataStore real**. Por eso:
   - `GardenMigration` se crea con `ENABLED = false` y solo se activa cuando el usuario lo autorice (ver `06`, fase de migración).
   - Cualquier Play antes de eso puede leer o guardar los perfiles reales con el código nuevo. **Preguntar al usuario** si quiere desactivar API Services mientras se desarrolla (recomendado) y activarlo solo para la prueba final.
4. **No borrar las parcelas ni los sistemas viejos** hasta que lo nuevo funcione (la regla original sigue vigente y ahora es más importante).
5. El juego principal tiene **los jardines originales con sus parcelas** (probablemente `Garden_001`…`Garden_008`). Lo que el usuario hizo en el DEV (suelo, cartel, soportes, etc.) **hay que rehacerlo o traerlo** al principal. Ver `03_WORLD_MANUAL_SETUP.md`.

---

## 2. Modo de trabajo: CONTROL TOTAL con puntos de parada fijos

La nueva ventana **toma el control de la implementación**:
- decide el orden fino, los nombres internos y la forma del código, siguiendo `gosa`, `go-worker-contract` y las convenciones del proyecto;
- puede encadenar fases sin pedir permiso entre cada una;
- usa `06_IMPLEMENTATION_PLAN.md` como **ruta recomendada** (respeta las dependencias), no como un guion rígido.

**Paradas OBLIGATORIAS (únicas):**
1. **Al inicio, antes de tocar nada:** presentar al usuario el **checklist de su parte del mapa** (`03_WORLD_MANUAL_SETUP.md` §0), explicando **qué hizo y cómo lo hizo** en el desarrollo anterior, y preguntar si ya está hecho. Si dice que sí → verificarlo con el MCP. Si dice que no → darle instrucciones.
2. **Decisiones de gameplay** que no estén en ninguna config (radios de sprinkler, balance de XP, etc.).
3. **Ambigüedades** marcadas `UNKNOWN / REQUIERE VERIFICACIÓN` que no se resuelvan leyendo el proyecto.
4. **Acciones destructivas:** borrar parcelas, sistemas o scripts.
5. **Activar la migración** (`ENABLED = true`) o cualquier prueba que toque datos reales.
6. **Publicar.**

Fuera de esas paradas, avanzar y reportar en bloques (formato de `go-worker-contract`).

---

## 3. Reglas que no cambian

1. Cargar los skills **`gosa`** y **`go-worker-contract`** antes de empezar.
2. **Implementar siempre la versión FINAL** (`02_SYSTEMS.md`), nunca la versión 1 histórica (`05_BUGS_AND_FINAL_SOLUTIONS.md`).
3. **NO INVENTAR.** Un `UNKNOWN` se resuelve leyendo el proyecto con el MCP o preguntando.
4. **No sustituir por código lo que el usuario hace a mano** (`03`). Si falta, se le pide.
5. **Gameplay pertenece al usuario.**
6. **Consola (`GamepadController`) fuera de alcance**, igual que en el desarrollo anterior.
7. Validar con `07_VALIDATION_CHECKLIST.md` antes de dar por cerrada una fase.

---

## 4. Archivos del paquete

| Archivo | Uso |
|---|---|
| `08_PROMPT_DE_ARRANQUE.md` | **Cómo entregar el paquete** a la nueva ventana (orden original + nota de cambios) |
| `00_MASTER_PLAN.md` | Este documento: modo de trabajo y reglas |
| `01_PROJECT_STATE.md` | Estado final alcanzado en el DEV y auditoría del juego principal |
| `02_SYSTEMS.md` | Versión final de cada sistema |
| `03_WORLD_MANUAL_SETUP.md` | **Parte del usuario en el mapa** + checklist de inicio |
| `04_PERFORMANCE_OPTIMIZATIONS.md` | Optimizaciones a conservar |
| `05_BUGS_AND_FINAL_SOLUTIONS.md` | Errores ya resueltos: qué no repetir |
| `06_IMPLEMENTATION_PLAN.md` | Ruta recomendada por fases |
| `07_VALIDATION_CHECKLIST.md` | Verificación por fase |
| `09_ORDEN_ORIGINAL.md` | Orden original dada al agente anterior (referencia histórica; prevalece esta guía) |

---

## 5. Places

| Place | Dato |
|---|---|
| **Juego principal (DESTINO)** | placeId `85407374189603`, Studio ID `cd0b5411-5667-4d94-b775-19170f9f529d` |
| Copia DEV anterior (solo referencia) | nombre `09282026_3`, placeId `86748110736040` |

**Atajo posible:** si la copia DEV sigue existiendo, los objetos que el usuario creó allí (`GardenFloor`, `GardenLevel`, soportes de huevos, carpetas) se pueden **copiar y pegar** entre los dos Studios, y los scripts finales también. Preguntar al usuario si el DEV sigue disponible antes de reconstruir nada desde cero.

---

## 6. Glosario

| Término | Significado final |
|---|---|
| `GardenFloor` | Part único, plano, grande y anclado por jardín. Attribute `GardenId`. |
| `slotId` | UUID string generado al plantar. Sustituye a `plotId`. |
| `plotId` | Clave vieja 1..160. Solo la lee la migración. |
| Parcela / `BaseSoil` | Sistema viejo. Desaparece. |
| Garden Level | Nivel 1..100 del jardín. Sube con XP por fruta cosechada. |
| Resonancia | En nivel 100: pagar 1.000.000 Odyssey Coins → nivel 1 + 1 resonancia (+0,05 % al valor de las frutas por nivel, máx. 50 %). |
| Migración | `GardenMigration`: convierte las plantas viejas en items, una vez por jugador. |

## 7. Etiquetas
- **[CONFIRMADO]** — dicho explícitamente en el historial.
- **[INFERIDO]** — deducido; verificar en el proyecto.
- **UNKNOWN / REQUIERE VERIFICACIÓN** — no consta. Leer el proyecto o preguntar.
