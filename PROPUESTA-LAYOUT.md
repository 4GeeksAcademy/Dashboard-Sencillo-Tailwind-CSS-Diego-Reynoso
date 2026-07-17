# Propuesta de Layout — Dashboard de Influencer & Marcas

> Rol: Arquitecto UX/UI + Frontend Senior
> Stack objetivo: **HTML + Tailwind CSS v4** — mobile-first, sin JavaScript de terceros
> Objetivo: consolidar el impacto de anuncios multi-plataforma (Instagram, TikTok, YouTube…) y responder a comisiones, productos top, conversión, retorno por plataforma y engagement.

---

## 0. Stack y configuración

### 0.1 Alcance

El proyecto se compone únicamente de archivos `.html` y una hoja de estilos generada por Tailwind. Las gráficas se construyen con SVG inline y CSS; la interactividad, con HTML nativo.

### 0.2 Cómo cargar Tailwind

**Opción A — Browser build (prototipado, sin paso de compilación):**

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

Compila en el navegador. Válido para maquetar; no usar en producción.

**Opción B — CSS compilado (entregable final):**

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./dist/app.css --watch --minify
```

```html
<link rel="stylesheet" href="./dist/app.css" />
```

El HTML sigue siendo HTML plano: solo enlaza una hoja de estilos estática. Sin runtime ni dependencias en el cliente.

### 0.3 Configuración

Toda la configuración vive en el CSS:

```css
@import "tailwindcss";

@theme {
  /* Colores de marca por plataforma (para las gráficas) */
  --color-ig: oklch(0.65 0.22 12);
  --color-ig-alt: oklch(0.75 0.16 60);
  --color-tiktok: oklch(0.78 0.14 195);
  --color-tiktok-alt: oklch(0.68 0.22 350);
  --color-youtube: oklch(0.6 0.23 27);

  /* Acento y semánticos del dashboard */
  --color-brand: var(--color-indigo-600);
  --color-positive: var(--color-emerald-500);
  --color-negative: var(--color-rose-500);
  --color-warning: var(--color-amber-500);
}
```

Cada token genera automáticamente sus utilidades: `bg-ig`, `text-youtube`, `border-brand`, `text-positive`.

**¿Dónde vive este CSS?** Depende de cómo cargues Tailwind (§0.2):

- **Opción A (browser build):** el `@import "tailwindcss"` y el bloque `@theme` deben ir dentro de una etiqueta especial `<style type="text/tailwindcss">`, **no** en un `<link>` normal. Si se pone en un `<style>` corriente, el browser build no lo procesa y las utilidades personalizadas (`bg-ig`, `text-positive`, `shadow-xs`, `text-brand`…) no se generan.

  ```html
  <head>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style type="text/tailwindcss">
      @import "tailwindcss";
      @theme {
        --color-brand: var(--color-indigo-600);
        /* …resto de tokens… */
      }
    </style>
  </head>
  ```

- **Opción B (CLI):** el mismo CSS va en `src/input.css`, se compila a `dist/app.css` y el HTML solo enlaza `<link rel="stylesheet" href="./dist/app.css" />`.

---

## 1. Estructura Principal (Layout General)

### 1.1 Anatomía del contenedor global

El dashboard usa un patrón **"Sidebar fija + Área de contenido con scroll"**, que es el estándar para paneles de datos porque mantiene la navegación siempre visible y deja el foco en las métricas.

```
┌───────────────────────────────────────────────────────────┐
│  NAVBAR (sticky top)                                      │
├──────────┬────────────────────────────────────────────────┤
│          │  MAIN (scroll)                                 │
│ SIDEBAR  │   ├─ Bloque Superior   → KPIs                  │
│ (aside)  │   ├─ Bloque Intermedio → Drivers (gráficas)    │
│          │   └─ Bloque Inferior   → Tablas / alertas      │
│          │                                                │
└──────────┴────────────────────────────────────────────────┘
```

### 1.2 Sistema de rejilla

Se combinan **Flexbox** (esqueleto: navbar / sidebar / main) y **CSS Grid** (rejilla interna de cada bloque de contenido). Regla mental: _Flexbox para el chasis, Grid para los datos._

| Zona            | Técnica          | Clases Tailwind base                                   |
| --------------- | ---------------- | ------------------------------------------------------ |
| Chasis app      | Flexbox          | `flex min-h-screen`                                    |
| Sidebar         | Ancho fijo       | `w-64 shrink-0`                                        |
| Contenido       | Columna flexible | `flex-1 flex flex-col`                                 |
| Rejilla de KPIs | Grid             | `grid grid-cols-2 md:grid-cols-4 xl:grid-cols-6 gap-4` |
| Rejilla Drivers | Grid             | `grid grid-cols-1 lg:grid-cols-2 gap-6`                |
| Tablas          | Ancho completo   | `w-full overflow-x-auto`                               |

### 1.3 Comportamiento responsivo (mobile-first)

| Breakpoint   | Rango      | Comportamiento                                                                                      |
| ------------ | ---------- | --------------------------------------------------------------------------------------------------- |
| Base (móvil) | `< 768px`  | Sidebar oculta (drawer). KPIs en 2 columnas. Gráficas apiladas 1 col. Tablas con scroll horizontal. |
| `md`         | `≥ 768px`  | KPIs a 4 columnas. Drivers empiezan a convivir.                                                     |
| `lg`         | `≥ 1024px` | Sidebar visible fija. Drivers a 2 columnas.                                                         |
| `xl`         | `≥ 1280px` | KPIs hasta 6 columnas. Máximo aprovechamiento horizontal.                                           |

Contenedor de contenido con respiración lateral: `max-w-[1600px] mx-auto px-4 md:px-6 lg:px-8`.

**Drawer móvil:** patrón checkbox + `peer`, o `<details>`/`<summary>`, o un `:target` con ancla.

```html
<input id="nav" type="checkbox" class="peer hidden" />
<label for="nav" class="lg:hidden">☰</label>
<aside
  class="-translate-x-full peer-checked:translate-x-0 transition lg:translate-x-0 …"
>
  …
</aside>
```

---

## 2. Navbar (Barra de Navegación)

Barra superior **sticky**, altura fija (`h-16`), separada por un borde inferior sutil.

```
┌────────────────────────────────────────────────────────────────────────────┐
│ [≡] [Logo]   Dashboard de Impacto      [Buscar] [Rango] [Alertas] [Perfil] │
└────────────────────────────────────────────────────────────────────────────┘
   IZQUIERDA      CENTRO                        DERECHA
```

**Distribución** (`flex items-center justify-between`):

- **Izquierda:** control hamburguesa (solo móvil, `lg:hidden`), logo/avatar de la influencer y nombre de marca personal.
- **Centro (opcional):** título de la vista actual o buscador global.
- **Derecha — acciones rápidas:**
  - **Selector de rango de fechas** (7d / 30d / 90d / personalizado) → enlaces (`?rango=30d`) o un `<details>` con opciones.
  - **Filtro de plataforma** (chips: Todas · IG · TikTok · YouTube) → enlaces con estado activo por clase.
  - Icono de notificaciones/alertas con badge numérico.
  - Menú de usuario (perfil, ajustes, exportar reporte) con `<details>` + `<summary>`.

**Comportamiento:** `sticky top-0 z-50 bg-white/90 backdrop-blur border-b`. En móvil, buscador y filtros colapsan dentro del drawer.

> Los filtros y rangos son **navegación**: cada estado es una URL/vista.

---

## 3. División del Contenido

Cada bloque es una `<section>` con el mismo esqueleto (ver §4.3): cabecera con título + subtítulo/acción, y una rejilla de contenido.

### 3.1 🟦 Bloque Superior — Indicadores Principales (KPIs)

Fila(s) de **KPI Cards** compactas. Es el "vistazo de 5 segundos". Rejilla `grid-cols-2 md:grid-cols-4 xl:grid-cols-6`.

Agrupados por categoría (una fila por grupo):

- **Volumen:** ventas · inscripciones · usuarios activos · impresiones
- **Ingresos:** revenue · comisiones (15%) · MRR · precio medio de venta
- **Engagement:** tasa de engagement · interacciones totales · tasa de conversión
- **Retención:** churn · permanencia · tasa de compleción
- **Rendimiento:** conversiones totales · CTR · tasa de conversión
- **Satisfacción:** NPS · CSAT
- **Eficiencia:** coste por resultado · margen · tiempos

> Cada KPI muestra valor, etiqueta y **variación vs. periodo anterior** (▲ verde / ▼ rojo).

### 3.2 🟨 Bloque Intermedio — Drivers (factores que explican el resultado)

Rejilla de **Widget Cards con visualizaciones**. `grid-cols-1 lg:grid-cols-2` (algún widget ancho a `lg:col-span-2`).

Todas las gráficas se construyen con SVG inline o CSS (ver §4.4):

- **Funnel de ventas** (Impresión → Clic → Lead → Venta) — SVG con polígonos.
- **Rendimiento por plataforma** — barras comparando IG / TikTok / YouTube (ingresos, coste, ROI).
- **Calidad** — ratio de leads cualificados, attendance rate, pass rate (gauges con `conic-gradient`).
- **Rendimiento por producto** — qué producto genera más comisiones y mejor conversión (barras A/B/C).
- **Actividad** — posts, stories, reels, videos por plataforma (barras apiladas con Flexbox).
- **Engagement por plataforma** — likes, comentarios, compartidos, guardados (radar en SVG o barras).

### 3.3 🟩 Bloque Inferior — Detalles Operacionales

**Tablas, listados y alertas.** Ancho completo, con scroll horizontal en móvil (`overflow-x-auto`).

- **Tabla de Productos:** precio · comisiones generadas · conversiones · ROI por producto.
- **Tabla de Plataformas:** alcance · engagement · conversiones · mejor plataforma por métrica.
- **Tabla de Campañas:** fechas · productos promocionados · resultados · rendimiento.
- **Panel de Alertas:** caídas fuertes de conversión, picos, anomalías (tarjetas de color semántico).
- **Listados con filtros:** top productos · top plataformas · top campañas · oportunidades de mejora.

---

## 4. Componentes Reutilizables

Consistencia = piezas base que se repiten en todo el tablero.

### 4.1 Cards (Tarjetas)

Contenedor genérico: `rounded-2xl bg-white border border-slate-200 p-5 shadow-xs hover:shadow-sm transition`.

**A) KPI Card** (bloque superior)

```
┌──────────────────────────┐
│[icono]         ▲ +12,4%  │  ← icono categoría + variación
│                          │
│ 1.240 €                  │  ← valor (text-2xl font-bold)
│ Comisiones generadas     │  ← etiqueta (text-sm text-slate-500)
└──────────────────────────┘
```

Estructura interna: `icono` + `badge de tendencia` + `valor` + `etiqueta`. Sin imagen; el icono comunica la categoría (SVG inline).

**B) Content Card / Widget Card** (bloque intermedio)

```
┌──────────────────────────────┐
│ Título del widget       [⋯]  │  ← header (título + menú/acción)
│ Subtítulo o rango            │
├──────────────────────────────┤
│                              │
│        [ GRÁFICA ]           │  ← cuerpo (SVG / CSS / lista)
│                              │
├──────────────────────────────┤
│ Leyenda · footer opcional    │
└──────────────────────────────┘
```

Estructura: `header (título + acción)` → `body (gráfica/contenido)` → `footer (leyenda o CTA)`. Los botones/enlaces viven en el header (`<details>` con `⋯`) o en el footer como CTA (`ver detalle`).

### 4.2 Cajas de Widgets (elementos interactivos / info rápida)

Piezas pequeñas que se incrustan dentro de las cards o en la sidebar:

- **Stat pill / Badge de tendencia:** `inline-flex items-center gap-1 rounded-full px-2 py-0.5 text-xs` en verde/rojo/ámbar.
- **Mini-selector / Toggle de rango:** enlaces segmentados (7d · 30d · 90d) con estado activo.
- **Chips de filtro de plataforma:** enlaces con estado activo (`aria-current="page"` + `aria-[current=page]:bg-brand`).
- **Progress bar / Gauge:** para tasas (conversión, compleción, churn).
- **Alert box:** icono + mensaje + severidad (info / warning / danger) con color de fondo semántico.
- **Sparkline:** micro-gráfica SVG de tendencia dentro de una KPI card.

### 4.3 Estructura por Secciones (esqueleto común)

Todas las secciones comparten el mismo molde para que el ojo no se pierda:

```html
<section class="mb-8">
  <!-- Cabecera de sección -->
  <header class="mb-4 flex items-end justify-between">
    <div>
      <h2 class="text-lg font-semibold text-slate-800">Título de sección</h2>
      <p class="text-sm text-slate-500">Subtítulo / contexto</p>
    </div>
    <a href="#" class="text-sm text-brand">Acción / Ver todo</a>
  </header>

  <!-- Rejilla de contenido -->
  <div class="grid gap-4 …">
    <!-- Cards / Widgets / Tablas -->
  </div>
</section>
```

Reglas de consistencia:

- **Márgenes:** separación vertical entre secciones `mb-8`; `gap-4` entre KPIs, `gap-6` entre widgets grandes.
- **Títulos:** `text-lg font-semibold`; **subtítulos:** `text-sm text-slate-500`.
- **Radio y sombra:** `rounded-2xl` + `shadow-xs` en todas las tarjetas.
- **Espaciado interno:** `p-5` en cards, `p-4` en widgets pequeños.

### 4.4 Gráficas

| Gráfica          | Técnica HTML/CSS                                                              |
| ---------------- | ----------------------------------------------------------------------------- |
| Barras           | `flex items-end gap-2` + altura por `style="height:64%"` o `h-*`              |
| Barras apiladas  | Columna `flex flex-col` con segmentos de altura proporcional                  |
| Progress / ratio | `<div class="h-2 rounded-full bg-slate-200"><div class="h-full bg-brand">`    |
| Gauge / donut    | `<div style="background: conic-gradient(var(--color-brand) 72%, #e2e8f0 0)">` |
| Sparkline        | `<svg viewBox="0 0 100 30"><polyline points="…" fill="none" stroke="…">`      |
| Funnel           | `<svg>` con `<polygon>` por etapa                                             |
| Radar            | `<svg>` con `<polygon>` sobre ejes `<line>`                                   |

Los valores se calculan al escribir el HTML (dashboard estático) mediante `style` inline o clases arbitrarias (`h-[64%]`). Accesibilidad: `role="img"` + `<title>` en cada SVG, y la tabla equivalente como fuente de verdad.

---

## 5. Sistema de diseño (tokens)

| Token            | Valor                    | Uso                   |
| ---------------- | ------------------------ | --------------------- |
| Fondo app        | `bg-slate-50`            | Lienzo general        |
| Superficie       | `bg-white`               | Cards                 |
| Texto principal  | `text-slate-800`         | Títulos/valores       |
| Texto secundario | `text-slate-500`         | Etiquetas             |
| Acento           | `brand` (indigo-600)     | Enlaces, activos, CTA |
| Positivo         | `positive` (emerald-500) | Tendencia ▲, buen ROI |
| Negativo         | `negative` (rose-500)    | Tendencia ▼, alertas  |
| Aviso            | `warning` (amber-500)    | Anomalías             |
| Radio            | `rounded-2xl`            | Tarjetas              |
| Sombra           | `shadow-xs`              | Elevación base        |

Colores de marca por plataforma para gráficas: `ig`, `tiktok`, `youtube` — definidos en el bloque `@theme` de §0.3.

---

## 6. Orden de implementación sugerido

1. **Setup:** `@import "tailwindcss";` + bloque `@theme` con los tokens.
2. Chasis (navbar + sidebar + main) con Flexbox.
3. Componente **KPI Card** reutilizable → poblar bloque superior.
4. **Widget Card** shell → gráficas en SVG/CSS (§4.4).
5. Tablas del bloque inferior + panel de alertas.
6. Responsividad (drawer con `peer`, breakpoints) y filtros globales del navbar como navegación.
