# 00 — PLAN MAESTRO DE RECUPERACIÓN
## Garden Odyssey — Rework "160 Parcelas → Jardín de Libre Colocación" + Garden Level / Resonancia

> Paquete de recuperación generado a partir del historial de ~2 días de desarrollo.
> Destinatario: **otra ventana de Claude** que reconstruirá la actualización **por secciones**.
> Este paquete **no contiene código fuente recuperado**: el agente original escribía los scripts directamente en Roblox Studio (herramienta *Execute Luau* del MCP de Studio), así que el código final nunca quedó en el chat. Lo que sí quedó son **especificaciones, decisiones finales, bugs corregidos y verificaciones**. Con eso se reimplementa la **versión final** sin recorrer las versiones fallidas.

---

## 1. Cómo usar este paquete (léelo entero antes de tocar nada)

| Archivo | Para qué sirve | Cuándo leerlo |
|---|---|---|
| `00_MASTER_PLAN.md` | Reglas globales, glosario, protocolo de trabajo | Primero, siempre |
| `01_PROJECT_STATE.md` | Estado final alcanzado + auditoría a ejecutar sobre el proyecto actual | Antes de la Sección 0 |
| `02_SYSTEMS.md` | Especificación final de cada sistema | Al implementar cada sección |
| `03_WORLD_MANUAL_SETUP.md` | Todo lo que **el usuario** crea a mano en Studio | Cada vez que una sección lo requiera → **DETENERSE Y PREGUNTAR** |
| `04_PERFORMANCE_OPTIMIZATIONS.md` | Optimizaciones que deben quedar presentes | Al implementar y al validar |
| `05_BUGS_AND_FINAL_SOLUTIONS.md` | Bugs históricos y su solución final | Antes de cada sección (sección "NO HACER") |
| `06_IMPLEMENTATION_PLAN.md` | **EL DOCUMENTO PRINCIPAL**: orden de secciones y pasos | Guía de trabajo |
| `07_VALIDATION_CHECKLIST.md` | Checklist de verificación por sección | Al cerrar cada sección |

---

## 2. Reglas globales (NO NEGOCIABLES)

1. **Cargar los skills `gosa` y `go-worker-contract` antes de empezar** (así lo exigía la orden original).
2. **Una sección a la vez.** Nunca implementar varios sistemas grandes de golpe.
3. **Protocolo por sección** (obligatorio, en este orden):
   1. Leer la sección en `06_IMPLEMENTATION_PLAN.md`.
   2. Revisar el estado actual del proyecto (MCP de Studio: leer, no escribir).
   3. Confirmar dependencias (secciones previas validadas).
   4. Comprobar objetos manuales (`03_WORLD_MANUAL_SETUP.md`).
   5. **Si falta algo manual → DETENERSE Y PREGUNTAR** (ver §4).
   6. Implementar **solo** esa sección.
   7. Verificar con `07_VALIDATION_CHECKLIST.md`.
   8. Reportar qué se hizo (formato de `go-worker-contract`).
   9. **Esperar confirmación del usuario.**
   10. Pasar a la siguiente sección.
4. **Implementar SIEMPRE la versión FINAL** descrita en `02_SYSTEMS.md`. Nunca la "versión 1" histórica. Ver `05_BUGS_AND_FINAL_SOLUTIONS.md`.
5. **NO INVENTAR.** Todo lo marcado `UNKNOWN / REQUIERE VERIFICACIÓN` se resuelve:
   - primero **leyendo el proyecto** con el MCP (puede que el sistema ya exista),
   - y si no se puede, **preguntando al usuario**. Nunca rellenando con suposiciones.
6. **NO sustituir estructuras manuales por código.** Si `GardenFloor`, `GardenLevel`, `Eggs`, `Sprinklers`, etc. no existen, se le pide al usuario que los cree (o confirme que se creen por código). No se generan "por si acaso".
7. **No borrar sistemas viejos hasta que el nuevo esté probado** (regla de la orden original). La limpieza es la penúltima sección.
8. **Gameplay pertenece al usuario.** No cambiar balance (XP, precios, radios, tiempos) sin preguntar.
9. **Consola (GamepadController) queda FUERA DE ALCANCE.** Decisión final del desarrollo original: se pospuso para una "update de consola". No reescribirlo.
10. **Antes de una acción destructiva** (borrar parcelas, borrar scripts, activar la migración sobre datos reales) → pedir confirmación explícita.

---

## 3. Contexto de los places

| Place | Dato | Fuente |
|---|---|---|
| Juego original (producción) | placeId `85407374189603`, Studio ID `cd0b5411-5667-4d94-b775-19170f9f529d` | Orden original |
| Copia DEV donde se hizo el trabajo | nombre `09282026_3`, placeId `86748110736040` | Primer reporte del agente |

- El rework se hizo **en el DEV**. El plan era portarlo luego **manualmente** al original.
- Al final del desarrollo el DEV tenía **solo `Garden_001`** (el usuario borró `Garden_002`…`Garden_008` para clonar luego `Garden_001`).
- **UNKNOWN / REQUIERE VERIFICACIÓN:** si el place DEV `86748110736040` sigue existiendo con el trabajo hecho. **Es lo primero que se audita (Sección 0).** Si existe, gran parte de la "reconstrucción" pasa a ser "verificar y portar".

---

## 4. Plantilla obligatoria de parada por objetos manuales

Cuando una sección necesite objetos de `03_WORLD_MANUAL_SETUP.md`, la ventana debe escribir algo equivalente a:

> **Antes de continuar necesito que reconstruyas manualmente los siguientes objetos:**
> - [lista exacta: nombre, tipo, parent, attributes]
>
> **¿Ya creaste estos objetos en Roblox Studio?**

- Si el usuario responde **NO** → darle instrucciones paso a paso (desde `03_WORLD_MANUAL_SETUP.md`), y volver a preguntar.
- Si responde **SÍ** → **verificar con el MCP** (leer jerarquía, nombres, attributes) antes de continuar. Si algo no coincide, reportarlo y detenerse.

---

## 5. Glosario

| Término | Significado final |
|---|---|
| `GardenFloor` | Part único, plano, grande, anclado, por jardín. Superficie donde se planta libremente. Attribute `GardenId`. |
| `slotId` | UUID string generado al plantar. Nueva clave de posición/datos de una planta. Sustituye a `plotId` (1..160). |
| `plotId` | Clave vieja (número 1..160). **Desaparece**. Solo la lee la migración. |
| Parcela / `BaseSoil` | Sistema viejo (160 `Model` por jardín en carpeta `Parcels`). **Desaparece**. |
| Garden Level | Nivel del jardín 1..100 que sube con XP por fruta cosechada. |
| Resonancia | Prestigio del jardín: en nivel 100, pagar 1.000.000 Odyssey Coins → vuelve a nivel 1 y suma +1 resonancia (+0,05 % al valor de venta de frutas por nivel, máx. 50 %). |
| Migración | `GardenMigration` (ModuleScript): convierte las plantas del formato viejo (parcelas) en items de inventario, una vez por jugador. |

---

## 6. Etiquetas usadas en el paquete

- **[CONFIRMADO]** — dicho explícitamente por el agente en un reporte final o por el usuario.
- **[INFERIDO]** — deducido del historial con alta probabilidad; verificar en el proyecto antes de depender de ello.
- **UNKNOWN / REQUIERE VERIFICACIÓN** — el historial no lo permite determinar. Leer el proyecto o preguntar.

---

## 7. Prompt de arranque sugerido para la nueva ventana

```
Carga los skills `gosa` y `go-worker-contract`.
Te adjunto el paquete de recuperación (00 a 07). Léelo completo.
Empieza SOLO por la Sección 0 de 06_IMPLEMENTATION_PLAN.md (auditoría de solo lectura).
No modifiques nada hasta que yo confirme el reporte de la Sección 0.
Trabaja una sección a la vez y detente al final de cada una.
Cuando una sección necesite objetos manuales, detente y pregúntame según 00_MASTER_PLAN.md §4.
```
