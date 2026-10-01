# 03 — WORLD MANUAL SETUP
## Todo lo que el USUARIO crea o ajusta a mano en Roblox Studio

> **Regla para la nueva ventana:** nada de este documento se crea por código "por si acaso".
> Cuando una sección lo necesite → **DETENERSE** y usar la plantilla de `00_MASTER_PLAN.md` §4.
> - Si el usuario dice **NO**: darle las instrucciones de este documento.
> - Si dice **SÍ**: **verificar con el MCP** (nombres, tipos, parent, attributes) antes de continuar.
>
> El historial **no registró** tamaños, posiciones, materiales ni colores exactos. Todo lo que no consta está marcado `UNKNOWN / REQUIERE VERIFICACIÓN`. **No inventar valores**: preguntar al usuario o leer un jardín existente.

---

## 1. Qué consta como manual en el historial

| # | Objeto | ¿Manual? | Evidencia |
|---|---|---|---|
| M1 | Model `GardenLevel` (cartel) con `SurfaceGui` | **Sí** [CONFIRMADO] | Usuario: "el cartel ya lo cree, pero lo cree en Garden_001 dentro hay un model GardenLevel y tiene una surfacegui en una cara" |
| M2 | Attachment/prompt dentro de un Part de `GardenLevel` | **Sí** [CONFIRMADO, nombre ambiguo] | Usuario: "va a haber un attachment dentro de un part del model GardenLevel que se llama ProximityPrompt" |
| M3 | Altura del `GardenFloor` | **Sí, el usuario la ajustó** [CONFIRMADO] | Agente: "el BaseSoil quedó más bajo cuando subiste el suelo" |
| M4 | `GardenFloor` (creación) | **UNKNOWN** si lo creó el usuario o el agente | No consta |
| M5 | Borrado de `Garden_002` … `Garden_008` | **Sí** [CONFIRMADO] | Usuario: "he borrado todos los garden_002 … hasta el 008" |
| M6 | Clonado de `Garden_001` → N jardines + IDs | **Sí, lo hará el usuario** [CONFIRMADO] | Usuario: "yo luego copiare garden_001" |
| M7 | Max Players del place | **Sí** [CONFIRMADO] | Game Settings |
| M8 | "Enable Studio Access to API Services" | **Sí** (solo para la prueba de migración en el original) | Game Settings → Security |
| M9 | Carpetas `Eggs` / `Sprinklers` del jardín | **UNKNOWN** si son manuales | Solo se sabe que existen |
| M10 | Soportes de huevos | **UNKNOWN** | Solo se sabe que existen |
| M11 | Eliminación de la carpeta `Parcels` (160 modelos) | La hizo **el agente** por código [CONFIRMADO] | "Eliminado: Las 160 parcelas de Garden_001" |

---

## 2. Jerarquía de referencia del jardín (estado final)

```
Workspace
└── Gardens                              (Folder — preexistente)
    └── Garden_001                       (Model — preexistente; Attribute GardenID = 1)
        ├── GardenFloor                  (Part — ver §3.1; Attribute GardenId = 1)
        ├── GardenZone                   (preexistente — zona de entrada, se conserva)
        ├── Eggs                         (Folder? — soportes de huevos; UNKNOWN estructura)
        ├── Sprinklers                   (Folder? — UNKNOWN estructura)
        ├── GardenLevel                  (Model — MANUAL, ver §3.4)
        │   ├── <Part con SurfaceGui>    (nombre UNKNOWN)
        │   │   └── SurfaceGui           (cara UNKNOWN)
        │   └── <Part>                   (nombre UNKNOWN)
        │       └── ProximityPrompt / Attachment (ver §3.4, nombre ambiguo)
        └── … (resto del jardín preexistente — 1.898 instancias en total tras el rework)
    (NO debe existir: Parcels/, BaseSoil, PlantPivot, InteractionPivot)
```

> El total de **1.898 instancias** en `Garden_001` tras el rework [CONFIRMADO] sirve de referencia aproximada. Un número cercano a ~20.600 indica que las parcelas siguen ahí.

---

## 3. Fichas de objetos

### 3.1 `GardenFloor` (uno por jardín)

| Propiedad | Valor | Estado |
|---|---|---|
| Nombre | `GardenFloor` | [CONFIRMADO] |
| Tipo | `Part` | [CONFIRMADO] (orden: "un solo Part plano, grande") |
| Parent | `Garden_00N` (el jardín) | [INFERIDO] (verificar si está directamente en el modelo o en una subcarpeta) |
| Anchored | `true` | [CONFIRMADO] (orden) |
| CanCollide | `true` | [CONFIRMADO] (orden) |
| CanQuery | debe ser `true` (lo usa el raycast de plantado) | [INFERIDO] — imprescindible |
| CanTouch | UNKNOWN (recomendado `false` si no se usa Touched) | UNKNOWN |
| Attribute `GardenId` | número del jardín (1, 2, …) | [CONFIRMADO] |
| Size | **UNKNOWN** — debe cubrir el área antigua de las 160 parcelas | UNKNOWN |
| Position / CFrame | **UNKNOWN** — el usuario la **subió** respecto al `BaseSoil` | UNKNOWN |
| Orientation | Plano (0, 0, 0) o (0, Y, 0) | [INFERIDO] |
| Material / Color / Transparency | **UNKNOWN** | UNKNOWN |

**Instrucciones si no existe:**
1. En `Garden_001`, insertar un `Part` y nombrarlo `GardenFloor`.
2. Escalarlo para cubrir toda la zona donde estaban las parcelas. Dejarlo plano.
3. `Anchored = true`, `CanCollide = true`, `CanQuery = true`.
4. Properties → Attributes → `+` → Name `GardenId`, Type `number`, Value `1`.
5. Material y color a gusto (el historial no registró los originales).

### 3.2 Attribute del modelo de jardín

| Objeto | Attribute | Valor | Estado |
|---|---|---|---|
| `Garden_00N` (Model) | `GardenID` | N | [CONFIRMADO por el agente: "todas las copias heredan GardenID = 1"] |

⚠ El modelo usa `GardenID` y el suelo `GardenId`. Verificar la grafía exacta en el proyecto. Si difiere, **no** renombrar sin preguntar: los scripts dependen de ella.

### 3.3 Carpetas `Eggs` y `Sprinklers`

- Existen en cada jardín [CONFIRMADO] y deben clonarse con `Garden_001`.
- Tipo, número de soportes, nombres de hijos y attributes: **UNKNOWN / REQUIERE VERIFICACIÓN**.
- Acción: en la Sección 0, leer su estructura en el place (si sobrevivió) y **documentarla aquí antes de seguir**. Si no existen, preguntar al usuario cómo eran. No inventar soportes.

### 3.4 `GardenLevel` (cartel de nivel) — MANUAL

| Elemento | Tipo | Detalle | Estado |
|---|---|---|---|
| `GardenLevel` | `Model` | Dentro de `Garden_001` | [CONFIRMADO] |
| Part con la cara del cartel | `Part` | Nombre UNKNOWN | UNKNOWN |
| `SurfaceGui` | `SurfaceGui` | En **una cara** del Part (`Face` UNKNOWN) | [CONFIRMADO que existe] |
| Labels dentro del `SurfaceGui` | `TextLabel` + barra | Título, "Lv. X", barra, "a / b EXP". **UNKNOWN** si los creó el usuario o el agente | UNKNOWN |
| Prompt | `ProximityPrompt` y/o `Attachment` | El usuario dijo: *"un attachment dentro de un part del model GardenLevel que se llama ProximityPrompt"* | **Ambiguo** |

**Ambigüedad del prompt (preguntar al usuario si no se puede leer del place):**
- (a) un `Attachment` **llamado** `ProximityPrompt` (y el código crea o usa un `ProximityPrompt` dentro), o
- (b) un `Attachment` con un `ProximityPrompt` dentro, o
- (c) un `ProximityPrompt` directamente en el Part.

Lo que sí es final [CONFIRMADO]: texto **"Resonate (1,000,000)"**, **HoldDuration 1 s**, habilitado solo en nivel 100 y solo para el dueño.

**Instrucciones si no existe** (pedir confirmación de la variante a/b/c antes):
1. En `Garden_001`, crear un `Model` llamado `GardenLevel`.
2. Dentro, un `Part` anclado para el cartel, con un `SurfaceGui` en la cara visible.
3. Un `Part` (puede ser el mismo) con el `Attachment` / `ProximityPrompt` según la variante elegida.

### 3.5 Configuración del place

| Ajuste | Dónde | Valor |
|---|---|---|
| Max Players | Game Settings → Places → Max Players (o Creator Hub) | = número de jardines (ej. 6) |
| Studio Access to API Services | Game Settings → Security | `ON` solo para probar la migración con datos reales |

---

## 4. Clonado de jardines (lo hará el usuario; la ventana lo verifica)

Para cada `k` de 2 a N:
1. Duplicar `Garden_001` (con `GardenFloor`, `Eggs`, `Sprinklers` y `GardenLevel`).
2. Renombrar a `Garden_00k`.
3. Cambiar `GardenID = k` en el modelo.
4. Cambiar `GardenId = k` en **su** `GardenFloor`.
5. Moverlo a su ubicación en el mapa (posiciones **UNKNOWN**; las del antiguo `Garden_00k` no se registraron).

**La ventana debe verificar:** IDs únicos, que coincidan modelo ↔ suelo, nombres únicos y que no quede ninguna carpeta `Parcels`.
