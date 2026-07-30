# Productos de la Costa — Panel de administración

Aplicación de gestión para un negocio de fabricación y distribución/reparto de
productos (frituras, sueros, etc.): control de inventario, clientes, pedidos,
créditos, rutas de reparto y gastos de caja.

## 1. Tipo de aplicación

**PWA (Progressive Web App)** instalable — no es una app nativa, pero se
comporta como una: tiene ícono, abre en pantalla completa (sin barra del
navegador), funciona offline gracias a un Service Worker, y se puede
"instalar" en Android/Escritorio directo desde el navegador.

- **Sin build step / sin Node.js para desarrollarla**: es HTML + React vía
  CDN, transpilado en el navegador en tiempo real con Babel Standalone. Se
  edita el `.html` directo y se recarga — no hay `npm install` ni proceso de
  compilación.
- **Backend**: Firebase (Authentication + Firestore) en tiempo real. No hay
  servidor propio ni API intermedia — el navegador habla directo con
  Firebase.
- **Multiplataforma**: funciona en cualquier navegador moderno (Chrome,
  Safari, Edge) en celular, tablet o escritorio. Está optimizada para
  pantalla angosta (`max-width:420px`), pensada primero para celular.

## 2. Stack técnico

| Pieza | Tecnología | Cómo se carga |
|---|---|---|
| UI | React 18 | CDN (`react.production.min.js` / `react-dom`) |
| Transpilado JSX | Babel Standalone 7.23.2 | CDN, en el navegador |
| Autenticación | Firebase Auth (compat) | CDN `firebase-auth-compat.js` |
| Base de datos | Firebase Firestore (compat) | CDN `firebase-firestore-compat.js` |
| Escaneo de código de barras | html5-qrcode 2.3.8 | CDN |
| Tipografías | Oswald, IBM Plex Sans, IBM Plex Mono | Google Fonts |
| Offline | Service Worker propio (`sw.js`) | cachea el "app shell" |
| Persistencia local | `Firestore.enablePersistence()` | IndexedDB del navegador |

No hay build tools (Webpack/Vite/npm scripts) ni framework de routing —
la navegación entre "pantallas" es un simple `useState` (`tab`) que decide
qué componente de React mostrar, todo dentro de una sola página cargada una
vez.

## 3. Estructura de archivos

```
index.html              → shell HTML + estilos + toda la app principal (React/JSX)
firebase-init.js        → SOLO configuración e inicialización de Firebase
gerencia.js             → módulo: gastos y conciliación de caja
rutas-repartidores.js   → módulo: lógica adicional de rutas/repartidores
manifest.json           → metadatos de instalación (íconos, colores, nombre)
sw.js                   → Service Worker (caché offline del app shell)
firestore.rules         → reglas de seguridad de la base de datos
```

Los módulos como `gerencia.js` y `rutas-repartidores.js` se cargan con un
`<script type="text/babel" src="...">` después del bloque principal. Como
todos los `<script>` clásicos de la página comparten el mismo scope, sus
componentes (`Gerencia`, etc.) quedan disponibles para `index.html` sin
imports — es el patrón a seguir para cualquier módulo nuevo que se agregue.

## 4. Diseño visual

**Estilo:** industrial / logística, tema claro, con acento en ámbar/mostaza.

**Tipografías**
| Uso | Fuente |
|---|---|
| Títulos, botones, etiquetas de pestaña | `Oswald` (700/600/500), mayúsculas, tracking amplio |
| Texto general, inputs | `IBM Plex Sans` |
| Códigos, montos pequeños, etiquetas técnicas (`Tag`, `Lbl`) | `IBM Plex Mono` |

**Paleta de colores** (variables CSS en `:root`)

| Token | Hex | Uso |
|---|---|---|
| `--bg` | `#ECEDE7` | Fondo general |
| `--surface` | `#FFFFFF` | Tarjetas (`Card`) |
| `--surface-2` | `#F1F1EA` | Inputs, fondos secundarios |
| `--ink` | `#1B1D19` | Texto principal |
| `--ink-soft` | `#585D53` | Texto secundario |
| `--ink-faint` | `#8B8F84` | Texto terciario / placeholders |
| `--line` / `--line-strong` | `#DEE0D5` / `#C7C9BB` | Bordes |
| `--rail` | `#1E211D` | Barra superior e inferior (navegación) |
| `--rail-border` | `#34372E` | Borde de la barra |
| `--accent` / `--accent-text` | `#E8A400` / `#8A5A00` | Color de marca (ámbar) — botones primarios, pestaña activa |
| `--info` / `--info-text` / `--info-bg` | `#3E7CA6` / `#1F4E6E` / `#DEE7EC` | Estados informativos (transferencia, edición) |
| `--ok` / `--ok-text` / `--ok-bg` | `#2E8B45` / `#1E6630` / `#DFEFE2` | Estados positivos (efectivo, activo, pagado) |
| `--danger` / `--danger-text` / `--danger-bg` | `#C23B2E` / `#8C2A20` / `#F4DEDA` | Errores, eliminar, stock bajo |
| `--warn` / `--warn-text` / `--warn-bg` | `#C2601D` / `#8C4515` / `#F3E4D3` | Crédito pendiente, alertas |
| `--admin` | `#3B4B6B` | Etiqueta de rol admin |

**Color de tema del navegador/PWA:** `#1E211D` (barra de estado en móvil al
instalar la app).

## 5. Componentes reutilizables (átomos de UI)

Definidos una sola vez en `index.html` y usados en todas las pantallas:

- `Card` — contenedor con borde y esquinas redondeadas.
- `BFill` / `BOut` — botón sólido / botón con borde (outline).
- `Inp` — input de texto estilizado.
- `Lbl` — etiqueta pequeña en mayúsculas (estilo formulario).
- `Row` — contenedor flex horizontal.
- `Tag` — pastilla de color para estados (rol, forma de pago, stock).
- `Modal` — hoja inferior (bottom sheet) para formularios.
- `PwInp` — input de contraseña con botón de mostrar/ocultar.
- `PinPad` — teclado numérico circular para el PIN local.
- `BarcodeScanner` — cámara + `html5-qrcode` para leer códigos de barras.
- Iconos SVG inline (`Ic`, `CDown`, `CUp`, `XI`, `ChkSq`, `SqI`, `EyeI`,
  `EyeX`, `Gear`) — sin librería de íconos externa.

## 6. Roles y pestañas

Tres roles: **admin**, **usuario** (vendedor) y **repartidor**.

| Pestaña | admin | usuario | repartidor | Qué hace |
|---|:---:|:---:|:---:|---|
| 🏠 Inicio | ✅ | ✅ | — | Dashboard: ventas del día, créditos pendientes, stock bajo, ranking de clientes |
| 📦 Productos | ✅ | ✅ | — | Alta/edición de productos, código de barras, stock. Eliminar e historial de inventario: solo admin |
| 🧾 Pedido | ✅ | ✅ | ✅ | Armar carrito, elegir/crear cliente, guardar venta (efectivo/transferencia/crédito), enviar ticket por WhatsApp |
| 👥 Clientes | ✅ | ✅ | — | Alta/edición, activar/desactivar, historial de pedidos por cliente |
| 💳 Créditos | ✅ | ✅ | — | Ver saldos pendientes, registrar abonos |
| 🚚 Ruta | ✅ | ✅ | ✅ | Cargar camión (escaneo o manual), registrar entregas, cerrar ruta |
| 💰 Gerencia | ✅ | ✅ | ✅ | Registrar gastos (efectivo/tarjeta), ver caja del día, reporte completo (solo admin) |
| ⚙️ Configuración | ✅ | ✅ | ✅ | Perfil, contraseña, PIN local; gestión de usuarios solo admin |

La navegación (`TABS_POR_ROL`) se define en `App` dentro de `index.html`.

## 7. Datos técnicos no sensibles

- **Colecciones de Firestore:** `productos`, `clientes`, `notas`, `creditos`,
  `inventario_historial`, `rutas`, `gastos`, `usuarios`, `_meta`. Esquema
  completo y reglas de acceso en `firestore.rules`.
- **Autenticación:** correo/contraseña (Firebase Auth). El primer usuario que
  inicia sesión sin perfil existente se crea automáticamente como `admin`.
- **PIN local:** candado opcional por dispositivo (no reemplaza la
  contraseña); se guarda como hash SHA-256 + sal en `localStorage`, nunca en
  Firebase ni en texto plano.
- **Offline:** `enablePersistence()` cachea los datos de Firestore en
  IndexedDB; el Service Worker (`sw.js`) cachea el app shell (HTML + JS de
  CDN) con estrategia network-first para navegación y cache-first para
  assets estáticos.
- **Idioma:** interfaz en español (`lang="es"`, formatos de fecha
  `es-MX`).

## 8. Cómo generar el APK con PWABuilder

Esta app ya cumple los requisitos de una PWA instalable (manifest + Service
Worker + íconos 192/512), así que **no hace falta reescribirla como app
nativa**. El camino es empaquetarla como **TWA (Trusted Web Activity)**.

### Requisito previo
La app tiene que estar publicada en un dominio con **HTTPS** — no funciona
desde un archivo local. La opción más simple, ya que usas Firebase, es
**Firebase Hosting** (gratis):
```
npm install -g firebase-tools
firebase login
firebase init hosting
firebase deploy
```
Esto te da una URL tipo `https://tu-proyecto.web.app`.

### Pasos en PWABuilder

1. Entra a **https://www.pwabuilder.com**
2. Pega la URL pública de tu app (la de Firebase Hosting) y da "Start".
3. PWABuilder analiza el `manifest.json` y el `sw.js` y te da un puntaje —
   con lo que ya tienes (manifest completo, service worker, íconos 192/512
   y 512 maskable) debería salir en verde o casi.
4. Ve a la pestaña **Android** → "Generate Package".
5. Configuración importante que te va a pedir:
   - **Package ID**: el identificador único de la app (ej.
     `mx.productosdelacosta.app`) — una vez publicado en Play Store, esto no
     se puede cambiar.
   - **App name / Launcher name**: como se ve en Play Store y en el ícono.
   - **Signing key**: puedes dejar que PWABuilder genere una nueva (te la
     descarga en un `.zip` — guárdala, la vas a necesitar para cualquier
     actualización futura de la app).
6. Descarga el paquete generado (`.aab` — Android App Bundle).

### Digital Asset Links (paso obligatorio)

Para que Android confirme que la app y el sitio web son del mismo dueño,
necesitas subir un archivo `assetlinks.json` a tu dominio, en:
```
https://tu-proyecto.web.app/.well-known/assetlinks.json
```
PWABuilder te genera este archivo automáticamente en el mismo paso de
"Generate Package" — solo tienes que subirlo a esa ruta exacta (con Firebase
Hosting: ponlo en una carpeta `.well-known/` dentro de tu carpeta de
`public`, y vuelve a hacer `firebase deploy`).

### Publicar en Google Play

1. Crea una cuenta de desarrollador en **Google Play Console**
   (pago único de $25 USD).
2. Sube el `.aab` generado.
3. Llena la ficha de la app (descripción, capturas de pantalla, ícono,
   política de privacidad — obligatoria incluso para apps internas).
4. Puedes publicarla como **prueba interna/cerrada** primero (para tus
   propios repartidores/vendedores) antes de hacerla pública.

### Actualizaciones futuras

Como es una TWA, **no necesitas volver a generar el `.aab` cada vez que
cambies algo del HTML/JS** — Android carga el contenido directo desde tu
sitio web (como un navegador), así que actualizar `index.html` en Firebase
Hosting actualiza la app instalada automáticamente. Solo vuelves a generar
el paquete en PWABuilder si cambias el `manifest.json` (ícono, nombre,
colores) o necesitas subir una nueva versión a Play Console por requisitos
de la tienda.

### iOS

Apple no acepta PWAs empaquetadas como TWA en el App Store. Si más adelante
se necesita estar en la App Store, se requeriría una app nativa o un
wrapper tipo Capacitor — eso es una decisión aparte para cuando llegue ese
punto.
