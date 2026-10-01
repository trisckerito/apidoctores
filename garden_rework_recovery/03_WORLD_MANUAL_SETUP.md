# 03 — WORLD MANUAL SETUP
## Todo lo que el USUARIO crea o ajusta a mano en Roblox Studio

> **Regla para la nueva ventana:** nada de este documento se crea por código "por si acaso".
> Cuando una sección lo necesite → **DETENERSE** y usar la plantilla de `00_MASTER_PLAN.md` §4.
> - Si el usuario dice **NO**: darle las instrucciones de este documento.
> - Si dice **SÍ**: **verificar con el MCP** (nombres, tipos, parent, attributes) antes de continuar.
>
> El historial **no registró** tamaños, posiciones, materiales ni colores exactos. Todo lo que no consta está marcado `UNKNOWN / REQUIERE VERIFICACIÓN`. **No inventar valores**: preguntar al usuario o leer un jardín existente.

---

## 0. CHECKLIST DE INICIO (presentar al usuario ANTES de tocar nada)

> La nueva ventana debe mostrar esta lista **tal cual** y preguntar: *"¿Ya hiciste esto en el juego principal?"*.
> Para cada punto, recordar **cómo lo hizo o lo describió el usuario** en el desarrollo anterior (columna "Cómo se hizo antes").
> Si el usuario no recuerda un detalle marcado UNKNOWN → preguntarle si la copia DEV sigue existiendo para **leer allí el objeto** o **copiarlo** al principal.

| # | Tu parte en el mapa | Cómo se hizo antes (según el historial) | Qué verifica la ventana |
|---|---|---|---|
| 1 | **Respaldo del juego principal** | — (nuevo, por trabajar en el principal) | Que el usuario confirme que hay una versión guardada o publicada de respaldo |
| 2 | **`GardenFloor`** en cada jardín | En el DEV existía un Part único `GardenFloor` con attribute `GardenId`. **Tú lo subiste de altura** respecto a las parcelas ("cuando subiste el suelo"). No consta si lo creaste tú o el agente, ni su tamaño, material o posición | Part, `Anchored`, `CanCollide`, `CanQuery`, `GardenId` = `GardenID` del jardín, que cubra el área de plantado |
| 3 | **Cartel `GardenLevel`** | Tus palabras: *"el cartel ya lo cree, lo cree en Garden_001, dentro hay un model GardenLevel y tiene una surfacegui en una cara"* | `Model` `GardenLevel` dentro del jardín, con `SurfaceGui` en una cara de un Part |
| 4 | **Prompt de resonancia del cartel** | Tus palabras: *"va a haber un attachment dentro de un part del model GardenLevel que se llama ProximityPrompt"*. Ambiguo: ¿el Attachment se llama "ProximityPrompt", o es un Attachment con un ProximityPrompt dentro? | Preguntar la variante y verificar que existe |
| 5 | **Soportes de huevos (carpeta `Eggs`)** | En el DEV cada jardín tenía una carpeta `Eggs`, y el sistema de huevos colocaba los huevos "en soportes" conservando la incubación. **El historial que se analizó NO incluye cómo creaste los soportes** (cantidad, nombres, tipo de objeto, attributes, ubicación). **UNKNOWN** | Pedir al usuario que describa cómo los creó, o leerlos en la copia DEV. No inventarlos |
| 6 | **Carpeta `Sprinklers`** | Existía en cada jardín del DEV. No consta si la creaste tú ni su contenido. **UNKNOWN** | Igual que el punto 5 |
| 7 | **Jardines del juego principal** | En el DEV **borraste `Garden_002`…`Garden_008`** para luego **clonar `Garden_001`** ya terminado. En el principal siguen los 8 jardines con parcelas | Preguntar si repite la estrategia (dejar `Garden_001`, terminarlo y clonarlo al final) o adapta cada jardín |
| 8 | **IDs de los clones** | Instrucción del agente: renombrar `Garden_002`, `Garden_003`…; cambiar `GardenID` en el modelo y `GardenId` en su `GardenFloor` a 2, 3… (al clonar, todos heredan 1) | IDs únicos y coincidentes (se verifica al final) |
| 9 | **Max Players** | Pregunta tuya: con 6 bases entran 6 jugadores. Respuesta: sí, poniendo Max Players = número de jardines (Game Settings → Places) | Valor = número de jardines (se verifica al final) |
| 10 | **API Services en Studio** | Para probar la migración con datos reales: Game Settings → Security → *Enable Studio Access to API Services*, y salir del juego publicado con tu cuenta | Preguntar si lo desactiva mientras se desarrolla |

> Los puntos 7 a 9 se hacen **al final** (tras la limpieza), pero se mencionan al inicio para que el usuario sepa lo que vendrá.

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
