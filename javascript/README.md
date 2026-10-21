# Chuletario JavaScript

> Todo lo que necesitas para trabajar con JavaScript en el día a día, en una sola página.

**TL;DR:** JavaScript tiene un solo tipo numérico (`number`, más `BigInt` para enteros grandes), un único modelo de objetos basado en prototipos y dos valores vacíos (`null` y `undefined`). `const` y `let` reemplazaron a `var` hace más de una década, `===` es la única comparación segura, y `async`/`await` sobre `Promise` es el modelo estándar de asincronía. La edición vigente es **ECMAScript 2026 (17.ª edición, junio de 2026)**, verificada el 21 de octubre de 2026.

**Audiencia:** Este chulet asume que ya sabes qué es una variable, un bucle, una función y una clase en cualquier lenguaje. Si te falta base, empieza por el [chulet de HTML](./../html/README.md) y el de [CSS](./../css/README.md). Para ejecutar JavaScript en el servidor, el [chulet de Node.js](./../nodejs/README.md) es el complemento natural.

---

## 📋 Requisitos

| Herramienta / edición | Versión | Nota |
|-------------|----------|------|
| ECMAScript (ECMA-262) | **2026 — 17.ª edición (junio de 2026)** | La edición publicada vigente. Es la referencia de este chulet |
| Node.js | `v24.22.1` (LTS "Krypton") | Para ejecutar JavaScript fuera del navegador. `v26.12.1` es "Current", no producción |
| Navegador | Baseline "widely available" (2026) | Características soportadas por los navegadores principales durante al menos 30 meses |
| Editores con soporte de ES2025+ | VS Code 1.100+ / WebStorm 2026.1+ | Autocompletado y tipos de las APIs nuevas |

```bash
node --version   # v24.22.1
```

---

## 🚀 Setup / Cómo ejecutar

**En el navegador.** Abre las DevTools (`F12`), ve a la pestaña **Console** y escribe `console.log('Hola')`.

**En Node.js (archivo).**

```bash
mkdir js-playground && cd js-playground
printf 'const n = 21;\nconsole.log(n * 2);\n' > app.js
node app.js     # 42
```

**REPL interactivo.** `node` y luego:

```console
> const [a, b] = [1, 2]
undefined
> a + b
3
```

**Modo watch.** `node --watch app.js` reinicia el proceso cuando cambia un archivo importado: es más simple que `nodemon` y viene incluido en Node. Admite `--env-file=.env`.

---

## 📖 Cheatsheet

### Variables y scope

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Función | Bloque (`{ }`) | Bloque (`{ }`) |
| Hoisting | Sí, a `undefined` | Sí, a `undefined` | Sí, a `undefined` |
| Reasignable | Sí | Sí | No (lanza `TypeError`) |
| Redeclarable | Sí | No | No |

Regla práctica: `const` por defecto, `let` cuando el valor cambia, `var` solo en código legacy.

**Hoisting y TDZ.** Las declaraciones se elevan al principio del ámbito, pero no su valor: `let` y `const` quedan en la *zona temporal muerta* y usarlos antes de la declaración lanza `ReferenceError`. `typeof` sobre un identificador **no declarado** devuelve `'undefined'` sin error, pero sobre una variable en TDZ **sí** lanza.

**`const` no congela el objeto.** Impide reasignar la *variable*, no mutar el valor: `config.retries = 5` es legal, `config = {}` lanza `TypeError`. `Object.freeze` congela, y solo en el nivel superior. Un `const` dentro de una función oculta al exterior (*shadowing*); sin declaración, una asignación crea un binding global: nunca lo hagas, en el navegador contamina `window`.

### Tipos de datos

| Tipo | `typeof` | Ejemplo | Notas |
|------|----------|---------|-------|
| `undefined` | `"undefined"` | `let x;` | Ausencia de valor |
| `null` | `"object"` | `let x = null;` | Ausencia *intencional*. El `typeof` es un bug histórico |
| `boolean` | `"boolean"` | `true` | |
| `number` | `"number"` | `42`, `3.14`, `NaN` | IEEE-754 doble (64 bits) |
| `bigint` | `"bigint"` | `42n` | Enteros de tamaño arbitrario. No se mezcla con `number` |
| `string` | `"string"` | `"hi"`, `` `hi` `` | Inmutables |
| `symbol` | `"symbol"` | `Symbol("id")` | Identidad única, no serializable |
| `object` | `"object"` | `{}`, `[]`, `null`, `new Date()` | Todo lo demás, incluidas las funciones |

**El bug de `typeof null`.** Devuelve `'object'` porque en 1995 `null` se implementó como un placeholder de un solo valor y `typeof` solo miraba el bit de tipo. No se puede corregir sin romper la web, así que el lenguaje lo congela: usa `x === null` o `Object.prototype.toString.call(x)`. `null == undefined` es `true` (por `==`), `null === undefined` es `false`.

**`NaN` y precisión.** `0.1 + 0.2` da `0.30000000000000004` porque `0.1` no es representable en binario; el resultado es el número representable más cercano, que no coincide bit a bit con `0.3`. `NaN === NaN` es `false` (usa `Number.isNaN` u `Object.is`). Para dinero usa enteros de centavos o `BigInt`.

**`BigInt` y `Symbol`.** A partir de `2^53`, `number` deja de representar enteros consecutivos de forma exacta, así que `9007199254740993` se redondea a `9007199254740992`: usa `BigInt` para IDs y contadores grandes. `BigInt` y `number` no se pueden mezclar (`10n + 1` lanza `TypeError`). Cada `Symbol` es único aunque dos tengan la misma descripción, y `JSON.stringify` los omite.

**Boxing.** Los primitivos se envuelven automáticamente en objetos para llamar métodos, y el envoltorio es temporal: `(42).toFixed(2)` funciona, pero `new Number(42) === 42` es `false`. Como `typeof []` es `'object'`, usa `Array.isArray(x)` para arrays y `Object.prototype.toString.call(x)` para el resto.

### Operadores

| Operador | Significado | Nota |
|----------|-------------|------|
| `+` | Suma o concatenación | Convierte a string si algún operando es string |
| `-` `*` `/` `%` | Resta, multiplicación, división, resto | Convierte a `number` |
| `**` `++` `--` | Exponente, incremento, decremento | `2 ** 3 === 8`; el postfijo devuelve el valor **antes** de operar |
| `==` | Coerción + comparación | Trampa histórica. No lo uses |
| `===` `!==` | Igualdad / desigualdad estricta | Sin coerción. `Object.is(a, b)` es la variante `SameValue`: distingue `+0` de `-0` y trata `NaN` como igual a sí mismo |
| `&&` `\|\|` `??` | Cortocircuito y nullish | Devuelven un operando; `??` solo mira `null`/`undefined` |
| `!a` / `a ? b : c` | Negación / ternario | |
| `a?.b` | Optional chaining | Cortocircuita si `a` es `null`/`undefined`. También funciona con métodos: `obj.method?.()` |
| `\|\|=` `&&=` `??=` | Asignación lógica | Equivale a `a op (a = b)` |
| `[...arr]` `{ ...obj }` `f(...args)` | Spread / rest | Copia enumerable, sin crear el original |
| `&` `\|` `^` `~` `<<` `>>` `>>>` | Bitwise | `5 & 3` → `1`, `5 << 1` → `10` |
| `delete obj.prop` | Elimina una propiedad own | `delete user.age` |
| `prop in obj` / `obj instanceof Ctor` | ¿Existe la propiedad? / ¿Prototipo en la cadena? | `'toString' in {}` → `true`; `[] instanceof Array` → `true` |
| `void expr` | Devuelve `undefined` | `void 0` |

> `==` no es "flojo": es una máquina de coerción con reglas implícitas. Si ambos lados son del mismo tipo, compara directo; si no, convierte. El resultado de `a == b` depende de los **tipos**, no de los valores: por eso la comunidad lo evita. `===` compara tipo y valor, sin excepciones. `in` busca también en el prototipo, así que para pedir solo propiedades own usa `Object.hasOwn(obj, key)`.

```js
const port = 0;
console.log(port || 3000, port ?? 3000);   // 3000 0
const user = { profile: { address: { city: "Valencia" } } };
console.log(user?.profile?.address?.city);  // 'Valencia'
```

> `?.` solo protege el eslabón anterior: `user?.contact.email` lanza `TypeError` si `user.contact` no existe.

### Conversión y truthiness

| Tipo | Valor | ¿Truthy? |
|------|-------|----------|
| `boolean` | `true` | Sí |
| `number` | `0`, `NaN`, `-0` / cualquier otro | No / Sí |
| `string` | `""` / cualquier otra | No / Sí |
| `undefined` / `null` | — | No |
| `object` | `{}`, `[]`, `function` | Sí (incluso los vacíos) |

```js
console.log(Number("42"), Number("42px"), Number(""));    // 42 NaN 0
console.log(parseInt("42px", 10), parseFloat("3.14abc")); // 42 3.14
console.log("5" * 2, "5" + 2, +true, 1 + null);            // 10 '52' 1 1
```

> `parseInt` y `parseFloat` leen el prefijo numérico y se detienen en el primer carácter inválido; `Number()` exige que toda la cadena sea numérica. `parseInt` acepta prefijos como `0x` salvo que le pases la base 10 explícitamente. Con `+`, si un operando es string gana la concatenación: por eso `"5" * 2` da `10` pero `"5" + 2` da `'52'`.

### Strings

| Método | Qué hace | Nota |
|--------|----------|------|
| `s.at(i)` | Carácter en `i` | Negativo cuenta desde el final |
| `s.includes(x)` / `s.startsWith(x)` / `s.endsWith(x)` | ¿Contiene? / prefijo / sufijo | `includes` no acepta regex |
| `s.padStart(n, c)` / `s.padEnd(n, c)` | Rellena hasta `n` | Alineación de columnas |
| `s.slice(a, b)` | Subcadena | Final exclusivo; negativos desde el final |
| `s.substring(a, b)` | Subcadena | Intercambia los extremos si `a > b`; no acepta negativos |
| `s.substr(a, len)` | Subcadena | Obsoleto, sigue disponible |
| `s.split(sep)` | Divide | Con `''` parte en caracteres |
| `s.replace(a, b)` / `s.replaceAll(a, b)` | Reemplaza | `replace` solo la primera, salvo regex global |
| `s.trim()` | Quita espacios | `trimStart` / `trimEnd` también |
| `s.match(re)` / `s.matchAll(re)` | Coincidencias | `match` devuelve `null` si no hay; `matchAll` exige regex global |

**Template literals.** Interpolan, admiten saltos de línea reales y evalúan expresiones dentro del placeholder. `\${name}` escapa el placeholder. Una *tagged template* es una función que recibe primero las cadenas y luego los valores interpolados: es la base de las librerías de HTML y de sanear.

**Emoji.** `length` cuenta unidades UTF-16, no caracteres visibles: un emoji con ZWJ mide más de 2. Recorre con `for...of` o `Array.from()` para no cortar un par suplente por la mitad, y usa `codePointAt(0)` para el punto de código real.

### Arrays

| Método | Devuelve | ¿Muta? |
|--------|----------|--------|
| `map(fn)` / `filter(fn)` | Array transformado / filtrado | No |
| `reduce(fn, init)` | Un valor acumulado | No |
| `find(fn)` / `findLast(fn)` / `findIndex(fn)` | Primer / último elemento, o el índice | No |
| `some(fn)` / `every(fn)` / `includes(x)` | Booleano | No |
| `flat(n)` / `flatMap(fn)` / `at(i)` / `entries()` | Array aplanado / elemento / iterador | No |
| `sort(cmp)` / `reverse()` | El mismo array | **Sí** |
| `splice(i, n)` / `push` / `pop` | Elementos eliminados / muta | **Sí** |

```js
const prices = [10, 25, 5];
console.log(prices.map((p) => p * 1.21));             // [ 12.1, 30.25, 6.05 ]
console.log(prices.filter((p) => p > 8));             // [ 10, 25 ]
console.log(prices.reduce((sum, p) => sum + p, 0));    // 40
```

**Ordenar.** `sort()` sin comparador convierte cada elemento a string y compara por unidades UTF-16: `"100" < "9"` es `true`, así que `[10, 9, 100, 1].sort()` da `[1, 10, 100, 9]`. Usa `(a, b) => a - b` para números y `a.localeCompare(b)` para texto con acentos. `sort` muta el array: `toSorted(cmp)` (ES2023) devuelve una copia.

**`Array.from` vs `Array.of` vs spread.** `Array.from(iterable, fn)` construye desde cualquier iterable; `Array.of(7)` siempre devuelve un array. En cambio `Array(3)` crea un array *sparse* con tres huecos, no tres ceros: la mayoría de métodos lo saltan, así que `[1, , 3].map((x) => x * 2)` da `[2, <1 empty item>, 6]`. Usa `Array.from` o `Array(n).fill(0)`.

**Destructuring en `for...of`:** `for (const { name, age } of users) console.log(name, age);` y `for (const [x, y] of points) console.log(x, y);`.

### Objetos

| Método | Qué hace |
|--------|----------|
| `Object.keys(obj)` / `Object.values(obj)` / `Object.entries(obj)` | Arrays de claves own / valores own / pares `[clave, valor]` |
| `Object.fromEntries(pairs)` | Construye un objeto desde pares (inversa de `entries`) |
| `Object.assign(target, ...src)` | Copia propiedades own de los sources |
| `Object.freeze(obj)` / `Object.seal(obj)` | Congela / impide añadir o eliminar (no afecta a objetos anidados) |
| `Object.hasOwn(obj, key)` / `Object.is(a, b)` | ¿Propiedad own? / igualdad `SameValue` |
| `Object.groupBy(items, fn)` / `Map.groupBy(items, fn)` | Agrupa en un objeto o en un `Map` (ES2024) |

```js
const orders = [{ id: 1, status: "paid", total: 10 }, { id: 3, status: "paid", total: 20 }];
const byStatus = Object.groupBy(orders, (o) => o.status);
console.log(byStatus.paid.map((o) => o.total)); // [ 10, 20 ]
```

**Sintaxis del literal y destructuring.** Shorthand (`{ name }` equivale a `{ name: name }`), claves computadas (`{ [key]: value }`), métodos (`greet() {}`) y campos (`name = "Ada"`). El destructuring admite anidado, defaults, renombrado y rest, y también funciona en los parámetros de una función (por eso `function f({ a = 1 } = {})` no falla si no le pasas nada).

```js
const { data: { user: { name, email = "sin email" } } } = response;
const { meta: { page: currentPage, ...restMeta } } = response;
function draw({ x = 0, y = 0, color = "black" } = {}) { return `(${x}, ${y}) ${color}`; }
```

**Spread de objetos y orden de claves.** `{ ...defaults, ...prefs }` copia propiedades own: el último objeto en gana para las claves repetidas. En el resultado, las claves que son índices enteros van primero en orden numérico ascendente y el resto en orden de inserción: `JSON.stringify({ b: 1, 2: 2, a: 3, 1: 4 })` produce `{"1":4,"2":2,"b":1,"a":3}`.

### Funciones

| | Declaración | Expresión | Arrow |
|---|---|---|---|
| Sintaxis | `function f() {}` | `const f = function () {}` | `const f = () => {}` |
| Hoisting | Sí, completa | No | No |
| `this` | Dinámico (según la llamada) | Dinámico | Léxico (del scope donde se define) |
| `arguments` | Sí | Sí | **No** |
| Callable con `new` | Sí | Sí | **No** |
| Retorno implícito | No | No | Sí, con expresión única |

**Cómo se bindea `this`:** llamada simple en modo estricto → `undefined`; en modo flojo → `globalThis`; como método → el objeto receptor; con `call`/`apply` → el primer argumento; con `new` → la instancia creada; arrow → el `this` léxico del scope exterior. El [Ejemplo 2](#ejemplo-2--el-mismo-método-en-seis-contextos-de-binding) muestra todos los casos con el mismo código.

> Una arrow no tiene `this` propio: hereda el de donde se *definió*, no el de donde se *llamó*. Por eso no sirven como métodos de objeto ni como constructoras: no se pueden invocar con `new` y no tienen `arguments`.

**Closures.** Una función captura el ámbito donde se creó, incluso si ese ámbito ya terminó: el contador de un factory vive en el closure, no en la función que lo crea.

**Parámetros y orden superior.** Defaults (`name = "mundo"`), rest (`...rest`) e IIFE (`(function () { ... })()`). Una función que recibe o devuelve funciones es de orden superior, y una que se pasa como argumento es un *callback*; el currying convierte una función de varios argumentos en una cadena de funciones de un argumento.

**`call`, `apply` y `bind`.** `call` recibe argumentos sueltos, `apply` un array, y `bind` prefija argumentos y devuelve una función nueva sin ejecutarla. Con `person = { name: "Ada" }` e `introduce(greeting, punctuation)`: `introduce.call(person, "Hello", "!")` da `Hello, I am Ada!` y `introduce.bind(person, "Hello")(".")` da `Hello, I am Ada.`.

### Clases

| Elemento | Visibilidad | Se hereda |
|----------|-------------|-----------|
| `campo` / `get` / `set` | Pública | Sí |
| `#campo` | Privada (solo dentro de la clase) | No |
| `static` / `static {}` | Pertenece a la clase, no a las instancias / bloque de inicialización que se ejecuta una vez al cargar | Sí / no |

```js
class Point {
  #x;                    // campo privado
  static instances = 0;   // campo estático
  constructor(x, y = 0) { this.#x = x; this.y = y; Point.instances += 1; }
  get x() { return this.#x; }              // getter
  set x(value) { this.#x = value; }        // setter
  #privateMethod() { return this.#x * 2; } // método privado
  static origin() { return new Point(0, 0); }  // método estático
  static { console.log("clase cargada"); }    // bloque estático
}
```

**Herencia.** `class Dog extends Animal` y `super(...)` en el constructor: debe llamarse antes de usar `this`, porque hasta entonces `this` no existe en la subclase. `instanceof` comprueba la cadena de prototipos.

### Asincronía

| API | Resuelve | Rechaza |
|-----|----------|---------|
| `await promise` | Con el valor | Con el error |
| `Promise.all(promises)` | Cuando **todas** resuelven | Con el primer error |
| `Promise.allSettled(promises)` | Siempre, con el estado de cada una | Nunca |
| `Promise.race(promises)` | La primera que resuelva | La primera que rechace |
| `Promise.any(promises)` | La primera que resuelva | `AggregateError` si todas fallan |

```js
async function loadUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

> Un `async` siempre devuelve una `Promise`: si la función lanza, la promesa se rechaza, y `try/catch` alrededor de un `await` captura el error.

**Event loop: microtareas vs macrotareas.** El código síncrono se ejecuta entero antes de que el loop procese la cola. Después se vacían **todas** las microtareas (promesas resueltas, `queueMicrotask`) antes de la siguiente macrotarea (`setTimeout`, eventos de I/O). `setTimeout(fn, 0)` no significa "inmediato", significa "en la próxima vuelta del loop".

```js
console.log("A: sync start");
setTimeout(() => console.log("E: macrotask (setTimeout 0)"), 0);
Promise.resolve().then(() => console.log("C: microtask (Promise.then)"));
queueMicrotask(() => console.log("D: microtask (queueMicrotask)"));
console.log("B: sync end");
```

Salida: `A: sync start`, `B: sync end`, `C: microtask (Promise.then)`, `D: microtask (queueMicrotask)`, `E: macrotask (setTimeout 0)`.

**Cancelar con `AbortController`.** `controller.abort()` dispara un `AbortError` en el `fetch` que usa su `signal`.

### Iteradores y generadores

```js
function* range(start, end) {
  for (let i = start; i < end; i++) yield i;
}
for (const n of range(0, 3)) console.log(n);  // 0, 1, 2
console.log([...range(0, 3)]);                // [ 0, 1, 2 ]
```

| | `for...of` | `for...in` |
|---|---|---|
| Recorre | Valores de un iterable | Claves (índices) de un objeto |
| Orden y strings | El del iterador; en strings, por puntos de código | Orden de inserción de las claves; en strings, por índices |
| Objetos planos | Falla (no son iterables) | Funciona: recorre las claves |

**Protocolo iterador.** Un objeto es iterable si implementa `[Symbol.iterator]()` devolviendo un objeto con `next()` que devuelve `{ value, done }`. `Symbol.asyncIterator` y `for await...of` hacen lo mismo con promesas, y `yield*` delega en otro iterador.

### Módulos ESM

| Concepto | Sintaxis | Nota |
|----------|----------|------|
| Exportar con nombre / por defecto | `export const a = 1` / `export default expr` | `import { a } from "./m.js"` / `import expr from "./m.js"` |
| Reexportar | `export { a } from "./m.js"` | `export * from "./m.js"` reexporta todo |
| Import dinámico | `await import("./m.js")` | Devuelve una promesa; carga bajo demanda |
| Metadatos del módulo | `import.meta.url` | Ruta absoluta del archivo |
| Await de nivel superior / activar ESM | `await` fuera de una función async / `"type": "module"` en `package.json` | Solo en ESM. También vale la extensión `.mjs` |

```js
// utils.js
export const TAX = 0.21;
export default function format(amount) { return `$${amount.toFixed(2)}`; }
```

```js
// main.js
import format, { TAX } from "./utils.js";
console.log(format(100 * (1 + TAX)));  // $121.00
```

### Errores

| Elemento | Para qué |
|----------|----------|
| `try` / `catch (err)` / `finally` | Bloque que puede lanzar / captura el error (sin binding: `catch {}`) / se ejecuta siempre |
| `err.cause` | Error original que causó este error |
| `err.name` / `err instanceof Error` | `'TypeError'`, `'RangeError'`, ... / verificación de tipo real |
| `AggregateError` | Varios errores a la vez (`.errors`) |

```js
class ValidationError extends Error {
  constructor(field, message) { super(message); this.name = "ValidationError"; this.field = field; }
}
try { throw new ValidationError("email", "formato inválido"); }
catch (err) { throw new Error("no se pudo cargar el usuario", { cause: err }); }
```

> `instanceof` falla si el error viene de otro realm (un iframe) o si la cadena de prototipos se rompió. Para errores que cruzan fronteras, compara `err.name` o usa `Error.isError(err)` (ES2026).

### JSON y datos

| Comportamiento | Detalle |
|----------------|---------|
| `undefined` | En un objeto se **omite** la propiedad; en un array se convierte en `null` |
| `NaN` / `Infinity` | Se convierten en `null` |
| `Date` / `BigInt` | `Date` se convierte a string ISO 8601; `BigInt` **lanza `TypeError`**, no es serializable |
| Funciones, `Symbol`, `Map`, `Set` | Se omiten o se serializan como `{}` |

```js
console.log(JSON.parse('{"a":1}', (key, v) => (key === "a" ? v * 2 : v))); // { a: 2 }
console.log(JSON.stringify({ a: 1, b: undefined, c: NaN }, null, 2));  // { "a": 1, "c": null }
```

> `JSON.stringify` solo sabe serializar estructuras JSON: para `BigInt` conviértelo a string antes, y para `Map`/`Set` usa `Object.fromEntries(map)` o un replacer.

### Navegador: DOM y eventos

| API | Qué hace |
|-----|----------|
| `document.querySelector(sel)` / `querySelectorAll(sel)` | Primer elemento / `NodeList` de todos los que coinciden |
| `el.createElement("div")` / `parent.append(child)` / `el.remove()` | Crea, inserta o elimina un elemento del DOM |
| `el.classList.add/remove/toggle` / `el.dataset.id` | Clases CSS / atributos `data-*` |
| `el.addEventListener(type, fn)` / `e.preventDefault()` | Escucha un evento / cancela la acción por defecto |
| `localStorage.getItem(k)` / `history.pushState(state, "", url)` | Almacenamiento persistente por origen / cambia la URL sin recargar |
| `requestAnimationFrame(fn)` | Callback antes del siguiente repintado |

**Delegación de eventos.** En vez de un listener por elemento, pon uno solo en el contenedor y usa `event.target.closest(...)`: los eventos burbujean, así que el click del botón llega primero al botón, luego al `li` y después al contenedor. Con muchos nodos es la diferencia entre un listener y cientos, y evita fugas por listeners que nunca se limpian.

```js
const list = document.querySelector("#tasks");
list.addEventListener("click", (event) => {
  const item = event.target.closest("li[data-id]");
  if (!item) return;
  console.log("completed task", item.dataset.id);
  item.classList.toggle("done");
});
```

| Función | Se dispara | Uso típico |
|---------|-------------|-------------|
| `setTimeout(fn, ms)` | Una vez, tras `ms` | Retardo, debounce |
| `setInterval(fn, ms)` | Cada `ms` | Reloj, polling |
| `requestAnimationFrame(fn)` | Antes de cada repintado | Animaciones |
| `queueMicrotask(fn)` | Al final de la cola de microtareas | Trabajo inmediato |

Para el HTML y el CSS, ve al [chulet de HTML](./../html/README.md) y al de [CSS](./../css/README.md).

### Novedades por edición

| Feature | Edición | Ejemplo |
|---------|---------|---------|
| Iterator Helpers (sync) | ES2025 | `Iterator.from([1,2]).map(x => x * 2).toArray()` |
| `RegExp.escape` | ES2025 | `RegExp.escape("a+b")` → `"a\\+b"` |
| `Promise.try` | ES2025 | `Promise.try(() => risky())` |
| `Float16Array` / `Math.f16round` | ES2025 | `Math.f16round(1.1)` |
| Métodos de `Set` | ES2025 | `a.union(b)`, `a.difference(b)`, `a.isSubsetOf(b)` |
| Import Attributes / JSON Modules | ES2025 | `import data from "./d.json" with { type: "json" }` |
| Grupos de captura con nombre duplicados | ES2025 | `/(?<n>a)\|(?<n>b)/` |
| `Array.fromAsync` | ES2026 | `await Array.fromAsync(stream)` |
| `Math.sumPrecise` | ES2026 | `Math.sumPrecise([0.1, 0.2])` → `0.3` exacto |
| `Error.isError` | ES2026 | `Error.isError(x)` (funciona entre realms) |
| `Uint8Array.fromBase64` / `toBase64` | ES2026 | `Uint8Array.fromBase64("SGk=")` |
| `Map.prototype.upsert` | ES2026 | `map.upsert(k, v)` |
| Acceso al texto fuente en `JSON.parse` | ES2026 | Reporta la posición exacta del error |
| Iterator Sequencing | ES2026 | Secuencia de iteradores |
| Temporal API | **Próxima edición (ES2027)** | `Temporal.Now.instant()` |
| Explicit Resource Management (`using` / `await using`) | **Próxima edición (ES2027)** | `using res = open()` |
| Iterator chunking / Joint Iteration | **Próxima edición (ES2027)** | `iter.chunks(10)`; iterar varios iterables en paralelo |
| `Atomics.pause` | **Próxima edición (ES2027)** | `Atomics.pause(10)` |

---

## 💻 Ejemplos

### Ejemplo 1 — Closure: un contador y el error clásico de `var`

```js
function createCounter() {
  let count = 0;
  return { increment: () => ++count, read: () => count };
}
const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.read()); // 2

// var is function-scoped, so every closure shares one binding.
function withVar() {
  const fns = [];
  for (var i = 0; i < 3; i++) fns.push(() => i);
  return fns.map((f) => f());
}

// let creates a new binding per iteration, so each arrow captures its own value.
function withLet() {
  const fns = [];
  for (let i = 0; i < 3; i++) fns.push(() => i);
  return fns.map((f) => f());
}
console.log(withVar()); // [ 3, 3, 3 ]
console.log(withLet()); // [ 0, 1, 2 ]
```

Salida:

```console
2
[ 3, 3, 3 ]
[ 0, 1, 2 ]
```

> `var` tiene ámbito de función, no de bloque: las tres flechas comparten la misma variable `i`. Cuando se ejecutan, el bucle ya terminó y `i` vale 3. `let` crea un binding por iteración, así que cada flecha captura el valor de su vuelta.

### Ejemplo 2 — El mismo método en seis contextos de binding

```js
"use strict";

function describe() {
  return this === undefined ? "undefined this" : `this.name = ${this.name}`;
}
const obj = { name: "obj", describe };

console.log(describe());                          // undefined this
console.log(obj.describe());                      // this.name = obj
console.log(describe.call({ name: "explicit" })); // this.name = explicit
console.log(describe.apply({ name: "applied" })); // this.name = applied
console.log(describe.bind({ name: "bound" })());  // this.name = bound
function Point(x) { this.name = "instance"; this.x = x; }
console.log(new Point(3).name); // instance
```

Salida:

```console
undefined this
this.name = obj
this.name = explicit
this.name = applied
this.name = bound
instance
```

### Ejemplo 3 — Destructuring y spread en el procesamiento de un pedido

```js
function processOrder({ customer, items = [], shipping = "standard" } = {}) {
  const { name, address } = customer;
  const total = items.reduce((sum, { price, qty = 1 }) => sum + price * qty, 0);
  const lines = items.map(({ name: itemName, qty = 1 }) => `${qty} x ${itemName}`);
  return { name, address, shipping, total, lines };
}
const order = {
  customer: { name: "Ada", address: "Calle Mayor 1" },
  items: [{ name: "Teclado", price: 80 }, { name: "Raton", price: 25, qty: 2 }],
};
const { name, total, lines } = processOrder(order);
console.log(name, total, lines);
console.log({ ...order, shipping: "express" }.shipping);
```

Salida:

```console
Ada 130 [ '1 x Teclado', '2 x Raton' ]
express
```

### Ejemplo 4 — `Promise.allSettled` con retardos simulados

```js
function delay(ms, value, shouldFail = false) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (shouldFail) reject(new Error(value));
      else resolve(value);
    }, ms);
  });
}
const results = await Promise.allSettled([
  delay(30, "slow ok"),
  delay(10, "fast ok"),
  delay(20, "boom", true),
]);
for (const r of results) {
  if (r.status === "fulfilled") console.log("OK   ", r.value);
  else console.log("FAIL ", r.reason.message);
}
```

Salida:

```console
OK    slow ok
OK    fast ok
FAIL  boom
```

> `allSettled` mantiene el **orden de entrada**, no el de finalización: el primero tardó 30 ms pero sigue siendo el primero de la lista. Además nunca rechaza, así que un fallo en una petición no cancela las demás.

### Ejemplo 5 — Clase con campos privados, getter y factory estática

```js
class Temperature {
  #celsius;
  static #scale = "C";
  constructor(celsius) { this.#celsius = celsius; }
  static fromFahrenheit(f) { return new Temperature((f - 32) * 5 / 9); }
  get fahrenheit() { return this.#celsius * 9 / 5 + 32; }
  set fahrenheit(value) { this.#celsius = (value - 32) * 5 / 9; }
  toString() { return `${this.#celsius} degrees ${Temperature.#scale}`; }
}
const t = Temperature.fromFahrenheit(212);
console.log(t.toString()); // 100 degrees C
console.log(t.fahrenheit); // 212
t.fahrenheit = 32;
console.log(t.toString()); // 0 degrees C
```

Salida:

```console
100 degrees C
212
0 degrees C
```

> Acceder a `t.#celsius` desde fuera de la clase lanza `SyntaxError: Private field '#celsius' must be declared in an enclosing class`: los campos privados no son una convención, son una restricción del motor.

### Ejemplo 6 — Navegador: consulta al DOM y delegación de eventos

```html
<ul id="tasks">
  <li data-id="1">Comprar pan <button class="done">Hecho</button></li>
  <li data-id="2">Llamar al dentista <button class="done">Hecho</button></li>
</ul>
```

```js
const list = document.querySelector("#tasks");

// One listener for the whole list, no matter how many items exist.
list.addEventListener("click", (event) => {
  const button = event.target.closest("button.done");
  if (!button) return;
  console.log("completed task", button.closest("li").dataset.id);
  button.closest("li").classList.add("completed");
});

const extra = document.createElement("li");
extra.dataset.id = "3";
extra.textContent = "Regar las plantas";
list.append(extra);
```

Salida esperada en la consola de DevTools (al pulsar el primer botón "Hecho"):

```console
completed task 1
```

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| `typeof null === 'object'` | Bug de 1995 congelado por compatibilidad | Usa `x === null` o `Object.prototype.toString.call(x)` |
| `0.1 + 0.2 !== 0.3` | `number` es IEEE-754 de 64 bits, no decimal exacto | Compara con tolerancia, o usa enteros/BigInt para dinero |
| `var` dentro de un bucle | Todas las closures comparten la misma variable | Usa `let` (o `const` si no reasignas) |
| Usar una variable antes de su `let` | `ReferenceError` por la zona temporal muerta (TDZ) | Declara antes de usar; `typeof` tampoco salva aquí |
| `==` con tipos distintos | Coerción implícita: `0 == '0'` es `true` | Usa siempre `===` (y `!==`) |
| `0 \|\| 3000` | `0` es falsy, así que devuelve el default | Usa `??` para defaults, `\|\|` solo para flags |
| `arr.sort()` con números | Ordena por string: `[1, 10, 100, 9]` | `arr.sort((a, b) => a - b)` |
| `arr.sort()` muta el array | El original queda reordenado | Usa `arr.toSorted(cmp)` (ES2023) para una copia |
| Arrow function como método | `this` es léxico, no el del objeto | Usa la sintaxis de método: `method() {}` |
| `for...in` sobre un array | Recorre índices como strings y puede incluir heredadas | Usa `for...of` |
| `event.target` en un listener | Apunta al nodo más profundo, que puede ser un `<span>` dentro del botón | Usa `event.target.closest("selector")` |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|---------|
| `const` por defecto sobre `let`/`var` | Evita reasignaciones accidentales y errores de scope | Requiere `let` explícito donde el valor cambia |
| `===` sobre `==` | Sin coerción implícita, resultados predecibles | Descarta comparaciones entre tipos distintos que quizá querías |
| `??` sobre `\|\|` para defaults | Respeta `0`, `""` y `false` como valores válidos | No sirve para un "si está vacío, usa X" genérico |
| Arrow functions sobre `function` | `this` léxico, sintaxis corta, retorno implícito | No son constructoras, no tienen `arguments`, no sirven como métodos |
| Módulos ESM sobre scripts clásicos | Ámbito propio por archivo, import/export explícito, tree-shaking | Requieren servidor HTTP o `type: "module"` (no funciona con `file://`) |
| `Promise.allSettled` sobre `Promise.all` | Ninguna petición falla el lote y obtienes el estado de cada una | Pierdes el fail-fast: hay que inspeccionar cada resultado |
| `Object.freeze` sobre mutar | Detecta errores de mutación temprano | Solo congela el nivel superior, no los objetos anidados |
| `structuredClone` sobre `JSON.parse(JSON.stringify(...))` | Preserva `Date`, `Map`, `Set`, `BigInt` y ciclos | No clona funciones, nodos del DOM ni clases propias |

---

## 📚 Further reading

- [MDN — JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) — la referencia más completa y práctica del lenguaje
- [MDN — Referencia de JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference) — la API estándar, con notas de compatibilidad por navegador
- [ECMA-262 (ECMAScript 2026)](https://262.ecma-international.org/17.0/) — la especificación oficial de la edición vigente
- [TC39 — Finished Proposals](https://github.com/tc39/proposals/blob/main/finished-proposals.md) — propuestas finalizadas y su año de publicación esperado
- [javascript.info](https://javascript.info/) — el tutorial más claro para entender el lenguaje por dentro
