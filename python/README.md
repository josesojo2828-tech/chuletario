# Chuletario Python

> Todo lo que necesitas para trabajar con Python en el día a día, en una sola página.

**TL;DR:** Python se gestiona con entornos virtuales (`python -m venv` o `uv`) y `pip` para dependencias. `pathlib` reemplaza a `os.path`, las f-strings formatean, las dataclasses modelan datos, `async/await` con `asyncio` cubre concurrencia y `pytest` es el estándar de testing. La versión actual es **Python 3.15.1** (verificada el 21 de octubre de 2026).

**Audiencia:** Este chulet asume que ya sabes lo básico: variables, `if`, bucles, funciones y clases. Si te falta base, el [tutorial oficial](https://docs.python.org/3/tutorial/) es el punto de partida real.

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| Python | `3.15.1` | Actual estable (liberada el 9 de octubre de 2026). `3.16` todavía es pre-release |
| Python | `3.14.x` | Soporte con correcciones de bugs |
| Python | `3.13` / `3.12` | Solo soporte de seguridad: ya no reciben features |
| pip | `26.2.2` | Viene con Python. Se actualiza con `python -m pip install -U pip` |
| uv | `0.13.4` | Alternativa rápida a `venv` + `pip`. Opcional |
| pytest | `9.1.1` | Test runner. Opcional |
| ruff | `0.17.1` | Linter y formatter. Opcional |

> **Recomendación para estudiantes:** apunta a **Python 3.13 o superior**. Cualquier biblioteca que uses en un curso debería declarar soporte para 3.13, así no te quedas con una versión que ya no recibe correcciones. Si tienes dudas sobre qué versión exige tu profesor, usa **3.15.1**: es la actual estable y ya está disponible en los gestores de paquetes.

```bash
python3 --version   # Python 3.15.1
pip --version       # pip 26.2.2 from /usr/lib/python3/dist-packages/pip
```

> `python3` en Linux y `python` en Windows o macOS con Homebrew suelen apuntar a lo mismo, pero no está garantizado. En scripts y en CI usa siempre `python3`.

---

## 🚀 Setup

### Entorno virtual con `venv` (estándar)

Un entorno virtual es un directorio con una copia aislada de los binarios de Python y sus paquetes. Aísla las dependencias del sistema: sin él, `pip install` escribe en `/usr/lib/python3/dist-packages` y rompe herramientas del sistema operativo.

```bash
mkdir mi-proyecto && cd mi-proyecto

# Create the virtual environment
python3 -m venv .venv

# Activate it (bash/zsh)
source .venv/bin/activate

# Confirm the active interpreter
which python
# .venv/bin/python
```

Para desactivarlo: `deactivate`. Borra `.venv` solo con `rm -rf .venv` y estando parado en la raíz del repo: recrearlo cuesta dos segundos, borrar el directorio equivocado cuesta el proyecto.

### Alternativa rápida: `uv`

`uv` hace lo mismo que `venv` + `pip`, pero resuelve dependencias en un orden de magnitud menos de tiempo: está escrito en Rust y cachea los wheels ya descargados. Además puede reemplazar `pip` dentro de un proyecto existente.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh   # Install uv

uv init mi-proyecto && cd mi-proyecto && uv venv   # Project + environment
uv add requests                                   # Dependency (writes pyproject.toml + uv.lock)
uv add --dev pytest ruff                          # Dev dependencies
uv sync                                           # Install everything from the lock file
uv run python main.py                             # Run
```

### Instalación de paquetes con `pip`

```bash
python -m pip install requests==2.32.3      # Install a specific version
python -m pip install "requests>=2.32,<3"    # Install within a range
python -m pip install -e .                    # Editable install of the current project
python -m pip install -r requirements.txt     # From a file
python -m pip list --outdated                # Packages with newer versions
python -m pip show requests                  # Metadata of one package
python -m pip freeze > requirements.txt      # Freeze the current state
```

> Usa `python -m pip` en vez de `pip` a secas: garantiza que el `pip` que se ejecuta es el del intérprete activo. Con `pip` suelto es fácil instalar en el Python del sistema y no en el venv.

### Estructura de proyecto recomendada

```text
mi-proyecto/
├── .venv/              # Virtual environment (never version it)
├── pyproject.toml      # Metadata, dependencies and tool config
├── src/mi_paquete/     # Package code (one folder = one package)
└── tests/              # pytest files
```

---

## 📖 Cheatsheet

### Ejecutar Python: `python -m` y el guard `if __name__ == "__main__"`

| Comando | Qué hace |
|---------|----------|
| `python main.py` | Ejecuta el archivo como script |
| `python -m mi_paquete` | Ejecuta el paquete como módulo (agrega el CWD al `sys.path`) |
| `python -m http.server 8000` | Servidor HTTP de una línea |
| `python -m pytest` | Corre los tests del proyecto |
| `python -m json.tool file.json` | Formatea JSON legible |

```python
def main() -> None:
    print("Hello")


if __name__ == "__main__":
    main()
```

> El guard existe porque un archivo importado no debe ejecutar su lógica principal: sin él, `import mi_modulo` correría el archivo entero, incluida la llamada a `main()`.

### `pathlib` — rutas multiplataforma

| Operación | Código |
|-----------|--------|
| Unir rutas | `Path("/var") / "log" / "app.log"` |
| Nombre, base, extensión | `p.name`, `p.stem`, `p.suffix` |
| ¿Existe? | `p.exists()`, `p.is_file()`, `p.is_dir()` |
| Crear directorio | `p.mkdir(parents=True, exist_ok=True)` |
| Listar | `p.iterdir()`, `p.glob("*.log")`, `p.rglob("*.py")` |
| Leer / escribir | `p.read_text()`, `p.write_text("data")` |
| Renombrar / borrar | `p.rename(p2)`, `p.unlink()` |

> `pathlib` no concatena cadenas con `/` a mano, funciona igual en Windows y en Linux, y devuelve objetos `Path` en vez de cadenas: por eso es la forma correcta de manipular rutas.

### f-strings y comprehensions

| Caso | Código |
|------|--------|
| Interpolar / expresión | `f"Hola {name}"`, `f"{a + b}"` |
| Formato numérico | `f"{price:.2f}"`, `f"{ratio:.1%}"` |
| Alineación | `f"{name:<10}\|"`, `f"{name:>10}\|"`, `f"{name:^10}\|"` |
| Debuggear rápido | `f"{value=}"` → `value=42` |
| List comprehension | `[f(x) for x in items if cond]` |
| Dict comprehension | `{k: f(v) for k, v in pairs}` |
| Generator expression | `(f(x) for x in items)` — lazy, sin materializar la lista |

```python
squares = [x * x for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]

index = {name: len(name) for name in ["ana", "benito"]}
# {'ana': 3, 'benito': 6}

total = sum(x * x for x in range(1_000_000))  # lazy: no allocates the list
```

> Usa un generator expression cuando solo vas a consumir el resultado una vez (dentro de `sum()`, `any()` o `max()`): la list comprehension materializa los 1.000.000 de elementos en memoria, el generador los produce de a uno.

### Métodos de `list` y `dict`

| Método | Efecto | Método | Efecto |
|--------|--------|--------|--------|
| `list.append(x)` | Agrega al final | `d[k] = v` | Asigna |
| `list.extend(it)` | Agrega varios | `d.get(k, default)` | Lee con default |
| `list.remove(x)` | Quita la primera coincidencia | `d.pop(k, default)` | Quita y devuelve |
| `list.pop()` | Quita el último | `d.update(other)` | Fusiona |
| `list.sort(key=fn)` | Ordena in-place | `d.setdefault(k, v)` | Inserta si falta |
| `sorted(x)` | Devuelve copia ordenada | `d \| other` | Unión (3.9+) |

### Type hints

| Anotación | Significado |
|-----------|-------------|
| `def f(x: int) -> str` | Tipo del parámetro y del retorno |
| `str \| None` | `str` o `None` (3.10+). `Optional[str]` es el alias antiguo |
| `list[int]`, `dict[str, int]` | Genéricos built-in (3.9+) |
| `Literal["a"]` / `TypedDict` | Literales / `dict` con forma conocida |
| `Sequence[str]`, `Mapping[str, int]` | Aceptan listas, tuplas y demás |
| `Protocol` | Interfaz estructural: duck typing con tipos |
| `Final` / `Self` | Constante / tipo de la subclase actual (3.11+) |
| `type Alias = int` / `@override` | Alias de tipo (3.12+) / marca un override |

```python
from typing import Protocol


class Repository(Protocol):
    def get(self, key: str) -> str | None: ...


class MemoryRepository:
    def __init__(self) -> None:
        self._data: dict[str, str] = {}

    def get(self, key: str) -> str | None:
        return self._data.get(key)
```

> Las anotaciones no se evalúan en tiempo de ejecución: son documentación para mypy, los IDE y `dataclass`. `Optional[str]` quedó en desuso porque `str | None` es más corto de leer.

### `dataclass`

| Parámetro | Para qué |
|-----------|----------|
| `@dataclass` | Genera `__init__`, `__repr__` y `__eq__` |
| `frozen=True` | Instancia inmutable (hashable) |
| `slots=True` / `kw_only=True` | Menos memoria (3.10+) / solo keyword (3.10+) |
| `field(default_factory=list)` | Default mutable, uno por instancia |
| `__post_init__` | Validación o post-procesado al crear |
| `asdict(obj)` / `replace(obj, x=1)` | A `dict` / copia modificada (3.13+) |

> `default_factory` es obligatorio para defaults mutables: `tags: list[str] = []` comparte la misma lista entre todas las instancias, porque el default se evalúa una sola vez al importar el módulo. Es un bug clásico y silencioso.

### Excepciones y context managers

| Patrón | Código |
|--------|--------|
| Lanzar | `raise ValueError("mensaje")` |
| Lanzar con causa | `raise RuntimeError("fallo") from exc` |
| Capturar | `try: ... except ValueError as e: ...` |
| Capturar varios | `except (TypeError, KeyError): ...` |
| Capturar todo | `except Exception as e:` (nunca `except:` a secas) |
| Limpiar siempre | `finally: ...` |
| Grupos de excepciones | `except* ValueError:` (3.11+) |
| Añadir contexto | `exc.add_note("contexto")` (3.11+) |
| Ignorar / cerrar | `with contextlib.suppress(E):` / `with open(p) as f:` |

```python
from contextlib import contextmanager


@contextmanager
def timer(name: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        print(f"{name}: {time.perf_counter() - start:.3f}s")
```

> `raise ... from exc` conserva la causa original en la traza. Sin eso, el mensaje real (por ejemplo, la línea donde falló el parseo de JSON) se pierde y la traza empieza por el error que inventaste. El `with` garantiza que `__exit__` se ejecute aunque el bloque lance una excepción: un `open()` sin `with` deja el archivo abierto y en un servidor eso termina en "too many open files".

### `async/await` y `asyncio`

| Concepto | Detalle |
|----------|---------|
| `async def` / `await` | Define una corrutina / espera su resultado |
| `asyncio.run(main())` | Punto de entrada del loop de eventos |
| `asyncio.gather(...)` | Corre corrutinas concurrentemente |
| `asyncio.TaskGroup()` | Grupo de tareas con cancelación limpia (3.11+) |
| `asyncio.timeout(5)` | Límite de tiempo (3.11+) |
| `asyncio.to_thread(fn)` | Ejecuta una función bloqueante en un thread |
| `asyncio.Queue` / `Lock` / `Event` | Cola y sincronización |

```python
async def main() -> None:
    async with asyncio.timeout(5):
        results = await asyncio.gather(fetch("a", 0.1), fetch("b", 0.2))
    print(results)
```

> `await` no crea un hilo: cede el control al loop de eventos, que corre otras corrutinas mientras espera. El beneficio de `async` está en el I/O, no en la CPU: para CPU-bound, el GIL sigue mandando y conviene `asyncio.to_thread` o `multiprocessing`.

### `pytest` — testing

| Comando | Qué hace |
|---------|----------|
| `pytest` / `pytest -q` | Corre todos los tests / salida compacta |
| `pytest tests/test_x.py::test_name` | Un test puntual |
| `pytest -k "keyword"` | Filtra por nombre |
| `pytest -x` | Se detiene en el primer fallo |
| `pytest --lf` | Solo los que fallaron en la corrida anterior |
| `pytest -m slow` | Corre por marcador |
| `pytest --cov=src` | Cobertura (requiere `pytest-cov`) |

| Fixture / helper | Para qué |
|------------------|----------|
| `tmp_path` | Directorio temporal por test (se borra solo) |
| `capsys` | Captura `stdout` y `stderr` |
| `monkeypatch` | Parchea atributos y variables de entorno |
| `pytest.raises(E)` | Verifica que se lance una excepción |
| `@pytest.mark.parametrize` | Casos de una función |

---

## 💻 Ejemplos

### Ejemplo 1 — `pathlib`: contar y resumir archivos

```python
# report.py
from pathlib import Path


def summarize(root: Path, suffix: str) -> tuple[int, int]:
    """Return (file_count, total_bytes) for files matching suffix."""
    files = [p for p in root.rglob(f"*{suffix}") if p.is_file()]
    return len(files), sum(p.stat().st_size for p in files)


if __name__ == "__main__":
    count, size = summarize(Path("."), ".py")
    print(f"{count} python files, {size / 1024:.1f} KiB")
```

```bash
python report.py
```

Salida (en un proyecto con 12 archivos `.py`):

```text
12 python files, 84.3 KiB
```

### Ejemplo 2 — `dataclass` con validación

```python
# inventory.py
from dataclasses import dataclass, field, asdict


@dataclass(frozen=True, slots=True)
class Product:
    name: str
    price: float
    stock: int = 0
    tags: list[str] = field(default_factory=list)

    def __post_init__(self) -> None:
        if self.price < 0:
            raise ValueError(f"price cannot be negative: {self.price}")


if __name__ == "__main__":
    book = Product("Dune", 18.5, stock=3, tags=["sci-fi"])
    print(asdict(book))
    try:
        Product("Broken", -1.0)
    except ValueError as exc:
        print(f"error: {exc}")
```

Salida:

```text
{'name': 'Dune', 'price': 18.5, 'stock': 3, 'tags': ['sci-fi']}
error: price cannot be negative: -1.0
```

### Ejemplo 3 — `asyncio`: tres consultas concurrentes

```python
# fetch_all.py
import asyncio


async def fetch(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"{name} done"


async def main() -> None:
    async with asyncio.timeout(5):
        results = await asyncio.gather(
            fetch("api-1", 0.3),
            fetch("api-2", 0.1),
            fetch("api-3", 0.2),
        )
    for line in results:
        print(line)


if __name__ == "__main__":
    asyncio.run(main())
```

Salida:

```text
api-1 done
api-2 done
api-3 done
```

> `asyncio.gather` devuelve los resultados en el orden de los argumentos, no en el de finalización: el segundo termina primero (0.1 s) pero sigue siendo el segundo de la lista. El tiempo total es ~0.3 s, no 0.6 s, porque las tres corrutinas corren concurrentemente.

### Ejemplo 4 — `pytest`: fixtures y parametrize

```python
# test_calculator.py
import pytest
from calculator import divide


@pytest.mark.parametrize(
    "a,b,expected",
    [(10, 2, 5.0), (7, 2, 3.5)],
)
def test_divide(a, b, expected):
    assert divide(a, b) == pytest.approx(expected)


def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(1, 0)


def test_config_file_is_read(tmp_path, capsys):
    path = tmp_path / "config.txt"
    path.write_text("42\n", encoding="utf-8")
    value = int(path.read_text(encoding="utf-8").strip())
    print(f"loaded {value}")
    assert value == 42
    assert "loaded 42" in capsys.readouterr().out
```

```bash
python -m pytest -q
```

Salida:

```text
....                                                     [100%]
4 passed in 0.12s
```

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| `pip install` sin entorno virtual | Instala en el Python del sistema y rompe herramientas del SO | `python -m venv .venv` + `source .venv/bin/activate` |
| `pip` en vez de `python -m pip` | Puede instalar en otro intérprete distinto al activo | Usa siempre `python -m pip` |
| `list[str] = []` como default en dataclass | Todas las instancias comparten la misma lista | `field(default_factory=list)` |
| `except:` sin tipo | Captura también `KeyboardInterrupt` y `SystemExit` | `except Exception:` |
| Perder la causa original | La traza muestra tu error, no la causa real | `raise NewError(...) from exc` |
| `open()` sin `with` | El archivo queda abierto si el bloque falla | `with open(...) as f:` |
| `await` fuera de `gather` | Las tareas se serializan y pierdes concurrencia | `asyncio.gather(...)` o `TaskGroup` |
| `asyncio.run()` dentro de un loop activo | `RuntimeError: asyncio.run() cannot be called from a running event loop` | Usa `await main()` si ya hay loop |
| F-string con `{}` literal | `f"{}"` es un error de sintaxis | `f"{{}}"` |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| `venv` estándar sobre `uv` | Cero instalación extra, funciona con cualquier Python | Resuelve dependencias mucho más lento |
| `uv` sobre `venv` + `pip` | Resuelve en segundos, cachea, gestiona lockfiles | Dependencia extra y un modelo de proyecto propio |
| `pyproject.toml` sobre `requirements.txt` | Un solo archivo, metadata estándar, config de herramientas | Necesitas herramientas que lo entiendan (pip 25+ ya lo hace) |
| `pathlib` sobre `os.path` | Multiplataforma, API orientada a objetos | Más verboso para casos triviales |
| f-strings sobre `%` o `.format()` | Legibles, formatos built-in, más rápidas | No permiten interpolar expresiones arbitrarias |
| `dataclass` sobre `NamedTuple` | Mutabilidad opcional, defaults, `slots` | No es hashable salvo con `frozen=True` |
| `asyncio` sobre `threading` | Miles de corrutinas con poca memoria | No sirve para CPU-bound |
| `Protocol` sobre `ABC` | Sin herencia, duck typing con tipos | Los errores de tipos se detectan más tarde |

---

## 📚 Further reading

- [Documentación oficial de Python](https://docs.python.org/3.15/) — referencia completa de la versión actual
- [Tutorial de Python](https://docs.python.org/3/tutorial/) — el punto de partida real
- [PEP 20 — Zen de Python](https://peps.python.org/pep-0020/) — los principios que explican el diseño del lenguaje
- [Documentación de `pathlib`](https://docs.python.org/3/library/pathlib.html) — la API completa
- [Documentación de uv](https://docs.astral.sh/uv/) — el gestor de paquetes y entornos de Rust
