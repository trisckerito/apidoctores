# 08 — PROMPT DE ARRANQUE (cómo entregar el paquete)

Pega en la nueva ventana, **en este orden**:

1. El bloque **"NOTA DE CAMBIOS"** de abajo.
2. Los archivos `00` a `07` y `09_ORDEN_ORIGINAL.md` (la orden original ya está guardada ahí, no hace falta volver a pegarla).

---

## NOTA DE CAMBIOS (pegar primero)

```
Carga los skills `gosa` y `go-worker-contract` antes de comenzar.

A continuación te doy:
(1) la ORDEN ORIGINAL que le di a otro agente hace unos días (09_ORDEN_ORIGINAL.md), y
(2) una GUÍA DE PROCESO (archivos 00 a 07) que resume cómo terminó ese desarrollo:
    versiones finales de cada sistema, errores que ya se resolvieron y no deben repetirse,
    optimizaciones, y lo que yo hice a mano en el mapa.

CAMBIOS QUE PREVALECEN SOBRE LA ORDEN ORIGINAL:
- Ya NO trabajamos en una copia DEV. Implementamos directamente en el JUEGO PRINCIPAL
  (placeId 85407374189603). Aplica las precauciones de 00_MASTER_PLAN.md §1
  (respaldo antes de empezar, no publicar sin mi permiso, migración con ENABLED=false
  hasta que yo la autorice, cuidado con API Services y datos reales).
- Donde la orden original y la guía difieran, manda la guía (es la versión final que ya funcionó).
  Si la diferencia es de gameplay, pregúntame.
- Además del rework de parcelas, la guía incluye el sistema Garden Level + Resonancia, que se
  añadió durante el desarrollo y también hay que implementar.
- Consola (GamepadController) queda fuera de alcance.

CÓMO QUIERO QUE TRABAJES:
- Toma el control total de la implementación usando la guía como referencia.
- ANTES DE TOCAR NADA: preséntame el checklist de 03_WORLD_MANUAL_SETUP.md §0, con la lista
  de lo que yo hice en el mapa y cómo lo hice, y pregúntame si ya lo hice. Si te digo que sí,
  verifícalo en Studio. Si te digo que no, dime exactamente cómo hacerlo.
- Después avanza por tu cuenta y detente solo en las paradas obligatorias de 00_MASTER_PLAN.md §2.
- No inventes: lo marcado UNKNOWN se resuelve leyendo el proyecto o preguntándome.
```
