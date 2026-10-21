# Chuletario Node.js

> Todo lo que necesitas para trabajar con Node.js en el día a día, en una sola página.

**TL;DR:** Node.js corre JavaScript fuera del navegador. Usa `npm` para dependencias, ESM (`import`) para módulos nuevos, `process.env` para configuración, y `node --watch` para no reiniciar el server a mano. La LTS actual es **v24.22.1** ("Krypton").

**Audiencia:** Este chulet asume que ya sabes JavaScript (funciones, `async/await`, `Promise`). Si no, empieza por el [chulet de JavaScript](./../javascript/README.md), que cubre el lenguaje completo. El runtime y las herramientas de Node están aquí; el lenguaje (variables, tipos, funciones, `this`) está en el chulet de JavaScript.

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| Node.js | `v24.22.1` (LTS "Krypton") | La LTS. `v26.12.1` es "Current", no la uses en producción |
| npm | `11.19.1` | Viene con Node. No hace falta instalar nada aparte |
| nvm | `0.40.x` | Para cambiar de versión de Node por proyecto |

Verificá tu instalación:

```bash
node --version    # v24.22.1
npm --version     # 11.19.1
```

---

## 🚀 Setup

### Instalar Node con nvm (recomendado)

```bash
# Instalar nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# Reiniciar la terminal, después:
nvm install 24.22.1
nvm use 24.22.1
nvm alias default 24.22.1
```

### Crear un proyecto

```bash
mkdir mi-api && cd mi-api
npm init -y
npm pkg set name="mi-api" type="module" scripts.start="node src/index.js"
npm install express@5.1.0
npm install --save-dev nodemon@3.1.10
```

Salida de `package.json`:

```json
{
  "name": "mi-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/index.js",
    "dev": "node --watch src/index.js"
  },
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

### Otros comandos de `npm`

```bash
npm ci                 # Instala exacto desde package-lock.json (CI/producción)
npm install            # Instala y actualiza el lock
npm install -D pkg     # Dependencia de desarrollo
npm install -g pkg     # Global
npm outdated           # Qué está desactualizado
npm audit fix          # Arregla vulnerabilidades sin romper major versions
npm run dev            # Corre un script
npm exec -- pkg        # Ejecuta binario de dependencia
npx create-vite@latest mi-app   # Crea un proyecto scaffold
```

---

## 📖 Cheatsheet

### `package.json` — campos que importan

| Campo | Para qué | Ejemplo |
|-------|----------|---------|
| `type` | Define el sistema de módulos. `"module"` = ESM | `"module"` |
| `main` | Punto de entrada CommonJS | `"index.js"` |
| `exports` | Punto de entrada (reemplaza a `main`) | `{ ".": "./src/index.js" }` |
| `scripts` | Atajos de comandos | `{ "dev": "node --watch src/index.js" }` |
| `engines` | Versión mínima de Node | `{ "node": ">=24.0.0" }` |
| `dependencies` | Paquetes de producción | `"express": "^5.1.0"` |
| `devDependencies` | Paquetes solo de desarrollo | `"vitest": "^3.2.4"` |

### Módulos: ESM vs CommonJS

| | ESM (`"type": "module"`) | CommonJS |
|---|---|---|
| Importar | `import fs from 'node:fs'` | `const fs = require('fs')` |
| Exportar | `export default x` / `export { x }` | `module.exports = x` |
| ¿Sincrono? | No, siempre `await` | Sí, se puede hacer sync |
| Recomendado en 2026 | ✅ Sí | Solo para libs legacy |

```js
// ESM — import siempre con extensión de archivo
import { readFile } from 'node:fs/promises';
import express from 'express';

// Dynamic import (útil para carga condicional)
const { default: heavy } = await import('./heavy-module.js');
```

### Built-ins que se usan todos los días

| Módulo | Para qué |
|--------|----------|
| `node:fs/promises` | Leer/escribir archivos (API con promesas) |
| `node:path` | Unir rutas de forma portable (`path.join`) |
| `node:child_process` | Ejecutar comandos del sistema |
| `node:crypto` | Hashes, IDs aleatorios, `randomUUID()` |
| `node:os` | Info del sistema: CPU, memoria, `os.cpus()` |
| `node:url` | Parsear URLs |
| `node:util` | `util.inspect`, `util.promisify` |
| `node:test` | Test runner nativo, sin dependencias |

### Console y errores

```js
console.log('info');                 // info
console.error('error');              // stderr
console.table([{ a: 1 }, { a: 2 }]); // tabla
console.time('t'); /* ... */ console.timeEnd('t'); // 1.234ms

// Error con contexto y causa
try {
  await fetch(url);
} catch (err) {
  throw new Error(`Failed to fetch ${url}`, { cause: err });
}
```

### Variables de entorno

```bash
# .env (NO se sube al repo)
PORT=3000
DB_URL=postgres://localhost:5432/mydb
```

```js
// Node 20.6+ soporta --env-file. Node 24 lo tiene estable.
import { config } from 'node:process';

const port = Number(process.env.PORT ?? 3000);
```

```bash
node --env-file=.env src/index.js
```

> `.env` nunca va al repo. Suma `.env` a `.gitignore` y versiona `.env.example` con las claves vacías.

### Scriptstypical de `package.json`

```json
{
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "node src/index.js",
    "test": "node --test",
    "lint": "eslint ."
  }
}
```

---

## 💻 Ejemplos

### Servidor HTTP mínimo sin dependencias

```js
// server.js
import { createServer } from 'node:http';

const server = createServer((req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);

  if (url.pathname === '/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    return res.end(JSON.stringify({ status: 'ok', uptime: process.uptime() }));
  }

  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

```bash
node server.js &
curl http://localhost:3000/health
```

Salida:

```json
{"status":"ok","uptime":0.123456}
```

### Servidor con Express y validación de error

```js
import express from 'express';

const app = express();
app.use(express.json());

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (err) {
    next(err); // el middleware de error lo maneja
  }
});

// Middleware de error: SIEMPRE al final, con 4 argumentos
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal server error' });
});

app.listen(3000);
```

### Worker Threads: CPU-bound sin bloquear el event loop

```js
// worker.js
import { parentPort } from 'node:worker_threads';

parentPort.on('message', (data) => {
  // trabajo pesado de CPU
  parentPort.postMessage(data * 2);
});
```

```js
// main.js
import { Worker } from 'node:worker_threads';

const worker = new Worker(new URL('./worker.js', import.meta.url));
worker.postMessage(21);
worker.on('message', (result) => console.log(result)); // 42
```

### Test nativo (sin framework)

```js
// test/user.test.js
import { test } from 'node:test';
import assert from 'node:assert/strict';

test('divide by zero throws', () => {
  assert.throws(() => divide(1, 0), RangeError);
});

test('adds correctly', () => {
  assert.equal(add(2, 3), 5);
});
```

```bash
node --test
```

Salida:

```
✔ adds correctly
✔ divide by zero throws
```

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| `require` en un proyecto ESM | `ReferenceError: require is not defined` | Usa `import`, o `createRequire(import.meta.url)` |
| Olvidar la extensión en el import | `ERR_MODULE_NOT_FOUND` | En ESM el path es explícito: `./utils.js` |
| `node_modules` commiteado | Repo gigante e inseguro | `.gitignore` con `node_modules/` |
| `npm install` en vez de `npm ci` en CI | Instala versiones distintas al lock | `npm ci` en CI, siempre |
| Sin `catch` en un `await` | Unhandled rejection tumba el proceso | `try/catch` o `.catch()` |
| `process.env.PORT` sin validar | `NaN` si viene string raro | `Number(process.env.PORT ?? 3000)` |
| `console.log` en producción | Fuga de datos / ruido | Usa un logger con niveles |
| Confundir `null` con `undefined` | Bugs silenciosos | `??` en vez de `\|\|` para defaults |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| ESM sobre CommonJS | Estándar, `import` más claro, tree-shaking | Compatibilidad con libs viejas |
| `node --watch` sobre nodemon | Viene con Node, cero dependencias | Reinicia el proceso, no hot-reload de estado |
| Dependencias mínimas | Menos superficie de ataque, installs rápidos | Escribes más código tú |
| `node:test` sobre Jest | Sin dependencias, rápido | Ecosistema de matchers más chico |

---

## 📚 Further reading

- [Node.js Docs](https://nodejs.org/docs/latest-v24.x/api/) — API oficial de la LTS
- [ES Modules en Node](https://nodejs.org/api/esm.html) — las reglas de resolución
- [npm workspaces](https://docs.npmjs.com/cli/using-npm/workspaces) — monorepos
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices) — guía extensa de la comunidad
