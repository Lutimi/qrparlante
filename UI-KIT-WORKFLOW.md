# UI Kit Documentation — Proceso de trabajo

Cómo construimos el `UI Kit Documentation` en Figma y qué convenciones seguir para mantenerlo.

---

## Resultado final

Frame: **`UI Kit Documentation`** en la página **`Design System`**.  
ID actual del nodo: `9011:81015` (puede cambiar si se reconstruye).

Para enfocar el viewport en él:

```powershell
cd figma-cli
node zoom-to-doc.mjs
```

---

## Estructura del documento

```
UI Kit Documentation (Frame 1680px)
├── Hero                         — título, descripción, pills
├── Content (Frame contenedor)
│   ├── 01 Buttons & Actions     — 9 variantes dark + light
│   ├── 02 Inputs & Form         — DocID + Contraseña
│   ├── 03 Labels & Text States  — 4 variantes Default/Active × M/S
│   ├── 04 Merchant Logos        — 9 comercios reales
│   ├── 05 Cards                 — Coupon Status + Merchant Detail
│   ├── 06 Navigation            — Bottombar 4 estados
│   ├── 07 Feedback              — Feed + Inline Feedback
│   ├── 08 Color Tokens          — 12 swatches con nombre y hex
│   └── 09 Typography Scale      — tabla text/{size}/{weight} con muestra real
```

---

## Scripts usados para construirlo

Cada sección tiene su propio script. Correrlos en orden si hay que reconstruir.

| Script | Qué hace |
|--------|----------|
| `uikit-v3-step1.mjs` | Borra el doc anterior y crea Hero + sección Buttons (dark + light rows) |
| `uikit-v3-step2.mjs` | Inputs (DocID y Contraseña) + Labels & Text States |
| `uikit-v3-step3.mjs` | Merchant Logos (9) + Cards (Coupon + Merchant Detail) |
| `uikit-v3-step4.mjs` | Navigation (Bottombar 4 estados) + Feedback (Feed + Inline) |
| `uikit-v3-step5.mjs` | Color Tokens (swatches) + Typography Scale |
| `uikit-apply-text-styles.mjs` | Reconstruye tabla Typography con nomenclatura real y aplica text styles a todos los nodos de texto del doc |

### Orden de ejecución

```powershell
cd figma-cli
node uikit-v3-step1.mjs   # esperar que termine (~50s)
node uikit-v3-step2.mjs   # (~45s)
node uikit-v3-step3.mjs   # (~17s)
node uikit-v3-step4.mjs   # (~17s)
node uikit-v3-step5.mjs   # (~17s)
node uikit-apply-text-styles.mjs  # (~17s)
node zoom-to-doc.mjs       # enfocar vista en el doc
```

> Los steps 1 y 2 tardan ~45-50s porque insertan muchas instancias. Hay que dejarlos terminar aunque parezca que cuelgan. Si superan 90s, matar con `taskkill /PID <pid> /F` e inspeccionar qué quedó con `inspect-uikit-sections.mjs`.

---

## Convenciones de diseño aplicadas

### Grid
- Documento: **1680px** ancho
- Padding horizontal: **56px** a cada lado
- Ancho de contenido: **1568px**

### Fondos por tipo de contenido

| Contexto | Fondo frame | Uso |
|----------|-------------|-----|
| Dark components | `#18181B` | Botones dark, logos, navigation, inline feedback |
| Slot dentro de dark | `#111113` | Caja individual de cada variante |
| Light components | `#F4F4F5` + stroke `#E4E4E7` | Botones light, inputs |
| Slot dentro de light | `#FFFFFF` + stroke `#E4E4E7` | Caja individual |
| Feed / cards light | `#FAFAFA` + stroke `#E4E4E7` | Coupon cards, feed |
| Foundations | `#1C1C20` | Color swatches, typography scale |
| Documento base | `#0F0F11` | Fondo de todo el frame |

### Colores de texto por fondo

| Fondo | Título | Etiqueta variante | Meta / secundario |
|-------|--------|-------------------|-------------------|
| Oscuro | `#FAFAFA` | `#A1A1AA` | `#52525B` |
| Claro  | `#18181B` | `#3F3F46` | `#71717A` |

### Separadores entre secciones
- Línea `#27272A`, 1px, ancho completo
- Número de sección `#52525B`, 11px Medium
- Título sección `#FAFAFA`, 30px Bold
- Subtítulo `#71717A`, 13px Regular

---

## Convención de text styles

Todos los nodos de texto del UI Kit usan los text styles del archivo.  
Nomenclatura: `text/{size}/{weight}`

Ejemplos:
```
text/base/bold      → 16px Bold
text/sm/medium      → 14px Medium
text/xs/regular     → 12px Regular
text/xxs/semibold   → 10px SemiBold
```

Para re-aplicar text styles a todos los nodos del doc:
```powershell
node uikit-apply-text-styles.mjs
```

---

## IDs de componentes clave (App V1)

Estos IDs se usan en los scripts para insertar instancias. Si un componente se mueve o elimina hay que volver a inspeccionarlos con `inspect-core-variant-ids.mjs`.

### Buttons — `Section/Auth/Actions`
| Variante | ID |
|----------|----|
| Default · Yellow · M | `7191:6700` |
| Inactive · Glass · M | `7191:6699` |
| Default · Yellow · S | `7191:6715` |
| Default · White · XS | `7191:6717` |
| Text · Default · M   | `7191:6698` |
| Text · Blue · S      | `7191:6719` |
| Text · White · S     | `7191:6721` |
| Text · Blue · XS     | `7191:6723` |
| Text · Red · XS      | `7235:10917` |

### Inputs
| Componente | ID |
|------------|----|
| Documento de identidad | `7195:5981` |
| Contraseña | `7195:5966` |

### Text Labels — `Text`
| Variante | ID |
|----------|----|
| Default · M | `7205:8434` |
| Active · M  | `7205:8433` |
| Default · S | `7243:9346` |
| Active · S  | `7243:9327` |

### Merchant Logos
| Comercio | ID |
|----------|----|
| Casa Andina | `7205:4697` |
| Edo | `7205:4696` |
| Groomers | `7205:4693` |
| JetSMART | `7205:4692` |
| La Cuadra de Salvador | `7205:4694` |
| Primax | `7205:4691` |
| Tanta | `7205:4690` |
| UVK | `7205:4689` |
| Wong | `7205:4695` |

### Coupon Status Card
| Estado | ID |
|--------|----|
| Cupon  | `7235:10932` |
| adaesfa | `7235:10931` |
| Expiró | `7235:10930` |

### Merchant Detail Card
| Variante | ID |
|----------|----|
| Con el comercio | `7240:9443` |
| Sin el comercio | `7240:9442` |

### Bottombar
| Estado | ID |
|--------|----|
| Bene active | `3896:1498` |
| Cupones active | `3896:1515` |
| Consumos active | `3896:1532` |
| Credencial active | `3896:1549` |

### Feed
| Variante | ID |
|----------|----|
| Check · Default | `7226:9398` |
| Error · Default | `7226:9396` |
| Check · Black | `7226:9397` |
| Error · Black | `7226:9395` |

### Inline Feedback
| Variante | ID |
|----------|----|
| Icono final · Default | `7226:8451` |
| Icono final · Red | `7226:8450` |
| Texto inicio · Default | `7226:8448` |
| Icono inicio · Default | `7226:9117` |
| Icono ambos · Default | `7226:8449` |

---

## Scripts de inspección útiles

```powershell
# Ver todas las páginas del archivo
node inspect-design-system.mjs

# Ver los children del UI Kit doc por sección
node inspect-uikit-sections.mjs

# Ver el ID actual del frame UI Kit Documentation (para screenshots)
node get-doc-id.mjs

# Obtener todos los text styles con familia, peso, tamaño
node inspect-text-styles-full.mjs

# Obtener IDs de componentes clave y sus variantes
node inspect-core-variant-ids.mjs
```
