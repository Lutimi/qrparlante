# Figma Bridge — Guía de conexión

Todo lo que necesitas saber para conectar el bridge CDP con Figma Desktop y no perder tiempo cada vez que empecemos a trabajar.

---

## Cómo funciona el bridge

El CLI se conecta a Figma Desktop vía **Chrome DevTools Protocol (CDP)**.  
Figma Desktop es una app Electron y expone un WebSocket de debugging en el puerto **9222**, pero las versiones modernas tienen esa función bloqueada por defecto.

El archivo `figma-cli/src/figma-patch.js` parchea el binario de Figma (`app.asar`) para desactivar ese bloqueo. Una vez parcheado y reiniciado Figma, el cliente WebSocket en `figma-client.js` se conecta y puede ejecutar código JavaScript directamente en el contexto de Figma.

```
Cursor / Node.js
    ↓  WebSocket (CDP)
Figma Desktop (puerto 9222)
    ↓  figma.* API
Documento abierto
```

---

## Checklist de arranque (hacer en orden)

### 1. Verificar si Figma ya está parcheado

```powershell
cd figma-cli
node src/figma-patch.js
```

Salidas posibles:

| Salida | Qué significa |
|--------|---------------|
| `PATCHED=true` | Todo listo, salta al paso 3 |
| `PATCHED=false` | Hay que parchear, ir al paso 2 |
| `PATCH_OK` | Se acaba de aplicar el parche |

### 2. Aplicar el parche (solo si PATCHED=false)

Abrir PowerShell **como Administrador** y correr:

```powershell
cd C:\Users\misat\Desktop\claude2figma\figma-cli
node src/figma-patch.js
```

Debe responder `PATCH_OK`.

> **Por qué se necesita admin**: el parche modifica el archivo `app.asar` dentro de `%LOCALAPPDATA%\Figma\app-X.X.X\resources\`. Sin permisos de escritura el script falla silenciosamente.

### 3. Cerrar y reabrir Figma Desktop

El parche solo tiene efecto si Figma se inicia **después** de aplicarlo.  
Si ya estaba abierto: cerrar completamente (incluir tray icon) y volver a abrir.

### 4. Abrir el archivo de diseño en Figma

El cliente busca una pestaña cuya URL contenga `figma.com/design` o `figma.com/file`.  
Si no hay ninguna pestaña con esa URL la conexión falla con:

```
Error: No Figma design file open. Please open a design file in Figma Desktop.
```

### 5. Probar la conexión

```powershell
node inspect-design-system.mjs
```

Si devuelve datos del documento → conexión OK.

---

## Por qué se pierde el parche

Figma Desktop se **auto-actualiza**. Cada actualización reemplaza el `app.asar` con uno nuevo sin el parche. Señales de que esto pasó:

- El script de conexión devuelve `DISCONNECTED` aunque Figma esté abierto.
- `node src/figma-patch.js` devuelve `PATCHED=false` cuando antes era `true`.

**Solución**: repetir los pasos 2 y 3.

---

## Arquitectura del código

```
figma-cli/
├── src/
│   ├── figma-client.js     # Cliente WebSocket CDP. Expone client.eval(code)
│   ├── figma-patch.js      # Parchea app.asar. Detecta versión auto.
│   └── index.js            # CLI principal (comandos: status, connect, variables…)
├── apply-icons.mjs         # Script de ejemplo — patrón base para todos los scripts
└── *.mjs                   # Scripts de trabajo (inspección, build, etc.)
```

### Patrón base de todo script

```js
import { FigmaClient } from './src/figma-client.js';

const client = new FigmaClient();
await client.connect();

const result = await client.eval(`
(async () => {
  // Código que corre dentro de Figma Desktop
  const page = figma.root.children.find(p => p.name === 'App V1');
  return page ? page.children.length : 0;
})()
`);

console.log(result);
client.close();
```

### Reglas del código dentro de `client.eval()`

| Regla | Motivo |
|-------|--------|
| Siempre envolver en `(async () => { ... })()` | El motor de eval requiere una expresión, no statements sueltos |
| Usar `return` dentro del async IIFE | Sin return el resultado es `undefined` |
| No usar template literals anidados con backtick | El backtick externo ya es el template de Node.js — usar `'` o `"` dentro |
| No usar `\n` literal en strings — usar `\\n` | El escape se interpreta en Node antes de llegar a Figma |
| Usar `var` en lugar de `const`/`let` en loops | Evita problemas de scope en algunas versiones del contexto Figma |
| Operaciones pesadas → scripts separados pequeños | Scripts muy largos cuelgan la conexión WebSocket (timeout ~30s) |

---

## Errores comunes y solución

### `Error: No Figma design file open`
→ Figma está abierto pero no hay ningún archivo de diseño en pestaña activa. Abrir el archivo.

### `SyntaxError: Illegal return statement`
→ El código dentro de `eval()` no está envuelto en `async IIFE`. Agregar `(async () => { ... })()`.

### `SyntaxError: Invalid or unexpected token`
→ Hay un backtick `` ` `` o `\n` literal dentro del string que se pasa a `eval`. Revisar el contenido del template.

### Script cuelga y no termina
→ Demasiadas operaciones en un solo `eval`. Partir en scripts más pequeños, verificar con un script de inspección, matar con `taskkill /PID <pid> /F` y continuar desde donde quedó.

### `TypeError: section.resize is not a function`
→ Los nodos tipo `SECTION` no soportan `resize()`. Solo los `FRAME` lo soportan.

### `Could not find Figma execution context`
→ Figma v39+ usa contextos de ejecución sandboxed. Normalmente el cliente los detecta automáticamente. Si falla: cerrar Figma, reabrirlo, y reconectar.

---

## Verificar estado rápido

```powershell
# Ver si el parche sigue activo tras una actualización de Figma
node src/figma-patch.js

# Ver qué páginas tiene el archivo abierto
node inspect-design-system.mjs

# Enfocar viewport en el UI Kit Documentation
node zoom-to-doc.mjs
```
