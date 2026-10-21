# Chuletario CSS

> Todo lo que necesitas para maquetar en CSS moderno: caja, layout, responsive, en una sola página.

**TL;DR:** CSS es un lenguaje de cascada: gana la regla más específica que aplica y, a igualdad de especificidad, la última declarada. Maqueta con Flexbox para una dimensión y Grid para las dos, mobile-first con `@media (min-width: ...)`, y tipografía fluida con `clamp()`. La referencia es **CSS Level 3 estable** más las funciones modernas ya implementadas en todos los navegadores, verificada el **21 de octubre de 2026**.

**Audiencia:** Este chulet asume que ya sabes CSS básico: selectores, colores, márgenes y el modelo de caja. Si todavía no, ve primero por el [chulet de HTML](./../html/README.md) y vuelve.

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| CSS | Level 3 estable + features modernas | `custom properties`, `clamp()`, `:is()`, `:where()`, `aspect-ratio`, container queries, `:has()`, `color-mix()`, `@layer` |
| Soporte | Baseline *widely available* (2026) | Chrome/Edge 120+, Firefox 121+, Safari 17.4+ |
| Minificador | Lightning CSS `1.x` | Transforma el CSS según los targets que le indiques |
| Linter | Stylelint `16.x` | Detecta reglas inválidas y deprecadas |

```bash
npm install --save-dev stylelint lightningcss
npx stylelint "**/*.css"
```

---

## 🚀 Setup

```html
<!-- styles.css enlazado desde el <head> -->
<link rel="stylesheet" href="styles.css">
```

```bash
mkdir mi-sitio && cd mi-sitio
touch index.html styles.css

# Minificar y transformar según los navegadores objetivo
npx lightningcss --targets="chrome 120, firefox 121, safari 17.4" \
  --minify styles.css -o styles.min.css
```

> Lightning CSS **transforma** el CSS según los targets que le pases. Eso permite usar sintaxis moderna sin preocuparte por los navegadores viejos, porque el output degrada a las equivalencias que sí soportan.

---

## 📖 Cheatsheet

### Cascada y especificidad

| Concepto | Regla |
|----------|-------|
| Especificidad | Tres columnas: ID (`a`), clase/atributo/pseudo-clase (`b`), elemento/pseudo-elemento (`c`) |
| Comparación | De izquierda a derecha: `#id` (1-0-0) gana a `.clase` (0-1-0), que gana a `div` (0-0-1) |
| `:where()` | Especificidad **0-0-0**, ideal para resets y bases |
| `:is()` | Toma la especificidad **más alta** de sus argumentos |

> **Por qué `:where()` existe:** un reset como `.card, .panel { margin: 0 }` tiene especificidad 0-1-0 y le gana a cualquier clase de utilidad que agregues después. Con `:where(.card, .panel)` la especificidad es 0-0-0, así que tus utilidades mandan sin necesidad de `!important`.

### Modelo de caja

| Concepto | Valor | Nota |
|----------|-------|------|
| `box-sizing` | `content-box` (por defecto) | El `width` es solo el contenido; padding y borde se suman |
| `box-sizing` | `border-box` | El `width` incluye padding y borde. Es lo que se usa casi siempre |
| `min-width` / `max-width` | Restricción de rango | `max-width` es lo que hace responsive un contenedor de texto |
| `padding` | Espacio interno | No colapsa con las margens de los hijos |
| `margin` | Espacio externo | Se colapsa con las margens verticales de los hermanos |
| `aspect-ratio` | `16 / 9` | Reserva el espacio correcto y evita el salto de layout |

```css
*, *::before, *::after { box-sizing: border-box; }
```

> **Por qué `border-box`:** con `content-box`, un `width: 300px` más un `padding: 20px` mide 340px. Con `border-box`, `width` es el tamaño final, que es lo que uno espera. El valor por defecto del navegador sigue siendo `content-box`, por eso el reset es casi obligatorio.

### `display`

| Valor | Comportamiento |
|-------|----------------|
| `block` | Ocupa todo el ancho disponible; acepta `width`/`height` |
| `inline` | Se acomoda en el flujo del texto; no acepta `width`/`height` |
| `inline-block` | Flujo de texto, pero acepta `width`/`height` y márgenes verticales |
| `flex` / `inline-flex` | El elemento se convierte en un contenedor flex |
| `grid` | El elemento se convierte en un contenedor grid |
| `none` | No genera caja: desaparece del layout y del árbol de accesibilidad |
| `contents` | El elemento desaparece y sus hijos ocupan su lugar |

> `display: none` saca el elemento del flujo **y** del árbol de accesibilidad: no lo ve un lector de pantalla ni un buscador. `visibility: hidden` lo oculta visualmente pero lo mantiene en el flujo y en la accesibilidad, que suele ser el comportamiento undesired.

### Posicionamiento

| `position` | Comportamiento | Referencia |
|------------|----------------|------------|
| `static` | Por defecto; sigue el flujo normal | |
| `relative` | Se mueve respecto de su posición original, sin sacarlo del flujo | Su propia posición |
| `absolute` | Se saca del flujo | Ancestro con `position` distinto de `static` |
| `fixed` | Se fija respecto de la ventana | Viewport |
| `sticky` | Se comporta como `relative` hasta que su límite lo empuja | El contenedor de scroll |

| Propiedad | Para qué |
|----------|----------|
| `z-index` | Orden de apilamiento dentro del mismo contexto |

> `position: absolute` sin un ancestro posicionado se referencia al **documento inicial**, no al viewport. Es la causa más común de "mi elemento absoluto se va al corner equivocado": el padre necesita `position: relative`.

### Flexbox

| Propiedad del contenedor | Para qué |
|--------------------------|----------|
| `flex-direction` | `row` (por defecto), `column`, `row-reverse`, `column-reverse` |
| `flex-wrap` | `nowrap` (por defecto), `wrap`, `wrap-reverse` |
| `justify-content` | Eje principal: `flex-start`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items` | Eje transversal: `stretch` (por defecto), `flex-start`, `center`, `flex-end`, `baseline` |
| `gap` | Separación entre elementos: `row-gap`, `column-gap` |

| Propiedad del item | Para qué |
|-------------------|----------|
| `flex: 1` | Crece y se encoge desde su base; ocupa el espacio sobrante |
| `flex: none` | No crece, no se encoge, base automática |

> `gap` en Flexbox está soportado en todos los navegadores modernos. Antes se resolvía con `margin` en los hijos, que dejaba espacio al final del contenedor y se acumulaba con `justify-content: space-between`.

### CSS Grid

| Propiedad | Para qué | Ejemplo |
|----------|----------|---------|
| `grid-template-columns` | Definición de columnas | `repeat(3, 1fr)` |
| `grid-column` / `grid-row` | Posición explícita del item | `1 / 3`, `span 2` |
| `grid-template-areas` | Mapa de áreas con nombre | `"a a" "b c"` |
| `grid-area` | Coloca el item en un área con nombre | `header`, `contenido` |
| `gap` | Separación entre tracks | `1.5rem` |

| Función | Para qué |
|---------|----------|
| `repeat(n, patron)` | Repite un patrón de pistas |
| `minmax(min, max)` | Rango de tamaño: mínimo fijo, máximo fluido |
| `auto-fit` vs `auto-fill` | `auto-fill` conserva las pistas vacías; `auto-fit` las colapsa y deja que las restantes crezcan |
| `fr` | Fracción del espacio sobrante |

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: 1.5rem;
}
```

> **`auto-fit` vs `auto-fill`:** con un contenedor de 1000px y `minmax(16rem, 1fr)`, ambas dan 3 columnas. La diferencia aparece cuando sobran pistas: `auto-fill` deja las vacías (y con ellas el hueco), `auto-fit` las colapsa a 0 y estira las que tienen contenido. Para un layout de tarjetas, `auto-fit` es lo que quieres.

### Custom properties (variables)

| Aspecto | Detalle |
|---------|---------|
| Declaración | `--color-fondo: #fff;` en cualquier elemento |
| Valor por defecto | `var(--color-fondo, #fff)` |

```css
:root {
  --espaciado: 1rem;
  --radio: 8px;
}

.card {
  padding: var(--espaciado);
  border-radius: var(--radio, 4px);
}
```

### Media queries y mobile-first

| Síntaxis | Significado |
|---------|-------------|
| `@media (min-width: 48rem)` | Se aplica **desde** ese ancho hacia arriba. Es el patrón mobile-first |
| `@media (max-width: 47.99rem)` | Se aplica hasta ese ancho: condicionales por el camino contrario |
| `@media (prefers-reduced-motion: reduce)` | Respeta la preferencia de menos movimiento del sistema |

```css
.container { padding: 1rem; }

@media (min-width: 48rem) {
  .container { padding: 2rem; max-width: 72rem; margin-inline: auto; }
}
```

> **Por qué mobile-first y no `max-width`:** con `min-width` escribes las reglas base sin media query y las mejoras van en bloques condicionales. Con `max-width` tienes que escribir primero la versión de escritorio y después "deshacerla" para móvil, y el orden del CSS queda al revés de la lógica de la página.

### Funciones modernas

| Función | Qué hace | Ejemplo |
|---------|----------|---------|
| `calc()` | Combina longitudes con `+ - * /` | `calc(100% - 2rem)` |
| `min()` | El menor de varios valores | `min(100%, 40rem)` |
| `max()` | El mayor de varios valores | `max(1rem, 2vw)` |
| `clamp(min, preferido, max)` | El preferido, limitado entre min y max | `clamp(1rem, 2.5vw, 2rem)` |

```css
h1 { font-size: clamp(1.5rem, 1rem + 2.5vw, 3rem); }
```

### Transiciones y animaciones

| Concepto | Para qué | Nota |
|----------|----------|-----|
| `transition` | Anima un cambio de valor | `transition: color 200ms ease` |
| `transform` / `opacity` | Las dos propiedades que se animan sin repintar | Preferilas para animaciones de 60fps |
| `@keyframes` | Define los estados de una animación | |
| `animation` | Atajo de una animación | `animation: fade 300ms ease forwards` |

```css
@media (prefers-reduced-motion: no-preference) {
  .card { transition: transform 200ms ease, box-shadow 200ms ease; }
  .card:hover { transform: translateY(-2px); }
}
```

### `z-index` y contextos de apilamiento

| Regla | Detalle |
|------|---------|
| Solo aplica a elementos posicionados | `position` distinto de `static`, o `flex`/`grid` items |
| Contexto de apilamiento | Un `z-index` alto dentro de un contexto no puede escapar de él |
| Crea un contexto | `position` + `z-index` (salvo `auto`), `opacity < 1`, `transform`, `filter`, `will-change`, `contain: paint` |

> El error clásico: un modal con `z-index: 9999` queda debajo de un header con `z-index: 100` porque el header crea su propio contexto en un ancestro común. La solución no es subir el número: es controlar los contextos con `isolation: isolate` y una escala de valores definida en variables.

### Centrado

| Técnica | Cuándo usarla |
|---------|---------------|
| `display: grid; place-items: center;` | Centrar un hijo en ambas dimensiones. La forma más simple y robusta |
| `display: flex; align-items: center; justify-content: center;` | Igual, con menos control fino |
| `margin-inline: auto` | Centrar horizontalmente un bloque con `width` o `max-width` definido |
| `position: absolute; inset: 0; margin: auto;` | Centrar un hijo con `position: absolute` |
| `place-content: center` | Centrar el contenido del grid cuando sobra espacio |

```css
.centrado { display: grid; place-items: center; min-height: 100dvh; }
```

> `100dvh` (dynamic viewport height) es preferible a `100vh` en móvil: `vh` mide el viewport con la barra de direcciones visible, y al ocultarse deja un hueco. `dvh` se recalcula cuando la barra cambia de estado.

### Temas con `prefers-color-scheme`

| Pieza | Para qué |
|-------|----------|
| `@media (prefers-color-scheme: dark)` | Aplica estilos cuando el sistema pide tema oscuro |

```css
:root {
  color-scheme: light dark;
  --fondo: #ffffff;
  --texto: #1a1a1a;
  --acento: #2563eb;
}

@media (prefers-color-scheme: dark) {
  :root {
    --fondo: #101418;
    --texto: #e8eaed;
    --acento: #60a5fa;
  }
}

body { background: var(--fondo); color: var(--texto); }
```

> El patrón correcto es declarar los **tokens** en `:root` y usarlos en todas partes. Si escribes `color: white` en un componente, ese componente no se adapta al tema oscuro y terminas escribiendo una media query por cada componente.

---

## 💻 Ejemplos

### Grid responsive de tarjetas

```html
<ul class="cards">
  <li class="card"><h2>Bash</h2><p>Scripting en la terminal.</p></li>
  <li class="card"><h2>HTML</h2><p>Estructura y semántica.</p></li>
  <li class="card"><h2>CSS</h2><p>Layout y responsive.</p></li>
  <li class="card"><h2>Go</h2><p>Backend con concurrencia.</p></li>
  <li class="card"><h2>Python</h2><p>Automatización y datos.</p></li>
  <li class="card"><h2>Linux</h2><p>El sistema por dentro.</p></li>
</ul>
```

```css
:root {
  --gap: 1.5rem;
  --card-bg: #ffffff;
  --card-border: #e2e5e9;
}

.cards {
  margin: 0;
  padding: 0;
  list-style: none;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: var(--gap);
}

.card {
  padding: 1.25rem;
  border: 1px solid var(--card-border);
  border-radius: 12px;
  background: var(--card-bg);
}
```

### Navbar con Flexbox

```html
<nav class="navbar">
  <a class="brand" href="/">Chuletario</a>
  <ul class="nav-links">
    <li><a href="/bash">Bash</a></li>
    <li><a href="/html">HTML</a></li>
    <li><a href="/css">CSS</a></li>
  </ul>
</nav>
```

```css
:root {
  --navbar-bg: #101418;
  --navbar-fg: #f5f7fa;
  --navbar-gap: 1.5rem;
}

.navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--navbar-gap);
  padding: 0.75rem 1.5rem;
  background: var(--navbar-bg);
  color: var(--navbar-fg);
}

.nav-links {
  display: flex;
  align-items: center;
  gap: var(--navbar-gap);
  margin: 0;
  padding: 0;
  list-style: none;
}

@media (max-width: 40rem) {
  .navbar {
    flex-direction: column;
    align-items: flex-start;
  }
}
```

### Escala tipográfica fluida con `clamp()`

```css
:root {
  /* Escala modular: cada tamaño es 1.25x el anterior */
  --step-0: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  --step-2: clamp(1.563rem, 1.4rem + 0.8vw, 2rem);
  --step-3: clamp(1.953rem, 1.7rem + 1.2vw, 2.5rem);
  --step-4: clamp(2.441rem, 2.1rem + 1.7vw, 3.25rem);
}

body {
  font-size: var(--step-0);
  line-height: 1.6;
}

h1 { font-size: var(--step-4); line-height: 1.15; margin-block: 0 1rem; }
h2 { font-size: var(--step-3); margin-block: 2rem 0.75rem; }
h3 { font-size: var(--step-2); margin-block: 1.5rem 0.5rem; }

p, ul, ol { max-width: 65ch; }
```

Salida esperada: cada encabezado crece con el ancho del viewport entre los límites de `clamp()`, sin ninguna media query.

```text
h1 @ 320px  ->  2.441rem  (39px)
h1 @ 1024px ->  3.0rem    (48px)
h1 @ 1600px ->  3.25rem   (52px)
```

> `max-width: 65ch` en los párrafos es la regla de legibilidad más subestimada: una línea de texto mide entre 45 y 75 caracteres. Más de eso, el ojo pierde el hilo al volver al inicio de la línea.

### Layout centrado

```css
body {
  margin: 0;
  min-height: 100dvh;
  display: grid;
  place-items: center;
  background: #0f1115;
  color: #f5f7fa;
  font-family: system-ui, sans-serif;
}

.panel {
  width: min(100% - 2rem, 30rem);
  padding: 2rem;
  border-radius: 16px;
  background: #1a1f26;
}
```

Salida esperada: el panel queda centrado vertical y horizontalmente, con 1rem de margen a cada lado en pantallas angostas y un ancho máximo de 30rem (480px) en pantallas grandes.

```text
320px  ->  panel de 288px, centrado
1440px ->  panel de 480px, centrado
```

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| No resetear `box-sizing` | `width: 300px` con `padding: 20px` mide 340px | `*, *::before, *::after { box-sizing: border-box; }` |
| Usar `100vh` en móvil | La barra de direcciones deja un hueco de varios píxeles | `100dvh` o `100svh` |
| Márgenes verticales que se colapsan | Se fusionan y el cálculo de alturas falla | `gap`, `padding` o `display: flow-root` en el contenedor |
| `position: absolute` sin ancestro posicionado | Se posiciona respecto del documento inicial | `position: relative` en el padre |
| `z-index: 99999` para tapar algo | No funciona si el otro elemento está en otro contexto de apilamiento | Escala de valores con variables y `isolation: isolate` |
| `transition: all` | Anima propiedades que no quieres y fuerza repintados | Transitiona solo `transform` y `opacity` |
| `display: none` para ocultar | El elemento desaparece del árbol de accesibilidad y del layout | `visibility: hidden` o el atributo `hidden` |
| `100%` de `height` sin `height` en el padre | El padre no tiene altura y el hijo colapsa a 0 | `min-height` en el contenedor, o unidades `dvh` |
| `repeat(3, 1fr)` fijo | Se rompe en móvil porque no hay `minmax` ni `auto-fit` | `repeat(auto-fit, minmax(16rem, 1fr))` |
| `clamp()` sin unidad fluida en el valor preferido | `clamp(1rem, 2rem, 3rem)` es constante | El valor preferido debe incluir `vw` o `vmin` |
| `@media (max-width: ...)` para móvil | El CSS de escritorio se aplica primero y después se deshace | Mobile-first: base sin media query y `min-width` para ampliar |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| Grid sobre Flexbox | Dos dimensiones, `auto-fit` y áreas con nombre sin hacks | Curva de aprendizaje mayor; overkill para una fila o una columna |
| Flexbox sobre floats | Sin clears, sin problemas de altura, alineación en ambos ejes | No distribuye en dos dimensiones |
| Custom properties sobre variables en preprocesador | Cambian en runtime, se heredan, permiten temas | Sin tipado ni valores por defecto en el lenguaje |
| `prefers-color-scheme` con media query | Un solo origen de verdad para los tokens | Hay que repetir la media query; `light-dark()` es más corto pero menos conocido |
| Mobile-first sobre desktop-first | Menos código de sobrescritura, más rápido en móvil | Obliga a pensar el layout base antes que la versión grande |
| `border-box` sobre `content-box` | `width` es el tamaño final, cálculos previsibles | Rompe layouts que dependían del comportamiento por defecto |
| `clamp()` sobre media queries para tipografía | Escala continua, sin saltos entre breakpoints | El valor final es menos predecible que un tamaño fijo |
| `@layer` sobre orden por especificidad | El orden de precedencia es explícito y legible | Los navegadores sin soporte ignoran la capa: hay que verificar el fallback |
| Transiciones de `transform`/`opacity` | Se animan en el compositor, sin repintado | No sirve para animar `width`, `height` o `top` |

---

## 📚 Further reading

- [MDN: CSS reference](https://developer.mozilla.org/en-US/docs/Web/CSS) — la referencia práctica más completa
- [Learn CSS (web.dev)](https://web.dev/learn/css) — curso con ejercicios y rutas por nivel
- [CSS Grid Layout (W3C)](https://www.w3.org/TR/css-grid-2/) — la especificación de Grid, con diagramas de referencia
- [MDN: Container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries) — consultas por tamaño del contenedor, no del viewport
- [Can I Use](https://caniuse.com/) — soporte real por navegador para cada feature
