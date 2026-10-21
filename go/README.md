# Chuletario Go

> Todo lo que necesitas para trabajar con Go en el día a día, en una sola página.

**TL;DR:** Go tiene un solo sistema de módulos (`go.mod`) y una sola forma de declarar dependencias. Los errores se manejan con valores de retorno y `%w`, no con excepciones. La concurrencia se construye con goroutines, canales y `context`. La versión actual es **go1.27.3** (verificada el 21 de octubre de 2026).

**Audiencia:** Este chulet asume que ya sabes lo básico de programación: funciones, structs y control de flujo. Si te falta base, empieza por el [tutorial oficial de Go](https://go.dev/doc/tutorial).

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| Go | `go1.27.3` | Publicada el 19 de octubre de 2026. Es la estable |
| `go.mod` | `go 1.27` | Versión mínima que el módulo requiere |
| `gofmt` | Incluido en el toolchain | Formateador oficial: `gofmt -w .` |
| `go vet` | Incluido en el toolchain | Analizador estático: `go vet ./...` |
| `go test` | Incluido en el toolchain | Runner de tests, benchmarks y race detector |
| `golangci-lint` | `v2.x` | Linter externo, opcional |

```bash
go version
# go version go1.27.3 linux/amd64
```

---

## 🚀 Setup

### Instalar Go

```bash
curl -LO https://go.dev/dl/go1.27.3.linux-amd64.tar.gz   # Official tarball
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.27.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
```

En Windows con `winget`: `winget install GoLang.Go`. En macOS con Homebrew: `brew install go`.

### Crear un proyecto y declarar dependencias

```bash
mkdir mi-app && cd mi-app
go mod init github.com/tu-usuario/mi-app
```

Esto crea `go.mod`:

```go
module github.com/tu-usuario/mi-app

go 1.27
```

| Directiva de `go.mod` | Para qué |
|------------------------|----------|
| `module` | Ruta del módulo. Debe coincidir con la URL del repositorio |
| `go 1.27` | Versión mínima de Go que el módulo necesita |
| `require` | Dependencias directas e indirectas con su versión exacta |
| `indirect` | Dependencia que no importas directamente pero necesitas |
| `go.sum` | Checksums de cada módulo: verifica que nadie lo adultere |

> `go 1.27` en `go.mod` no dice "compila con 1.27 y nada más". Es una versión **mínima**: cualquier Go más nuevo compila el módulo. Si la máquina tiene un Go más viejo, el toolchain descarga la versión correcta automáticamente (con `GOTOOLCHAIN=auto`, que es el valor por defecto).

```bash
go mod tidy                 # Add missing / remove unused dependencies
go mod download             # Download modules to the local cache
go mod verify               # Verify the checksums in go.sum
```

> Minimal Version Selection (MVS): Go elige **la versión más alta** que pidan las dependencias entre sí. Por eso una dependencia transitiva puede subir de versión sin que toques tu código.

### Layout de un proyecto

```text
mi-app/
├── cmd/mi-app/main.go   # Entry point: one binary per folder
├── internal/service/    # Code private to this module
└── pkg/                 # Code intended to be imported by others
```

> `internal/` es una convención del compilador, no una palabra clave: el código bajo `internal/` **no puede** importarse desde otro módulo. Sirve para que nadie dependa de tus detalles internos.

### Compilar, ejecutar e instalar

```bash
go run ./cmd/mi-app          # Run without creating a binary
go run .                     # Run the package in the current directory
go build ./cmd/mi-app        # Create the binary in the current directory
go build -o bin/mi-app ./cmd/mi-app  # Choose the output path
go install ./cmd/mi-app      # Install into $GOBIN (usually ~/go/bin)
go install golang.org/x/tools/cmd/goimports@latest  # Install a tool by version
go clean -cache              # Free the build cache
```

> `go run` y `go build` compilan en memoria o en un temporal: por eso el binario de `go install` es el que va al `PATH`. `go install pkg@version` (con `@`) instala fuera del módulo actual, sin tocar tu `go.mod`.

---

## 📖 Cheatsheet

### `struct`, métodos y receivers

| Concepto | Código |
|----------|--------|
| Definir | `type Point struct { X, Y float64 }` |
| Receiver por valor | `func (p Point) Norm() float64` (no muta el original) |
| Receiver por puntero | `func (p *Point) Move(dx float64)` (muta el original) |
| Constructor | `func NewPoint(x, y float64) *Point` |
| Stringer | `func (p Point) String() string` cambia cómo se imprime |
| Embedding | `type Base struct{}` embebido dentro de otra struct |
| Comparar | `==` compara campos; usa `slices.Equal` para structs con slices |

```go
type Point struct {
	X, Y float64
}

// Value receiver: the method gets a copy.
func (p Point) Norm() float64 { return math.Hypot(p.X, p.Y) }

// Pointer receiver: the method can mutate the original.
func (p *Point) Scale(factor float64) { p.X *= factor; p.Y *= factor }
```

> Regla práctica: usa receiver por puntero si el método muta, si la struct es grande, o si tiene campos de referencia (slices, maps, punteros). Si mezclas los dos estilos en la misma struct, `go vet` te lo señala.

### Interfaces: satisfacción implícita

| Regla | Detalle |
|-------|---------|
| No hay `implements` | Una struct implementa una interface si tiene todos sus métodos |
| Interfaces pequeñas | Una interface con un solo método es más fácil de probar y reutilizar |
| El consumidor define la interface | Quien provee el servicio solo la implementa |
| Interfaces en la stdlib | `io.Reader`, `io.Writer`, `fmt.Stringer` |
| `any` | Alias de `interface{}` desde Go 1.18 |

> Que la interface se satisfaga implícitamente evita el acoplamiento entre quien la declara y quien la implementa: puedes cambiar la implementación sin tocar el consumidor, y hasta sustituirla en tests por un fake.

### Errores

| Patrón | Código |
|--------|--------|
| Crear | `errors.New("not found")` |
| Error sentinel | `var ErrNotFound = errors.New("not found")` |
| Envolver con contexto | `fmt.Errorf("read %s: %w", path, err)` |
| Comparar | `errors.Is(err, ErrNotFound)` |
| Extraer el tipo | `var e *MyError; errors.As(err, &e)` |
| Unir errores | `errors.Join(err1, err2)` (Go 1.20+) |

> El `%w` es lo que hace que `errors.Is` funcione: sin él, el error envuelto es opaco y el consumidor solo puede compararlo por mensaje de texto, lo que se rompe en cuanto cambia el texto. `errors.Is` recorre la cadena de envolturas; `errors.As` extrae el error concreto con su tipo.

### Concurrencia: goroutines, canales y `select`

| Concepto | Detalle |
|----------|---------|
| `go f()` | Lanza una goroutine. El scheduler decide en qué hilo corre |
| `make(chan T)` | Canal sin buffer: el receptor espera al emisor |
| `make(chan T, n)` | Canal con buffer: `n` elementos sin bloquear |
| `close(ch)` | Cierra el canal. Solo el emisor cierra |
| `sync.WaitGroup` | Espera a que terminen N goroutines |
| `sync.RWMutex` | Protege datos compartidos |
| `select` | Espera varios canales a la vez; el que esté listo gana |
| `default` en `select` | No bloquea: es un intento no bloqueante |

```go
// select waits for whichever channel is ready first
select {
case msg := <-messages:
	fmt.Println("received:", msg)
case <-time.After(2 * time.Second):
	fmt.Println("timeout")
}
```

```go
var wg sync.WaitGroup
results := make(chan int, len(items))

for _, item := range items {
	wg.Add(1)
	go func(v Item) {
		defer wg.Done()
		results <- process(v)
	}(item)
}

wg.Wait()
close(results)

for r := range results {
	fmt.Println(r)
}
```

> Pasa el valor del bucle como argumento a la goroutine, no la variable: desde Go 1.22 la variable de rango es nueva en cada iteración, pero hacerlo explícito no depende de esa versión.

### `context` para cancelación y deadlines

| Función | Para qué |
|---------|----------|
| `context.Background()` | Contexto raíz, nunca se cancela |
| `context.WithCancel(ctx)` | Cancelación manual con `cancel()` |
| `context.WithTimeout(ctx, d)` | Deadline: se cancela solo pasado `d` |
| `ctx.Done()` | Canal que se cierra al cancelar |
| `ctx.Err()` | `context.Canceled` o `context.DeadlineExceeded` |

> `defer cancel()` no es opcional: sin él, el timer del contexto sigue vivo hasta que expire, y `go vet` lo marca con `lostcancel`. El contexto se propaga hacia abajo, nunca hacia arriba: el que llama decide si se cancela.

### `defer`, slices y maps

| Concepto | Detalle |
|----------|---------|
| `defer` | Se ejecuta al salir del scope, en orden LIFO |
| Argumentos de `defer` | Se evalúan al registrarlo, no al ejecutarlo |
| `[]int{}` | Slice: descriptor (puntero, len, cap) sobre un array |
| `make([]int, 0, 10)` | Slice con capacidad 10, sin copiar al hacer `append` |
| `copy(dst, src)` | Copia elementos entre slices del mismo tipo |
| `map[K]V` | Tabla hash. `make(map[K]V)` la crea |
| `v, ok := m[k]` | Consulta con indicador de existencia |
| `m == nil` | Un map nil se puede leer, pero no escribir |

> Un slice no guarda datos: guarda un puntero a un array, su longitud y su capacidad. Por eso `append` puede reasignar la memoria y dejar la variable original apuntando al array viejo. Y como los maps no son seguros para escritura concurrente, Go te obliga a pasar slices y maps por referencia explícito (`[]T`, `map[K]V`).

### Formateo, vet, tests y build tags

| Comando | Qué hace |
|---------|----------|
| `gofmt -w .` | Formatea el código en el sitio |
| `gofmt -l .` | Lista los archivos mal formateados |
| `go vet ./...` | Analizador estático (printf, mutex mal copiado, etc.) |
| `go test ./...` | Corre los tests de todos los paquetes |
| `go test -run TestName ./...` | Filtra por nombre |
| `go test -race ./...` | Detecta condiciones de carrera |
| `go test -bench=. -benchmem` | Benchmarks con asignaciones por operación |
| `//go:build linux` | Compila el archivo solo en Linux |
| `GOOS=linux GOARCH=arm64 go build` | Compila para otra plataforma |

> Las build tags son la forma idiomática de tener variantes de plataforma sin `if runtime.GOOS` por todas partes. El nombre del tag va **antes** del `package`, en su propia línea, con una línea en blanco después.

---

## 💻 Ejemplos

### Ejemplo 1 — struct + interface

```go
// main.go
package main

import "fmt"

type Notifier interface {
	Notify(message string) error
}

type ConsoleNotifier struct{ Sent int }

func (c *ConsoleNotifier) Notify(message string) error {
	c.Sent++
	fmt.Println("notified:", message)
	return nil
}

func notifyAll(n Notifier, messages []string) error {
	for _, m := range messages {
		if err := n.Notify(m); err != nil {
			return err
		}
	}
	return nil
}

func main() {
	n := &ConsoleNotifier{}
	if err := notifyAll(n, []string{"disk full", "backup done"}); err != nil {
		fmt.Println("error:", err)
	}
	fmt.Println("total sent:", n.Sent)
}
```

Salida:

```text
notified: disk full
notified: backup done
total sent: 2
```

### Ejemplo 2 — goroutines, canales y `sync.WaitGroup`

```go
// main.go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- string, wg *sync.WaitGroup) {
	defer wg.Done()
	for j := range jobs {
		results <- fmt.Sprintf("worker %d processed %d", id, j)
	}
}

func main() {
	jobs := make(chan int, 5)
	results := make(chan string, 10)

	var wg sync.WaitGroup
	for id := 1; id <= 2; id++ {
		wg.Add(1)
		go worker(id, jobs, results, &wg)
	}

	for j := 1; j <= 4; j++ {
		jobs <- j
	}
	close(jobs)

	wg.Wait()
	close(results)

	for r := range results {
		fmt.Println(r)
	}
}
```

Salida (el orden de las líneas cambia entre corridas):

```text
worker 2 processed 1
worker 1 processed 2
worker 2 processed 3
worker 1 processed 4
```

> El orden de impresión no está garantizado: cada worker consume del canal `jobs` en el orden en que el scheduler lo despierta. Lo que sí está garantizado es que los 4 trabajos se procesan exactamente una vez.

### Ejemplo 3 — errores envueltos y `errors.Is`

```go
// main.go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

type ValidationError struct {
	Field string
}

func (e *ValidationError) Error() string {
	return fmt.Sprintf("invalid field %q", e.Field)
}

func fetchUser(id int) error {
	if id <= 0 {
		return fmt.Errorf("fetchUser(%d): %w", id, &ValidationError{Field: "id"})
	}
	return fmt.Errorf("fetchUser(%d): %w", id, ErrNotFound)
}

func main() {
	var verr *ValidationError

	err := fetchUser(0)
	fmt.Println("is validation:", errors.As(err, &verr))
	fmt.Println("is not found:", errors.Is(err, ErrNotFound))
	fmt.Println("message:", err)

	err = fetchUser(7)
	fmt.Println("is not found:", errors.Is(err, ErrNotFound))
	fmt.Println("message:", err)
}
```

Salida:

```text
is validation: true
is not found: false
message: fetchUser(0): invalid field "id"
is not found: true
message: fetchUser(7): not found
```

> `errors.Is` y `errors.As` recorren la cadena de envolturas, así que funcionan aunque el error haya pasado por tres capas. El consumidor puede comprobar `ErrNotFound` sin saber qué función lo envoltió.

### Ejemplo 4 — tests

```go
// calculator_test.go
package calculator

import (
	"errors"
	"testing"
)

var ErrDivisionByZero = errors.New("division by zero")

func divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, ErrDivisionByZero
	}
	return a / b, nil
}

func TestDivide(t *testing.T) {
	cases := []struct {
		name    string
		a, b    float64
		want    float64
		wantErr bool
	}{
		{"positive", 10, 2, 5.0, false},
		{"by zero", 1, 0, 0, true},
	}

	for _, tc := range cases {
		t.Run(tc.name, func(t *testing.T) {
			got, err := divide(tc.a, tc.b)
			if tc.wantErr {
				if err == nil {
					t.Fatalf("expected error, got nil")
				}
				return
			}
			if err != nil {
				t.Fatalf("unexpected error: %v", err)
			}
			if got != tc.want {
				t.Errorf("got %v, want %v", got, tc.want)
			}
		})
	}
}
```

```bash
go test -v ./...
```

Salida:

```text
=== RUN   TestDivide
=== RUN   TestDivide/positive
=== RUN   TestDivide/by_zero
--- PASS: TestDivide (0.00s)
    --- PASS: TestDivide/positive (0.00s)
    --- PASS: TestDivide/by_zero (0.00s)
PASS
ok  	example.com/calculator	0.002s
```

> `t.Run` crea un subtest por caso: si uno falla, los demás siguen corriendo y el reporte dice exactamente cuál falló. `t.Fatalf` detiene el test actual; `t.Errorf` lo marca como fallido y sigue.

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| Ignorar el error de `err` | El programa sigue y falla más tarde, en otro lado | Revisa siempre `err`; ningún linter te obliga a hacerlo |
| Comparar errores envueltos con `==` | Nunca coincide: el error es otro valor | `errors.Is(err, ErrX)` |
| Olvidar `%w` en `fmt.Errorf` | Se pierde la cadena de causas | Usa `%w` cuando el error debe seguir siendo inspeccionable |
| `close(ch)` desde el receptor | `panic: close of closed channel` | Solo el emisor cierra el canal |
| Iterar sobre un canal sin cerrarlo | El `range` nunca termina | Cierra el canal cuando el emisor termina |
| Map leído y escrito a la vez | `fatal error: concurrent map read and map write` | Protégelo con `sync.RWMutex` |
| Mutar un slice recibido como parámetro | El llamador ve los cambios inesperados | Copia con `append([]T(nil), s...)` si vas a mutar |
| Lanzar goroutines sin `WaitGroup` | El programa termina antes de que terminen | Sincroniza con `sync.WaitGroup` o canales |
| `t.Fatal` dentro de una goroutine | Detiene solo esa goroutine, no el test | Usa `t.Error` y `return` dentro de goroutines |
| Comparar structs con slices con `==` | Error de compilación | Usa `reflect.DeepEqual` o `slices.Equal` |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| Errores como valores sobre `panic` | El compilador te obliga a considerar el caso | Hay que repetirlos en cada firma |
| Interfaces implícitas sobre `implements` | Desacoplamiento, sin código repetitivo | Los errores aparecen en tiempo de ejecución, no al compilar |
| Interfaces pequeñas sobre grandes | Fáciles de probar y de reutilizar | Más tipos que definir |
| `context` explícito sobre goroutines sueltas | Cancelación y deadlines uniformes | Hay que propagarlo por todas las firmas |
| Canales sobre variables compartidas | Sin data races por diseño | Más verboso y riesgo de deadlock si se bloquea |
| Módulos sobre `GOPATH` | Versionado exacto, compilaciones reproducibles | Un módulo por repositorio, sin vendor implícito |

---

## 📚 Further reading

- [Documentación oficial de Go](https://go.dev/doc/) — índice de la documentación de `go1.27`
- [A Tour of Go](https://go.dev/tour/) — el tutorial interactivo más rápido
- [Effective Go](https://go.dev/doc/effective_go) — las convenciones que la comunidad espera
- [Go by Example](https://gobyexample.com/) — ejemplos ejecutables tema por tema
- [errors en la stdlib](https://pkg.go.dev/errors) — la API de `errors`, `Is`, `As` y `Join`
