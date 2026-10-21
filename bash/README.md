# Chuletario Bash

> Todo lo que necesitas para automatizar tu terminal con scripts robustos, en una sola página.

**TL;DR:** Bash es el intérprete de línea de comandos de GNU/Linux. Todo script serio empieza con `#!/usr/bin/env bash` y `set -euo pipefail`, entrecomilla siempre las variables, usa `[[ ]]` en vez de `[ ]` porque no rompe con espacios, y maneja los errores con `trap`. La versión de referencia es **Bash 5.2.37** (Debian GNU/Linux 13 "trixie"), verificada el **21 de octubre de 2026**.

**Audiencia:** Este chulet asume que ya usas la terminal con soltura (`ls`, `grep`, `pipe`, redirección). Si te sientes cómodo con eso, estás en condiciones de leerlo entero. Si todavía no, ve primero por el [chulet de Linux](./../linux/README.md).

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| Bash | `5.2.37` | Referencia: Debian GNU/Linux 13 "trixie". Verifica con `bash --version` |
| jq | `1.7` | Para manipular JSON en la línea de comandos |
| ShellCheck | `0.10+` | Analizador estático: `shellcheck script.sh` |

```bash
bash --version    # GNU bash, versión 5.2.37(1)-release (x86_64-pc-linux-gnu)
jq --version      # jq-1.7
```

> **Ojo con macOS:** el sistema trae Bash **3.2** (de 2009) preinstalado en `/bin/bash`. Bash 5.x existe en upstream pero no viene por defecto. Si usas arrays asociativos (`declare -A`), `mapfile` o `${var@Q}`, tu script explota en macOS. Por eso el shebang importa: `#!/usr/bin/env bash` busca el Bash que tengas primero en el `PATH`, y `#!/bin/bash` te ata a la versión vieja del sistema.

---

## 🚀 Setup

```bash
# Debian / Ubuntu
sudo apt-get install -y jq shellcheck curl

# macOS (además de Bash 5, que no viene por defecto)
brew install bash jq shellcheck curl
```

---

## 📖 Cheatsheet

### Modo estricto y estructura del script

| Línea | Qué hace | Por qué importa |
|-------|----------|-----------------|
| `#!/usr/bin/env bash` | Indica el intérprete | `env` busca el Bash del `PATH`; `/bin/bash` te ata a la versión del sistema |
| `set -e` | Aborta si un comando falla | Sin esto, los errores se acumulan y el script "termina bien" |
| `set -u` | Error si se usa una variable no definida | Detecta typos en nombres de variables |
| `set -o pipefail` | El pipeline falla si alguna parte falla | `set -e` solo mira el último comando del pipe |
| `set -x` | Imprime cada comando antes de ejecutarlo | Depuración; se desactiva con `set +x` |

Combinado en una línea, que es lo estándar: `set -euo pipefail`.

### Variables

| Concepto | Sintaxis | Alcance |
|----------|----------|---------|
| Global | `NAME="value"` | Toda la shell |
| Local | `local name="value"` (en una función) | Esa función y sus subfunciones |
| Solo lectura | `readonly NAME="value"` | No se puede reasignar ni desasignar |
| Exportar | `export NAME="value"` | La heredan los procesos hijos |
| Valor por defecto | `${VAR:-default}` | Usa `default` si `VAR` está vacía **o** no existe |
| Valor por defecto (existe pero vacía) | `${VAR-default}` | Usa `default` solo si `VAR` **no existe** |

```bash
# Diferencia entre :- y - (esto confunde a todo el mundo)
unset UNSET
echo "${UNSET:-fallback}"    # fallback
EMPTY=""
echo "${EMPTY:-fallback}"    # fallback  (está vacía)
echo "${EMPTY-fallback}"     # (vacío)   (existe, no aplica el default)
```

### Comillas: el tema que más bugs causa

| Forma | Interpretación | Cuándo usarla |
|-------|----------------|---------------|
| `"$var"` | Expande la variable, pero NO divide en palabras | **Siempre**, salvo razón justificada |
| `'$var'` | No expande nada, es texto literal | Strings con `$` que van a otro programa |
| `\$var` | Escapa el `$` dentro de comillas dobles | Igual que `'$var'` pero con comillas dobles |
| `"$@"` | Cada argumento como elemento separado | **Siempre** al pasar argumentos |
| `"$*"` | Todos los argumentos unidos en una sola palabra | Solo para joiner deliberado |
| `"${arr[@]}"` | Elementos de un array, separados | **Siempre** con arrays |

```bash
file="my report.txt"
rm $file      # MAL: dos argumentos, "my" y "report.txt"
rm "$file"    # BIEN: un argumento
```

> **Por qué `"$@"` y no `"$*"`:** con `"$*"` los argumentos se pegan en una sola cadena separada por el primer carácter de `IFS`, y un argumento que contiene un espacio se vuelve indistinguible de dos. `"$@"` preserva la frontera exacta de cada argumento, que es lo que quieres en el 99% de los casos.

### Arrays

| Tipo | Declaración | Acceso | Longitud |
|------|--------------|--------|----------|
| Indexado | `arr=(uno dos tres)` | `${arr[0]}`, `${arr[-1]}` (último) | `${#arr[@]}` |
| Indexado (append) | `arr+=(cuatro)` | | |
| Indexado (borrar) | `unset 'arr[1]'` | | |
| Asociativo | `declare -A mapa` | `${mapa[clave]}`, `${mapa[clave]:-default}` | `${#mapa[@]}` |

> `set -u` y arrays asociativos: `${mapa[clave]}` falla si la clave no existe. `${mapa[clave]:-default}` es la forma segura.

### Aritmética

| Operación | Sintaxis | Resultado |
|-----------|----------|-----------|
| Suma / resta | `$((1 + 2))`, `$((5 - 3))` | `3`, `2` |
| Multiplicación / división | `$((3 * 4))`, `$((10 / 3))` | `12`, `3` (división entera) |
| Módulo / exponente | `$((10 % 3))`, `$((2 ** 10))` | `1`, `1024` |
| Incremento / decremento | `((i++))`, `((++i))` | post/pre incremento |
| Comparación | `(( a > b ))` | código de salida 0 o 1 |

> **Trampa clásica:** `((count++))` devuelve código de salida 1 cuando `count` vale 0 (el valor post-incrementado es 0, que es "falso"). Con `set -e` eso aborta el script. Usa `count=$((count + 1))` o agrega `|| true`.

### Condicionales

| Forma | Uso | Nota |
|-------|-----|------|
| `[[ cond ]]` | Bash puro | No rompe con espacios, soporta `=~` y `&&`/`\|\|` |
| `[ cond ]` / `test` | POSIX | Requiere quoting estricto de las variables |
| `(( expr ))` | Aritmético | Devuelve 0/1 como código de salida |
| `-f` / `-d` / `-e` | Es archivo / directorio / existe | |
| `-s` | Existe y tiene tamaño > 0 | |
| `-z` / `-n` | String vacío / no vacío | |
| `-r` / `-w` / `-x` | Legible / escribible / ejecutable | |

### Bucles

| Tipo | Sintaxis | Cuando usarlo |
|------|----------|---------------|
| `for` sobre lista | `for x in a b c; do ... done` | Lista finita conocida |
| `for` sobre glob | `for f in *.txt; do ... done` | Archivos de un patrón |
| `while` | `while cond; do ... done` | Condición evaluada **antes** |
| `until` | `until cond; do ... done` | Condición evaluada **después**: se ejecuta al menos una vez |
| `read` en bucle | `while read -r line; do ... done < file` | Procesar línea por línea |
| Romper / seguir | `break`, `continue` | |

```bash
# El "|| [[ -n "$line" ]]" cubre el último renglón sin newline final.
while IFS= read -r line || [[ -n "$line" ]]; do
  printf '%s\n' "$line"
done < "/etc/hostname"
```

### Funciones y códigos de salida

| Concepto | Sintaxis | Nota |
|----------|----------|-----|
| Definir / llamar | `name() { ...; }` / `name arg` | Sin `function` ni `()` en la llamada |
| Éxito / error | `return 0` / `return 1` | `1` = error genérico |
| Argumentos | `$1`, `$2`, `"$@"` | |
| Capturar stdout | `result=$(name)` | Sin `set -e` el fallo no se detecta |
| Detectar fallo | `if ! name; then ...` | Captura `$?` correctamente |
| Códigos comunes | `0` éxito, `1` error, `2` uso incorrecto, `127` no encontrado, `130` Ctrl+C | `128+N` = señal N |

### Redirección y flujos

| Sintaxis | Dirección | Propósito |
|----------|-----------|-----------|
| `cmd > file` | stdout | Sobrescribe el archivo |
| `cmd >> file` | stdout | Añade al final |
| `cmd 2> file` | stderr | Aísla errores |
| `cmd > file 2>&1` | stdout+stderr | Orden importa: `2>&1` debe ir **después** |
| `cmd < file` | stdin | Lee desde archivo |
| `cmd 2>&1 \| tee log` | — | Muestra en pantalla y guarda |

> **`> file 2>&1` vs `2>&1 > file`:** en el primero, `stdout` ya apunta a `file` cuando se duplica `stderr`, así que ambos van al archivo. En el segundo, `stderr` se duplica cuando todavía apunta a la terminal, así que los errores siguen apareciendo en pantalla. El orden de las redirecciones no es cosmético: es la diferencia entre ver el error o no verlo.

### Here-documents

| Variante | Sintaxis | Para qué |
|----------|----------|----------|
| Here-doc | `cat <<EOF` | Texto multilínea, con expansión |
| Here-doc quoted | `cat <<'EOF'` | Texto literal, sin expansión |
| Here-doc indentado | `cat <<-EOF` (con `<<-`) | Quita los tabuladores iniciales |

### `getopts` — parseo de argumentos

| Elemento | Significado |
|----------|-------------|
| `getopts ":a:b"` | `:` al inicio = errores silenciosos, `a` sin `:` = flag, `b:` = requiere argumento |
| `OPTARG` | Valor del argumento de la opción actual |
| `OPTIND` | Índice del próximo argumento; se resetea con `OPTIND=1` |
| `shift $((OPTIND - 1))` | Elimina las opciones procesadas, deja los posicionales |

### `curl` + `jq` — JSON en la terminal

| Comando | Qué hace |
|---------|----------|
| `curl -sS --fail "$URL"` | Silencioso, muestra errores y sale con error en HTTP != 0 |
| `jq '.'` / `jq -c '.'` | Pretty-print / compacto |
| `jq '.a.b?'` | El `?` evita el error si el path no existe |
| `jq 'select(.x > 5)'` | Filtra elementos |
| `jq -r '.[] \| "\(.name)\t\(.stars)"'` | Interpola en un string |

### `find` con `-exec`

| Variante | Qué hace |
|----------|----------|
| `find . -name '*.log'` | Solo encuentra |
| `find . -name '*.log' -exec rm {} \;` | Ejecuta por archivo |
| `find . -name '*.log' -exec rm {} +` | Ejecuta con todos de una vez (más rápido) |

> `\;` termina un solo comando; `+` agrupa todos los resultados en una sola invocación. Con muchos archivos, `+` reduce drásticamente el número de procesos.

### Sustitución de comandos y `source`

| Forma | Propósito | Salida |
|-------|-----------|--------|
| `$(cmd)` | Captura stdout | String, se divide por `IFS` si no está entrecomillado |
| `` `cmd` `` | Variante antigua | Evitala: no anida bien |
| `<(cmd)` / `>(cmd)` | Process substitution | Pseudo-archivo (FD) en lectura o escritura |
| `source file` / `. file` | Ejecuta el archivo en la shell actual | Igual, forma POSIX |

> `source` no crea un subshell: las variables y funciones quedan en tu sesión. Es lo que quieres para un `.bashrc`, y lo que **no** quieres dentro de un script donde el efecto se perdería igual.

---

## 💻 Ejemplos

### Script con modo estricto, `usage` y `getopts`

```bash
#!/usr/bin/env bash
#
# summarize.sh — summarize the last N lines of a log file by severity
#
# Usage: summarize.sh [-n LINES] [-l LEVEL] FILE

set -euo pipefail

readonly PROGRAM_NAME="${0##*/}"
readonly DEFAULT_LINES=10
readonly VALID_LEVELS="debug info warn error"

usage() {
  cat <<EOF
Usage: $PROGRAM_NAME [-n LINES] [-l LEVEL] FILE

Show the last LINES entries of FILE that match at least the given severity.

Options:
  -n LINES   Number of entries to show (default: $DEFAULT_LINES)
  -l LEVEL   Minimum severity: debug|info|warn|error (default: info)
  -h         Show this help and exit
EOF
}

die() {
  printf 'error: %s\n' "$1" >&2
  exit "${2:-1}"
}

lines=$DEFAULT_LINES
level=info

while getopts ":n:l:h" opt; do
  case "$opt" in
    n) lines=$OPTARG ;;
    l) level=$OPTARG ;;
    h) usage; exit 0 ;;
    :) die "option -$OPTARG requires an argument" 2 ;;
    \?) die "unknown option: -$OPTARG" 2 ;;
  esac
done
shift $((OPTIND - 1))

# After shift, only positional arguments are left.
[[ $# -eq 1 ]] || { usage >&2; exit 2; }

readonly file=$1
[[ -f "$file" ]] || die "not a file: $file" 1
[[ "$lines" =~ ^[0-9]+$ ]] || die "LINES must be a positive integer" 2
[[ " $VALID_LEVELS " == *" $level "* ]] || die "LEVEL must be one of: $VALID_LEVELS" 2

# Severities are ordered, so "warn" must also match "error".
case "$level" in
  debug) pattern='debug|info|warn|error' ;;
  info)  pattern='info|warn|error' ;;
  warn)  pattern='warn|error' ;;
  error) pattern='error' ;;
esac

grep -E "\[($pattern)\]" "$file" | tail -n "$lines"
```

Ejecución:

```bash
chmod +x summarize.sh
./summarize.sh -h
./summarize.sh -n 3 -l warn app.log
```

Salida de `./summarize.sh -h`:

```text
Usage: summarize.sh [-n LINES] [-l LEVEL] FILE

Show the last LINES entries of FILE that match at least the given severity.

Options:
  -n LINES   Number of entries to show (default: 10)
  -l LEVEL   Minimum severity: debug|info|warn|error (default: info)
  -h         Show this help and exit
```

Salida de `./summarize.sh -n 3 -l warn app.log` sobre un log de ejemplo:

```text
[2026-10-21 09:12:04] [warn]  cache miss ratio above 40%
[2026-10-21 09:13:19] [error] upstream timeout after 30s
[2026-10-21 09:14:02] [warn]  retrying request (attempt 2/5)
```

### Procesar JSON con `curl` + `jq`

```bash
#!/usr/bin/env bash
#
# top-repos.sh — list the most starred repositories of an organization
#
# Usage: top-repos.sh [ORG] [LIMIT]

set -euo pipefail

readonly ORG="${1:-github}"
readonly LIMIT="${2:-5}"
readonly API_URL="https://api.github.com/orgs/${ORG}/repos?per_page=100"

# --fail makes curl exit non-zero on HTTP errors, so the pipeline aborts
# instead of piping an error body into jq.
curl -sS --fail --header "Accept: application/vnd.github+json" "$API_URL" \
  | jq -r --argjson limit "$LIMIT" '
      sort_by(-.stargazers_count)
      | .[:$limit]
      | .[]
      | "\(.stargazers_count)\t\(.name)\t\(.description // "no description")"
    '
```

Ejecución:

```bash
chmod +x top-repos.sh
./top-repos.sh github 3
```

Salida (columnas separadas por tabulador):

```text
206331	linux	Linux kernel source tree
133423	developer-roadmap	Interactive roadmaps, guides and other educational content
121915	awesome	Awesome lists about all kinds of interesting topics
```

### Renombrar y respaldar archivos con `trap` de limpieza

```bash
#!/usr/bin/env bash
#
# backup-files.sh — copy files to a timestamped backup directory, safely
#
# Usage: backup-files.sh FILE...

set -euo pipefail

readonly BACKUP_DIR="$PWD/.backup"
readonly STAMP="$(date +%Y%m%d-%H%M%S)"

# Track the staging file so the EXIT trap can remove it if anything fails.
staging=""

cleanup() {
  local exit_code=$?
  if [[ -n "$staging" && -e "$staging" ]]; then
    rm -f -- "$staging"
    printf 'cleanup: removed partial file %s\n' "$staging" >&2
  fi
  return "$exit_code"
}

# EXIT runs on normal exit, on error, and on an explicit exit.
trap cleanup EXIT
# INT/TERM handle Ctrl+C and SIGTERM with a dedicated code.
trap 'printf "interrupted\n" >&2; exit 130' INT TERM

[[ $# -ge 1 ]] || { printf 'Usage: %s FILE...\n' "${0##*/}" >&2; exit 2; }

mkdir -p "$BACKUP_DIR"

for file in "$@"; do
  [[ -f "$file" ]] || { printf 'error: not a regular file: %s\n' "$file" >&2; exit 1; }

  destination="$BACKUP_DIR/$(basename "$file").$STAMP.bak"

  # Copy to a staging name first, then move it into place. If the copy
  # fails halfway, the staging file is what the trap removes.
  staging="$BACKUP_DIR/.staging.$$"
  if ! cp --preserve=timestamps -- "$file" "$staging"; then
    printf 'error: could not back up %s\n' "$file" >&2
    exit 1
  fi
  mv -- "$staging" "$destination"
  staging=""

  printf 'backed up %s -> %s\n' "$file" "$destination"
done

printf 'done: %d file(s) in %s\n' "$#" "$BACKUP_DIR"
```

Ejecución:

```bash
chmod +x backup-files.sh
./backup-files.sh notes.txt report.csv
```

Salida:

```text
backed up notes.txt -> /home/user/project/.backup/notes.txt.20261021-091500.bak
backed up report.csv -> /home/user/project/.backup/report.csv.20261021-091500.bak
done: 2 file(s) in /home/user/project/.backup
```

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| Shebang `#!/bin/sh` en un script con arrays | `syntax error near unexpected token` en Bash | `#!/usr/bin/env bash`, y el shebang va en la primera línea |
| Olvidar comillas en `rm $file` | Con un espacio en el nombre, borra dos archivos | Siempre `"$file"` |
| `set -e` sin `set -u` ni `pipefail` | Los typos en variables pasan inadvertidos | `set -euo pipefail` de una vez |
| `set -e` dentro de una condición | El `if` neutraliza el `errexit`: el error no aborta | Maneja los errores explícitamente con `if ! cmd` |
| `set -e` en una pipeline | Solo falla si falla el último comando del pipe | `set -o pipefail` |
| `((count++))` con `count=0` | Devuelve código 1 y aborta el script con `set -e` | `count=$((count + 1))` o `((count++)) \|\| true` |
| `$@` sin comillas | Separa los argumentos con espacios | `"$@"` |
| `source` de un archivo que no existe | Aborta el script si usas `set -e` | `[[ -f "$f" ]] && source "$f"` |
| Heredoc sin comillas con `$` | Expande variables que no querías | `<<'EOF'` para texto literal |
| `grep` con un patrón que empieza con `-` | Lo interpreta como una opción | `grep -- "$pattern"` o `grep -e "$pattern"` |
| `find ... -exec rm {} \;` con miles de archivos | Un proceso por archivo, lentísimo | Usa `-exec rm {} +` o `-delete` |
| Leer un archivo con `read` sin `-r` | Los backslashes se interpretan | `read -r` |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| Bash sobre `sh` | Arrays, `[[ ]]`, `local`, `mapfile`, process substitution | No es POSIX: no corre en `dash` ni en Alpine por defecto |
| `set -euo pipefail` | Falla rápido ante errores silenciosos | Requiere reescribir scripts legacy que dependían de errores ignorados |
| `[[ ]]` sobre `[ ]` | No rompe con espacios, soporta regex y `&&` | No es POSIX |
| `jq` sobre `grep`/`sed` para JSON | Consulta estructura, no texto | Requiere el binario instalado |
| Funciones puras con `local` | Menos estado global, más testeable | `local` es un builtin de Bash, no de POSIX `sh` |
| `trap ... EXIT` | Limpieza garantizada ante error | Un `trap` mal escrito puede enmascarar el código de salida original |
| `source` en vez de ejecutar | Comparte variables con la sesión actual | Aísla menos: un error puede matar la shell interactiva |
| `getopts` sobre parseo manual | Menos código, errores consistentes | Solo opciones cortas; para opciones largas hace falta `case` sobre `$1` |

---

## 📚 Further reading

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html) — la referencia oficial, completa y precisa
- [Bash Guide](https://mywiki.wooledge.org/BashGuide) — guía introductoria con buenos ejemplos
- [BashFAQ](https://mywiki.wooledge.org/BashFAQ) — las preguntas frecuentes con respuestas de la comunidad
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html) — convención de estilo para scripts en repositorios
- [ShellCheck](https://www.shellcheck.net/) — analizador estático que detecta la mayoría de los gotchas de esta página
