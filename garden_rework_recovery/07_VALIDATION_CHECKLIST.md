# 07 — CHECKLIST DE VALIDACIÓN

**Siempre (en cada fase):** ediciones releídas · scripts compilados · Output sin errores ni warnings nuevos · sin debug prints · sin RemoteEvents duplicados · place guardado · ningún bug de `05` reintroducido.

## F0
- [ ] Place `85407374189603` · respaldo `.rbxl` + versión · checklist `03` §0 respondido · tabla `01` §4 rellenada · nada modificado.

## F1
- [ ] `GardenFloor` en cada jardín (Part, Anchored, CanCollide, **CanQuery**, sin sombra, `GardenId`) · el click lo alcanza (no lo tapan los `BaseSoil`).

## F2 — Plantado
- [ ] Click/tap → la planta aparece en el punto exacto + 28 cubos de tierra · sin teleport, animación ni bloqueo de movimiento.
- [ ] Rechazos: fuera del suelo · < 1 stud (`too_close`) · 201 plantas · > 30 studs · jardín ajeno.
- [ ] Sin confirmación por rareza; sin `confirm_replace`.
- [ ] Datos: `plants[slotId]` con `x/z/localX/localZ`.
- [ ] Rejoin en el **mismo** jardín y en **otro** jardín → misma posición relativa (con `GardenService` forzado temporalmente o con 2 clientes).
- [ ] Planta apoyada en el suelo · modelo `Plant_g:slot` con `GardenId/SlotId/DefinitionId` · partes con CanQuery = true.

## F3 — Frutas
- [ ] Single-harvest (carrot): timer desde que se planta → exactamente en 0 aparece el prompt de cosecha (sin reconectar).
- [ ] Multi-harvest: timer de planta → 60 s + timer de fruta → prompt.
- [ ] Cosechar funciona · rejoin restaura las frutas.
- [ ] Regadera Cosmic: el timer acelera en vivo, sin congelarse con dos riegos, y el prompt solo aparece cuando el servidor la da por lista.
- [ ] Un solo timer visible (el más cercano, ≤ 12 studs), responsive en el emulador móvil · la E solo en frutas maduras.
- [ ] Sin logs de `ParcelGameplayService`.

## F4
- [ ] Sistemas borrados (no deshabilitados) · plantas y frutas se restauran sin errores · sin `require` rotos.

## F5
- [ ] Conteos de CanQuery/CastShadow reducidos · plantas, pets, NPCs, `EggPlaced` y `GardenFloor` siguen clickeables · `WaterAnimator` anima · sin scripts en los Water · `GiroScript` RunContext Client gira · StarCube animado por `DefinitionId` · scripts marca intactos y deshabilitados · Tools con `CanBeDropped = false` · lava sigue empujando.

## F6 — Tools
- [ ] Regaderas: radios 2,5/3,75/5/6,25 · VFX del tamaño del radio · consume 1 · sin plantas → no consume + aviso · animación del personaje.
- [ ] Un solo `WaterComplete`.
- [ ] Sprinklers: en cualquier punto · radios 7/14/21/28 · 300 s · tiers distintos suman · plantar dentro de la zona recibe el efecto · expira y limpia · gira · suena · sin combo secreto ni confirmación.
- [ ] Pala: 9 studs (horizontal) · 2 s · sin cambios de velocidad.
- [ ] Trowel: levantar ≤ 9 · colocar en cualquier punto · inválido mantiene la planta · cancelar no consume · rejoin conserva la posición.
- [ ] Extractor ≤ 9, 0,4 s → inventario · Plant item se planta como semilla conservando el estado.
- [ ] Cooldown de 0,2 s (varita 0,1 s) · móvil con toque.

## F7 — Huevos
- [ ] 14 `EggSlot` únicos · colocar en un soporte elegido · ocupado → aviso · noveno huevo → `garden full` · click en el jardín → aviso "red stands" una vez por intento.
- [ ] Huevo viejo (por parcela) reubicado con su progreso · crece x4 · prompt en el Attachment del asset · timer y hatch a 9 studs.
- [ ] Lejos → "Get closer to the egg!" una sola vez · vibración + sonido (solo ≤ 9) · desarme con física + sonido 3D · visible para otros jugadores.
- [ ] Sweet Egg = 18000.

## F8 — Pets
- [ ] Sweet Frog/Dog (y Nebula) caminan a la planta, aceleran el timer y la planta queda resaltada 3 s · sonidos de habilidades · Lucky Block en el mismo soporte o el primero libre · Lunaris descuenta.

## F9 — Garden Level
- [ ] Common +1 … OdysseySecret +8 · curva 6N²+44 · cartel en vivo, igual para los visitantes · Lv. 100 MAX dorado · prompt solo para el dueño, 1 s, confirmación, cobro 1M, nivel 1/0 XP, título "Garden Level N" · sin monedas → aviso · bonus 0,05 %/nivel (tope 50 %) al vender · persiste · `MaxLevel = 100`.

## F10–F11
- [ ] Migración: `ENABLED = false`, ModuleScript llamado antes de `restorePlants`, reglas según MIG-01 · búsqueda de `01` §5 → 0 referencias activas · todos los scripts compilan · Garden_001 ≈ 1.900 descendientes · tutorial apunta al suelo · `GamepadController` intacto.

## F12–F13
- [ ] Clones con IDs únicos y todas sus carpetas · Max Players · valores de producción · migración probada con la cuenta real (plantas → inventario, flag puesto) · publicado solo con autorización.
