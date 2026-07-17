```
╔══════════════════════════════════════════════════════════════════╗
║                                                                    ║
║               DASHBOARD SENCILLO CON TAILWIND CSS                 ║
║                                                                    ║
║   Panel de control estático para medir el impacto de campañas     ║
║      de una influencer en Instagram, TikTok y YouTube              ║
║                                                                    ║
║   Autor:            Diego Reynoso                                 ║
║   Tecnologías:       HTML5 semántico · Tailwind CSS v4 (CDN)       ║
║                      · Chart.css · SVG/CSS puro · sin JavaScript   ║
║                                                                    ║
╚══════════════════════════════════════════════════════════════════╝
```

## Descripción del proyecto

Una influencer que empieza a colaborar con marcas necesita consolidar en un solo lugar el rendimiento de sus campañas en varias redes sociales: cuánto está generando en comisiones, qué productos convierten mejor, qué tan bien funcionan sus anuncios y qué plataforma da mejor retorno.

Contexto de negocio de partida:

- 3 productos: Producto A (50 €), Producto B (120 €), Producto C (80 €).
- Comisión del 15 % por cada venta generada.
- Canales a comparar: Instagram, TikTok y YouTube.

Este repositorio **no** es una aplicación conectada a datos reales ni a ninguna API: es una página estática (un único `index.html`) que reproduce, con datos de ejemplo, cómo se vería y se comportaría ese dashboard. Al no tener backend ni JavaScript, no "resuelve" el problema de negocio en el sentido de procesar datos reales — lo que sí resuelve es el diseño completo: la arquitectura visual, la jerarquía de información, la responsividad y la accesibilidad necesarias para que, el día que existan datos reales, el HTML ya esté listo para recibirlos.

Repositorio: [github.com/4GeeksAcademy/Dashboard-Sencillo-Tailwind-CSS-Diego-Reynoso](https://github.com/4GeeksAcademy/Dashboard-Sencillo-Tailwind-CSS-Diego-Reynoso)

---

## Introducción

El proyecto traduce el brief de negocio anterior en un **dashboard de una sola página**, dividido en tres bloques de contenido (indicadores, drivers de rendimiento y detalle operacional) dentro de un chasis fijo de navbar + sidebar, siguiendo el patrón estándar de paneles de datos: navegación siempre visible, contenido con scroll propio.

Todo el maquetado se construyó con **Tailwind CSS v4**, cargado mediante su build para navegador (sin paso de compilación ni Node.js), y todas las gráficas se hicieron sin ninguna librería de JavaScript: con **Chart.css** (tablas HTML semánticas estilizadas como barras), SVG dibujado a mano (funnel, radar) y trucos de CSS puro (`conic-gradient` para los indicadores tipo gauge).

El documento de diseño completo, con la lógica de cada decisión de layout, vive en [`PROPUESTA-LAYOUT.md`](./PROPUESTA-LAYOUT.md).

---

## Características principales

- **Chasis de aplicación**: navbar superior *sticky*, sidebar fija en escritorio y drawer deslizable en móvil — todo con Flexbox, sin ninguna librería de UI.
- **22 KPI Cards** agrupadas en 7 categorías (Volumen, Ingresos, Engagement, Retención, Rendimiento, Satisfacción, Eficiencia), cada una con icono, variación vs. periodo anterior, valor y etiqueta.
- **6 Widgets de "Drivers de rendimiento"**, cada uno con una técnica de gráfica distinta:
  - Funnel de ventas (impresión → clic → lead → venta) en SVG con polígonos.
  - Rendimiento por plataforma y por producto, con **Chart.css** (gráficas de barras sobre `<table>` semánticas).
  - Calidad, con gauges circulares en `conic-gradient`.
  - Actividad por plataforma, con **Chart.css** (barras apiladas).
  - Engagement por plataforma, con un radar dibujado a mano en SVG.
- **Detalle operacional**: tablas de productos, plataformas y campañas (con scroll horizontal en móvil), panel de alertas con color semántico y un widget de rankings con pestañas.
- **100 % responsivo, mobile-first**: KPIs de 2 a 6 columnas, drivers de 1 a 2 columnas, y una vista compacta opcional por pestañas (sin scroll) para pantallas muy grandes (2xl+).
- **Accesible**: estructura semántica completa (`header`, `nav`, `main`, `aside`, `section`, `table`), *skip link*, `aria-label` en controles ambiguos, gráficas SVG con `role="img"` + `<title>`, y una tabla de datos visible detrás de cada gráfica (ninguna cifra vive *solo* dentro de un SVG o un `conic-gradient`).
- **Cero JavaScript**: el drawer móvil, los menús desplegables y las pestañas de rankings están resueltos con los patrones `checkbox + peer`, `<details>/<summary>` y `radio + peer`.

---

## Instalación y requisitos

No hay paso de instalación ni de compilación: Tailwind CSS y Chart.css se cargan en tiempo de ejecución desde su CDN, directamente en el `<head>` de `index.html`.

**Requisitos:**

- Un navegador moderno (Chrome, Firefox, Edge, Safari).
- Conexión a internet la primera vez que se abre la página (para descargar Tailwind CSS v4 y Chart.css desde CDN). No se necesita Node.js, npm ni ningún paso de build.
- Opcional: Python 3, si prefieres servir el proyecto con un servidor local en vez de abrir el archivo directamente.

**Clonar el repositorio:**

```bash
git clone https://github.com/4GeeksAcademy/Dashboard-Sencillo-Tailwind-CSS-Diego-Reynoso.git
cd Dashboard-Sencillo-Tailwind-CSS-Diego-Reynoso
```

---

## Uso

**Opción 1 — Abrir el archivo directamente (la más simple):**

Haz doble clic en `index.html`, o ábrelo desde tu navegador con `Ctrl/Cmd + O`. No requiere ningún servidor.

**Opción 2 — Servidor local con Flask (incluido en el repo):**

```bash
pip3 install flask
python3 server.py
```

Y visita [http://localhost:3000](http://localhost:3000) en el navegador.

> Todos los datos del dashboard (ventas, comisiones, ROI, engagement, etc.) son valores de ejemplo escritos directamente en el HTML — no hay formularios que envíen información ni conexión a ninguna base de datos.

---

## Estructura de carpetas / módulos

```
Dashboard-Sencillo-Tailwind-CSS-Diego-Reynoso/
├── index.html            # El dashboard completo: HTML + Tailwind + Chart.css + SVG (todo en un solo archivo)
├── PROPUESTA-LAYOUT.md    # Documento de diseño: anatomía del layout, tokens, componentes y orden de build
├── Lista-Pasos.txt        # Brief de negocio original y plan de construcción paso a paso
├── server.py              # Servidor de desarrollo local opcional (Flask)
├── styles.css             # Hoja de estilos suelta de una prueba con Tailwind CLI (no se usa; Tailwind corre por CDN)
├── package.json           # Dependencias de un intento con Tailwind CLI (no necesarias para ejecutar el proyecto)
└── .gitignore
```

Al ser un proyecto de una sola página, no hay carpetas de `src`, componentes ni assets: `index.html` concentra toda la estructura, los estilos (vía Tailwind) y las gráficas (vía Chart.css/SVG/CSS).

---

## Contribución

Este es un proyecto personal de práctica, pero las sugerencias son bienvenidas:

1. Haz un fork del repositorio.
2. Crea una rama para tu cambio: `git checkout -b mejora/nombre-del-cambio`.
3. Haz commit de tus cambios con un mensaje claro.
4. Abre un Pull Request describiendo qué cambia y por qué.

Para reportar un problema o proponer una mejora sin escribir código, abre un *issue* en el repositorio.
