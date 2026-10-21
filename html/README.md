# Chuletario HTML

> Todo lo que necesitas para escribir HTML5 válido, semántico y accesible, en una sola página.

**TL;DR:** HTML5 es un estándar vivo (living standard) definido por WHATWG, no una versión congelada. Un documento válido y bien formado lleva `<!doctype html>`, `<html lang>`, `<meta charset="utf-8">` y `<meta name="viewport">` en el `<head>`, y usa elementos semánticos (`header`, `nav`, `main`, `section`, `article`, `aside`, `footer`) en vez de `<div>` con clases. La referencia es el **WHATWG HTML living standard**, verificado el **21 de octubre de 2026**.

**Audiencia:** Este chulet asume que ya sabes qué es una etiqueta y un atributo. No hace falta que sepas accesibilidad ni semántica: justamente eso es lo que vas a aprender aquí. Si vienes de un framework, revisa igual la sección de [Formularios](#formularios) y la de [Accesibilidad](#accesibilidad).

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| HTML | WHATWG living standard | No hay "HTML 5.3": el estándar se actualiza de forma continua |
| Navegadores | Chrome/Edge 120+, Firefox 121+, Safari 17+ | Todos los elementos de este chulet tienen soporte amplio |
| Validador | Nu HTML Checker | `validator.w3.org/nu` — valida contra el living standard |
| Validador CLI | `vnu` (validator.nu) | `npm install -g vnu-jar` |
| Editor | Cualquiera con soporte de HTML5 | VS Code + extensión "HTML Language Support" |

---

## 🚀 Setup

```bash
# Crear el proyecto mínimo
mkdir mi-sitio && cd mi-sitio
touch index.html

# Instalar el validador como comando local
npm install -g vnu-jar

# Validar: devuelve código de salida 1 si hay errores
vnu --errors-only index.html
```

Salida con un documento válido:

```text
$ vnu --errors-only index.html; echo "exit=$?"
exit=0
```

> El validador de Nu es el mismo motor que usa `validator.w3.org/nu`. Sin instalar nada se puede consultar por HTTP: `curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @index.html "https://validator.w3.org/nu/?out=json" | jq '.messages | length'`.

---

## 📖 Cheatsheet

### Esqueleto del documento

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Título de la página</title>
  </head>
  <body>
    <!-- Contenido -->
  </body>
</html>
```

| Parte | Regla | Por qué importa |
|-------|-------|-----------------|
| `<!doctype html>` | Siempre, en minúsculas y sin versión | Activa el modo estándar del parser; sin él, el navegador aplica las reglas de *quirks* del modo legacy |
| `<html lang="es">` | Idioma del contenido | Los lectores de pantalla cambian la pronunciación; los buscadores lo usan para segmentar |
| `<meta charset>` | En los primeros 1024 bytes | Si llega tarde, el navegador reinterpreta bytes ya decodificados y aparecen caracteres rotos |
| `<meta name="viewport">` | `width=device-width, initial-scale=1` | Sin esto, el navegador móvil renderiza a ~980px y hace zoom solo |

### Metadatos del `<head>`

| Etiqueta | Propósito | Ejemplo |
|----------|-----------|---------|
| `charset` | Codificación de caracteres | `<meta charset="utf-8">` |
| `viewport` | Escala en dispositivos móviles | `<meta name="viewport" content="width=device-width, initial-scale=1">` |
| `description` | Resumen para buscadores (hasta ~160 caracteres) | `<meta name="description" content="...">` |
| `robots` | Directivas para indexadores | `<meta name="robots" content="noindex">` |
| Open Graph | Vista previa al compartir en redes | `<meta property="og:title" content="...">` |
| Twitter Card | Vista previa en Twitter/X | `<meta name="twitter:card" content="summary_large_image">` |
| `theme-color` | Color de la barra del navegador en Android | `<meta name="theme-color" content="#1a1a1a">` |

> `description` y `robots` son `<meta name=...>`; los de Open Graph usan `<meta property=...>`. No son intercambiables: los buscadores leen `name`, las redes leen `property`.

### Elementos semánticos y *landmarks*

| Elemento | Rol | Qué va adentro |
|----------|-----|----------------|
| `<header>` | `banner` | Cabecera del documento o de una sección |
| `<nav>` | `navigation` | Menú de navegación principal |
| `<main>` | `main` | Contenido único de la página. **Uno solo por documento** |
| `<section>` | `region` | Agrupación temática con título propio |
| `<article>` | `article` | Unidad autocontenida y re-publicable (un post) |
| `<aside>` | `complementary` | Contenido tangencial: glosario, enlaces relacionados |
| `<footer>` | `contentinfo` | Pie de página, copyright, enlaces legales |

> `<main>` es el único punto de entrada del contenido principal. Los lectores de pantalla permiten saltar directo a él, y por eso debe ser único: si hay dos `<main>`, la navegación por landmarks se vuelve ambigua.

### Jerarquía de encabezados

| Nivel | Uso | Error típico |
|-------|-----|---------------|
| `<h1>` | Título del documento. Uno por página | Poner un `<h1>` por sección |
| `<h2>` | Secciones principales | Saltar de `<h1>` a `<h3>` |
| `<h3>`…`<h6>` | Subniveles | Elegir el nivel por tamaño visual |

> El nivel del encabezado define la estructura semántica, no el tamaño en pantalla. Si un `<h2>` tiene que verse como un `<h1>`, se cambia el CSS, no el HTML. Saltarse niveles rompe la navegación por encabezados de los lectores de pantalla.

### Formularios

| Control | Etiqueta | Atributos clave |
|---------|----------|-----------------|
| `<label for="id">` | Etiqueta visible | `for` debe coincidir con el `id` del control |
| `<input type="text">` | Texto de una línea | `id`, `name`, `value`, `placeholder` (no reemplaza la etiqueta) |
| `<input type="email">` | Correo | Validación de formato en el navegador |
| `<input type="checkbox">` | Casilla | Comparte `name` entre opciones del mismo grupo |
| `<input type="radio">` | Opción única | Comparte `name` para formar el grupo |
| `<select>` | Lista desplegable | `<option value="...">`; `multiple` para selección múltiple |
| `<textarea>` | Texto multilínea | `rows`, `cols` |

| Atributo de validación | Qué hace |
|----------------------|----------|
| `required` | El campo no puede quedar vacío |
| `pattern="[0-9]{5}"` | Expresión regular (JavaScript flavor) |

```html
<form action="/subscribe" method="post">
  <label for="email">Correo electrónico</label>
  <input type="email" id="email" name="email" required autocomplete="email">
  <button type="submit">Suscribirse</button>
</form>
```

### Accesibilidad

| Herramienta | Para qué | Ejemplo |
|-------------|----------|---------|
| `alt` | Texto alternativo de una imagen | `alt="Gráfico de ventas del primer trimestre"` |
| `alt=""` | Imagen decorativa: el lector la ignora | `alt=""` en un separador |
| `aria-label` | Nombre accesible de un control sin texto visible | `aria-label="Cerrar"` |
| `aria-describedby` | Descripción adicional (errores, ayuda) | `aria-describedby="error-email"` |
| `aria-live` | Anuncia cambios dinámicos | `aria-live="polite"` |
| Skip link | Salta al contenido principal | `<a href="#main">` como primer elemento de `<body>` |
| `<fieldset>` / `<legend>` | Agrupa controles relacionados | Radio de un mismo grupo |

> `aria-label` solo funciona si el elemento tiene un rol que admita nombres accesibles (`button`, `a`, `input`, `textarea`, `select`). Ponerlo sobre un `<div>` no hace nada: el `div` no es enfocable ni expone nombre. Para un `div` que hace de botón, la solución correcta es usar `<button>`, no agregarle `role` y `tabindex` a mano.

### Imágenes responsive

| Atributo / elemento | Para qué | Ejemplo |
|----------------------|----------|---------|
| `srcset` | Lista de candidatos con su ancho | `srcset="gato-400.jpg 400w, gato-800.jpg 800w"` |
| `sizes` | Ancho esperado del slot, para elegir el candidato | `sizes="(max-width: 600px) 100vw, 50vw"` |
| `loading="lazy"` | Carga diferida para imágenes fuera de pantalla | `loading="lazy" decoding="async"` |
| `<picture>` | Fuente alternativa según formato o media query | `webp` + fallback `jpg` |

```html
<img
  src="gato-800.jpg"
  srcset="gato-400.jpg 400w, gato-800.jpg 800w, gato-1200.jpg 1200w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Un gato naranja durmiendo en un teclado"
  width="800"
  height="600"
  loading="lazy"
  decoding="async">
```

> `sizes` describe el ancho **del contenedor**, no el de la imagen final. El navegador lo combina con el `srcset` para elegir el archivo. `sizes` sin `srcset` no hace nada, y `srcset` sin `sizes` hace que el navegador asuma `100vw`, que en desktop significa imágenes más grandes de la cuenta.

### Entidades HTML

| Entidad | Carácter | Uso |
|---------|----------|-----|
| `&amp;` | `&` | Obligatorio en URLs con query string: `?a=1&amp;b=2` |
| `&lt;` / `&gt;` | `<` / `>` | Texto que describe etiquetas |
| `&nbsp;` | espacio duro | Evita el salto de línea en un espacio |
| `&copy;` | `©` | Copyright |
| `&mdash;` | `—` | Raya larga |
| `&#x1F600;` | emoji | Entidad numérica hexadecimal |

> Un `&` sin escapar en el texto o en una URL es un error de validación, y el parser puede convertir el resto de la línea en un atributo. En las URLs de query strings, el error clásico es `?a=1&b=2`: el `&b` se interpreta como el inicio de una entidad.

### iframes

| Atributo | Para qué | Ejemplo |
|----------|----------|---------|
| `src` | Documento a incrustar | `src="https://ejemplo.com/mapa"` |
| `title` | Nombre accesible del frame | Requerido para contenido con iframe |
| `sandbox` | Restringe lo que el contenido incrustado puede hacer | `sandbox="allow-scripts allow-same-origin"` |

```html
<iframe
  src="https://www.youtube-nocookie.com/embed/dQw4w9WgXcQ"
  title="Video tutorial de instalación"
  width="560"
  height="315"
  loading="lazy"
  referrerpolicy="strict-origin"
  allowfullscreen></iframe>
```

> `sandbox` sin `allow-same-origin` crea un origen opaco: el contenido incrustado pierde acceso a `localStorage` y a las cookies del sitio anfitrión. Es la opción segura por defecto para contenido de terceros.

### Atributos globales y booleanos

| Atributo | Se aplica a | Nota |
|----------|-------------|------|
| `id` | Cualquier elemento | Único en el documento. Necesario para `label for` y `aria-*` |
| `class` | Cualquier elemento | Uno o más, separados por espacios |
| `tabindex` | Elementos interactivos | `0` = orden natural, `-1` = solo por script, `>0` = orden explícito |
| `inert` | Contenedores | Desactiva interacción y foco de todo el subárbol |

Los atributos booleanos se escriben sin valor: `disabled`, `checked`, `selected`, `readonly`, `required`, `multiple`, `hidden`, `open`, `autofocus`, `novalidate`, `defer`, `async`.

---

## 💻 Ejemplos

### Página HTML5 completa y válida

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Chuletario — Referencia rápida</title>
    <meta name="description" content="Cheatsheets de referencia rápida para estudiantes.">
    <link rel="canonical" href="https://ejemplo.com/chuletario">
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <a class="skip-link" href="#contenido">Saltar al contenido principal</a>

    <header>
      <h1>Chuletario</h1>
      <p>Referencia rápida para el día a día.</p>
    </header>

    <nav aria-label="Navegación principal">
      <ul>
        <li><a href="./bash/README.md" aria-current="page">Bash</a></li>
        <li><a href="./html/README.md">HTML</a></li>
        <li><a href="./css/README.md">CSS</a></li>
      </ul>
    </nav>

    <main id="contenido">
      <article>
        <h2>¿Qué es esto?</h2>
        <p>Cada carpeta tiene un <code>README.md</code> con la sintaxis esencial.</p>

        <section>
          <h3>Última actualización</h3>
          <p>Actualizado el <time datetime="2026-10-21">21 de octubre de 2026</time>.</p>
        </section>
      </article>

      <aside>
        <h2>Nota</h2>
        <p>Las versiones están pineadas. Si tu máquina tiene otra, revisa los gotchas.</p>
      </aside>
    </main>

    <footer>
      <p><small>Contenido educativo bajo licencia MIT.</small></p>
    </footer>
  </body>
</html>
```

Salida esperada al validarlo con `vnu --errors-only index.html`: ninguna salida y código de salida `0`.

```text
$ vnu --errors-only index.html; echo "exit=$?"
exit=0
```

### Formulario accesible completo

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Registro — Chuletario</title>
  </head>
  <body>
    <main>
      <h1>Crear una cuenta</h1>

      <form action="/registro" method="post">
        <fieldset>
          <legend>Datos personales</legend>

          <div>
            <label for="nombre">Nombre completo</label>
            <input type="text" id="nombre" name="nombre" required
                   minlength="2" maxlength="80" autocomplete="name">
          </div>

          <div>
            <label for="email">Correo electrónico</label>
            <input type="email" id="email" name="email" required
                   autocomplete="email" aria-describedby="ayuda-email">
            <p id="ayuda-email">Te enviamos la confirmación a esta dirección.</p>
          </div>
        </fieldset>

        <fieldset>
          <legend>Preferencias</legend>

          <div>
            <input type="checkbox" id="boletin" name="boletin" value="si">
            <label for="boletin">Quiero recibir el boletín semanal</label>
          </div>

          <div>
            <input type="radio" id="plan-basico" name="plan" value="basico" checked>
            <label for="plan-basico">Plan básico</label>
            <input type="radio" id="plan-pro" name="plan" value="pro">
            <label for="plan-pro">Plan pro</label>
          </div>

        </fieldset>

        <button type="submit">Crear cuenta</button>
      </form>
    </main>
  </body>
</html>
```

Salida esperada: el navegador valida en el cliente los campos `required`, `type="email"` y `pattern` antes de enviar, y cada control tiene un nombre accesible tomado de su `<label for>`.

```text
# Con el campo email vacío, el navegador muestra el mensaje nativo:
Please fill out this field.
```

### Layout semántico completo

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Blog — Chuletario</title>
  </head>
  <body>
    <a class="skip-link" href="#principal">Ir al contenido</a>

    <header>
      <nav aria-label="Navegación principal">
        <a href="/" aria-label="Chuletario, ir al inicio">Chuletario</a>
        <ul>
          <li><a href="/blog">Blog</a></li>
          <li><a href="/docs">Documentación</a></li>
        </ul>
      </nav>
    </header>

    <main id="principal">
      <h1>Blog</h1>

      <article>
        <header>
          <h2>Cómo escribir un shell script mantenible</h2>
          <p>Publicado el <time datetime="2026-10-21">21 de octubre de 2026</time>
             por <span>Equipo Chuletario</span></p>
        </header>

        <section>
          <h3>El problema</h3>
          <p>Los scripts de shell se rompen sin que nadie se entere.</p>
        </section>

        <section>
          <h3>La solución</h3>
          <p>Modo estricto, funciones con <code>local</code> y <code>trap</code> de limpieza.</p>
        </section>
      </article>

    </main>

    <aside aria-label="Artículos relacionados">
      <h2>Relacionados</h2>
      <ul>
        <li><a href="/bash">Bash</a></li>
        <li><a href="/css">CSS</a></li>
      </ul>
    </aside>

    <footer>
      <address>Chuletario — <a href="mailto:hola@ejemplo.com">hola@ejemplo.com</a></address>
      <p><small>Licencia MIT</small></p>
    </footer>
  </body>
</html>
```

> Un `<article>` puede tener su propio `<header>` y su propio `<footer>`. Eso es intencional: el `<header>` de un artículo es distinto del `<header>` del documento, y HTML5 lo permite por diseño. El único que no se puede repetir es `<main>`.

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| `<meta charset>` después de 1024 bytes | El navegador ya decodificó los bytes con otra codificación y aparecen caracteres rotos | Poné el `charset` en las primeras líneas del `<head>` |
| Sin `<meta name="viewport">` | En móvil se renderiza a ~980px y el texto queda ilegible | `width=device-width, initial-scale=1` |
| Omitir `lang` en `<html>` | El lector de pantalla usa la pronunciación por defecto y los buscadores no segmentan | `<html lang="es">` |
| `<div>` para todo | Se pierde el árbol semántico: nada de landmarks, ni encabezados, ni navegación por regiones | Usa `header`, `nav`, `main`, `section`, `article`, `aside`, `footer` |
| Varios `<main>` en la misma página | La navegación por landmarks queda ambigua | Un solo `<main>` por documento |
| `alt` con el nombre del archivo | `"gato.jpg"` no describe nada; si la imagen no aporta información, `alt` debe ir vacío | Describí la función de la imagen, o `alt=""` si es decorativa |
| `placeholder` como etiqueta | desaparece al escribir y no es un nombre accesible | `<label for>` siempre; el `placeholder` es solo una ayuda |
| `aria-label` en un `<div>` | No hace nada: el `div` no tiene rol que admita nombre | Usa `<button>` o `<a href>` en vez de un `div` con `role` |
| `<h1>` por sección | Rompe la jerarquía de encabezados y confunde a los lectores de pantalla | Un `<h1>` por documento; el nivel lo define la estructura, no el tamaño visual |
| `<iframe>` sin `title` | El frame no tiene nombre accesible y el usuario no sabe qué hay | Siempre `title`, y `sandbox` para contenido de terceros |
| `&` sin escapar en una URL | `?a=1&b=2` se interpreta como el inicio de una entidad | `?a=1&amp;b=2` |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| Elementos semánticos sobre `<div>` | Landmarks, navegación por encabezados, mejor SEO | Los estilos por defecto de `section`/`article` hay que resetear |
| Validación con Nu HTML Checker en CI | Atrapa errores antes de producción | Falla el build por detalles menores; hay que configurarlo por severidad |
| `loading="lazy"` en todas las imágenes | Ahorra datos y mejora el LCP en páginas largas | Retrasa la carga de imágenes visibles; la imagen principal debe ir sin `lazy` |
| `srcset` + `sizes` sobre una sola imagen |Imágenes adaptadas al dispositivo y al viewport | Más bytes de HTML y más complejidad para mantener |
| `<picture>` sobre `srcset` | Permite cambiar de formato (`webp`/`avif`) | Más verboso; `srcset` con `type` cubre casi todo |
| `sandbox` estricto en iframes | El contenido de terceros no toca tus cookies ni tu DOM | Muchos embebidos dejan de funcionar sin `allow-scripts` |
| `aria-describedby` para errores | Asocia el mensaje de error al campo de forma explícita | Más marcado que un `aria-live` global, y hay que actualizarlo dinámicamente |

---

## 📚 Further reading

- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/multipage/) — la especificación oficial, siempre actualizada
- [MDN: HTML reference](https://developer.mozilla.org/en-US/docs/Web/HTML) — documentación práctica de MDN Web Docs
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) — patrones accesibles de componentes reales
- [Nu HTML Checker](https://validator.w3.org/nu/) — el validador oficial, con explicación de cada error
- [HTML: A Standardized API (web.dev)](https://web.dev/learn/html) — ruta de aprendizaje de web.dev
