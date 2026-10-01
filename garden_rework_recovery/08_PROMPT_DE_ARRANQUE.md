# 08 — PROMPT DE ARRANQUE

Adjuntar a la nueva ventana todos los archivos de esta carpeta (00, 01, 02, 02b, 02c, 03–07, 09) y pegar:

```
Carga los skills `gosa` y `go-worker-contract` antes de comenzar.

Hace unos días otro agente implementó conmigo un rework completo de Garden Odyssey en una copia DEV.
Todo funcionaba, pero Roblox Studio se cerró de golpe y se perdió todo.
Te adjunto:
- 09_ORDEN_ORIGINAL.md: la orden con la que empezó aquel trabajo.
- 00 a 07: una GUÍA DE PROCESO con el resultado FINAL de cada sistema, los valores exactos,
  los ~30 bugs que ya se resolvieron (no los repitas), las optimizaciones y lo que yo hice a mano.

CAMBIOS QUE PREVALECEN SOBRE LA ORDEN ORIGINAL:
- Implementamos directamente en el JUEGO PRINCIPAL (placeId 85407374189603), no en una copia.
  Aplica las precauciones de 00_MASTER_PLAN.md §1.
- Donde la orden y la guía difieran, manda la guía. Si es gameplay, pregúntame.
- Consola fuera de alcance.

CÓMO TRABAJAR:
- Toma el control total y sigue la ruta de 06_IMPLEMENTATION_PLAN.md.
- ANTES DE TOCAR NADA: preséntame el checklist de 03_WORLD_MANUAL_SETUP.md §0
  (qué hice yo a mano y cómo lo hice) y pregúntame qué ya está hecho.
- Respeta las reglas de ejecución de 00_MASTER_PLAN.md §3
  (relee cada edición, compila, no edites en Play, nada de tonumber en IDs, borra en vez de deshabilitar).
- Detente solo en las paradas de 00 §2. Guarda el place al cerrar cada fase.
- No inventes: lo marcado UNKNOWN lo resuelves leyendo el proyecto o preguntándome.
```
