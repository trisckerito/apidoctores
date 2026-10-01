# 03 — PARTE MANUAL DEL USUARIO EN EL MUNDO

> **Regla:** nada de lo que el usuario hizo a mano se reemplaza por código "por si acaso". Cuando una fase lo necesite → **parar y preguntar**.
> Si responde que sí → **verificar con el MCP** (nombres, tipos, parent, attributes). Si responde que no → darle las instrucciones de este documento.

---

## 0. CHECKLIST DE INICIO (presentar ANTES de tocar nada)
Mostrarlo al usuario tal cual, con la columna "Cómo se hizo antes", y preguntar **qué ya está hecho en el juego principal**.

| # | Tu parte | Cómo se hizo antes (según la conversación) | Cuándo hace falta |
|---|---|---|---|
| 1 | **Respaldo** del juego principal | Nuevo, por el cierre forzado de Studio: *File → Save to File As* (`.rbxl`) + una versión en el historial | Antes de empezar |
| 2 | **Ajustar el `GardenFloor` de Garden_001** | El agente lo crea por código (copiando el Water: 92×92). **Tú** después lo subiste un poco, lo rotaste unos grados, le diste elevación y lo **pintaste** (color/material/transparencia a tu gusto) | Después de la Fase 1 |
| 3 | **Cuadrícula de agua (regadío)** en Garden_001 | **Tú** multiplicaste el Part `Water` hasta tener **12**: 10 canales paralelos (1 × 0,4 × 92, separados 10 studs) + 2 bordes transversales (92 × 0,4 × 1), como una granja con agua. Los scripts internos los quita el agente | Cuando quieras (decorativo) |
| 4 | **Carpeta `Eggs` con soportes** en Garden_001 | **Tú** creaste la carpeta `Eggs` con Parts **`EggPlaced`**, cada uno con un **`Attachment` llamado `EggAttachment`** (ahí va el huevo). Primero 13 y luego **14**: 7 a cada costado del jardín. Son los "soportes rojos". El número `EggSlot` lo pone el agente | Antes de la fase de huevos |
| 5 | **Mover el prompt de los huevos** en los assets | **Tú** moviste el **`Attachment "ProximityPrompt"`** dentro de los assets de huevos (modelo "Placed") a la posición deseada | Antes de la fase de huevos |
| 6 | **Cartel `GardenLevel`** en Garden_001 | Tus palabras: *"dentro hay un model GardenLevel y tiene una surfacegui en una cara"* y *"un attachment dentro de un part del model GardenLevel que se llama ProximityPrompt"* | Antes de la fase de Garden Level |
| 7 | **Terreno / decoración** del jardín | Ajustaste el terreno y el jardín "para que se vea bonito" antes de la Fase 2 | Opcional |
| 8 | **Borrar Garden_002…008 y clonar Garden_001** | **Tú** los borraste al final para clonar Garden_001 terminado. Clonar con *Ctrl+D* y reposicionar a mano (así se evitaron los problemas de orientación del agua) | Al final |
| 9 | **IDs de los clones** | Renombrar `Garden_002`, `Garden_003`… y cambiar el attribute `GardenID` del modelo y `GardenId` de su `GardenFloor` a 2, 3… (al clonar, todos heredan 1) | Al final |
| 10 | **Max Players** | Número de jugadores = número de bases (Game Settings → Places) | Al final |
| 11 | **API Services** | Game Settings → Security → *Enable Studio Access to API Services*, y salir del juego publicado con tu cuenta | Solo para probar la migración |

**Atajo:** si quedó alguna versión del place DEV (`86748110736040`), los puntos 2 a 6 se pueden **copiar y pegar** desde ahí.

---

## 1. Fichas de los objetos

### 1.1 `GardenFloor` (uno por jardín)
| Propiedad | Valor |
|---|---|
| Nombre / tipo / parent | `GardenFloor` / `Part` / `Garden_00N` |
| Creación | Por código: mismo CFrame y tamaño que el `Water` (92×92) |
| Altura inicial | Cara superior **0,05 studs sobre el tope de los `BaseSoil`** |
| Anchored / CanCollide / CanQuery / CastShadow | `true` / `true` / **`true`** / `false` |
| Attribute | `GardenId` = número del jardín |
| Posición, rotación, color, material y transparencia finales | Los ajusta **el usuario** a mano (valores exactos **UNKNOWN**) |

### 1.2 Attribute del jardín
`Garden_00N.GardenID`: en el DEV era **string** (`"1"`) [CONFIRMADO]. Mantener el mismo tipo en los clones. Ojo: el modelo usa `GardenID` y el suelo `GardenId` (mayúsculas distintas).

### 1.3 Carpeta `Eggs`
```
Garden_001
└── Eggs (Folder)
    ├── EggPlaced (Part)  ← ×14, soportes rojos (7 a la izquierda, 7 a la derecha)
    │   └── EggAttachment (Attachment)
    └── ...
```
- **Por código:** attribute `EggSlot` = 1..14 (orden por posición) y `CanQuery = true`.
- Tamaño de referencia: ~5,5 studs de ancho, separados ~8 studs entre centros [CONFIRMADO por el agente].
- Si se duplica un soporte → pedir al agente que **renumere**.

### 1.4 Carpeta `Sprinklers`
- Contenedor vacío de los sprinklers colocados. **La crea el agente por código** (no es manual), pero debe existir en cada jardín: clonando Garden_001 se copia sola.

### 1.5 Cartel `GardenLevel`
```
Garden_001
└── GardenLevel (Model)
    ├── <Part con la cara del cartel>
    │   └── SurfaceGui   ← labels centrados (los crea o ajusta el agente)
    └── <Part>
        └── ProximityPrompt (Attachment)  ← el código crea ahí el ProximityPrompt "Resonate"
```
(Convención del proyecto: los prompts se crean en un `Attachment` **llamado** `ProximityPrompt`, igual que en plantas, frutas y huevos.)

### 1.6 Cuadrícula de agua (decorativa)
- 12 Parts `Water` en Garden_001, en coordenadas locales del `GardenFloor`. **Sin scripts dentro.**
- Las animan `WaterAnimator` (LocalScript, cliente) — 6 texturas por Water.
- `CanQuery = false`, `CastShadow = false`.
- `MeshId` no se puede cambiar en runtime → si hay que replicar, **clonar**.

---

## 2. Clonado final (usuario) y verificación (agente)
Para cada k = 2..N: duplicar `Garden_001` (con `GardenFloor`, `Eggs`, `Sprinklers`, `GardenLevel` y el agua) → renombrar a `Garden_00k` → `GardenID = k` en el modelo y `GardenId = k` en su `GardenFloor` → reposicionar y rotar a mano.
**El agente verifica:** IDs únicos y coherentes, que no queden `Parcels`, que los `EggSlot` estén completos y Max Players = N.
