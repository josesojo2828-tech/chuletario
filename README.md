# Chuletario

> Cheatsheets de referencia rápida para estudiantes: lo que necesitas copy-pastear sin bucear 200 páginas de documentación.

**TL;DR:** Siete chulets, uno por tecnología. Cada uno tiene comandos listos para copiar, ejemplos que se ejecutan, y una sección de errores comunes que te vas a encontrar. Empieza por el que necesitas y sigue.

---

## 📚 Índice de chulets

| Chulet | Tecnología | Versión de referencia | Qué te resuelve |
|--------|-----------|------------------------|-----------------|
| [Node.js](./nodejs/README.md) | JavaScript en el backend | 24 LTS (Krypton) | `npm`, módulos ESM/CJS, `package.json`, scripts, depuración |
| [Python](./python/README.md) | Python | 3.13+ (3.15 estable) | venv, `pip`, tipos, `pathlib`, async, debugging |
| [Go](./go/README.md) | Go | 1.27 | `go mod`, goroutines, channels, interfaces, `go vet` |
| [Linux](./linux/README.md) | Linux / terminal | Debian 13, kernel 6.x | permisos, procesos, red, `grep`/`find`, discos |
| [Bash](./bash/README.md) | Bash | 5.2+ | scripting, arrays, funciones, errores, `trap` |
| [HTML](./html/README.md) | HTML5 | 5.2 | semántica, accesibilidad, formularios, SEO |
| [CSS](./css/README.md) | CSS | 3 + funciones modernas | layout, flexbox, grid, custom properties, responsive |

---

## 🧭 Cómo usar este repo

Cada carpeta es autocontenida: `README.md` es el chulet. No hay build, ni dependencias, ni tests — es texto que se lee en el editor o en GitHub.

**Reglas de oro para leer un chulet:**

1. **Copiar, no memorizar.** El chulet existe para que no tengas que recordar la sintaxis exacta.
2. **Los ejemplos se ejecutan.** Cada bloque tiene su salida esperada debajo.
3. **Las versiones están pineadas.** Cada chulet dice con qué versión fue escrito. Si tu máquina tiene otra, la sección "Gotchas" te avisa.

### Convención de los chulets

Todos los chulets siguen la misma estructura para que puedas saltar directo a lo que buscas:

```
1. TL;DR              — el resumen en 3 líneas
2. Requisitos        — versiones exactas
3. Setup / instalar  — comandos listos para copiar
4. Cheatsheet        — tablas de sintaxis por tema
5. Ejemplos          — código completo + salida
6. Gotchas           — los errores que vas a cometer
7. Trade-offs        — cuándo NO usar esto
8. Further reading   — para profundizar
```

---

## 🤝 Cómo contribuir

Este repo es de estudiantes y para estudiantes. Si algo está mal, desactualizado o falta:

1. Edita el `README.md` de la carpeta correspondiente.
2. Guarda el cambio en una rama: `git checkout -b fix/chulet-python`.
3. Commit con mensaje en formato convencional: `fix(python): corregir ejemplo de venv`.
4. Abre un Pull Request describiendo qué estaba mal.

**Reglas de estilo para los chulets:**

- Español neutral y profesional. Sin voseo en el texto técnico de los ejemplos.
- Código y comentarios de código siempre en inglés.
- Todo bloque de código lleva su lenguaje (` ```bash `, ` ```python `).
- Todo ejemplo lleva su salida esperada.
- Nada de "simplemente haz X". Explica el porqué cuando la decisión sea contraintuitiva.
- Si algo depende de una versión, indícala.

---

## 📌 Versiones de referencia

Verificadas el **21 de octubre de 2026**:

| Herramienta | Versión | Nota |
|-------------|----------|------|
| Node.js | `v24.22.1` (LTS "Krypton") | La LTS actual. `v26.12.1` es "Current" (no LTS) |
| Python | `3.15.1` | `3.13` y `3.12` en soporte de seguridad |
| Go | `go1.27.3` | |
| Bash | `5.2.37` | Debian 13 "trixie" |
| Linux | kernel `6.12` | Debian GNU/Linux 13 |

---

## 📄 Licencia

Contenido educativo bajo licencia MIT. Ver [LICENSE](./LICENSE).

Hecho para estudiantes que no quieren googlear la misma sintaxis por quinta vez.
