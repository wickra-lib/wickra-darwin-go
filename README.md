# Wickra Darwin — Go

[![CI](https://github.com/wickra-lib/wickra-darwin/actions/workflows/ci.yml/badge.svg)](https://github.com/wickra-lib/wickra-darwin/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/wickra-lib/wickra-darwin/branch/main/graph/badge.svg)](https://codecov.io/gh/wickra-lib/wickra-darwin)
[![Go module](https://raw.githubusercontent.com/wickra-lib/.github/main/profile/badges/wickra-darwin/go.svg)](https://pkg.go.dev/github.com/wickra-lib/wickra-darwin-go)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT_OR_Apache--2.0-blue)](https://github.com/wickra-lib/wickra-darwin#license)

**Deterministic evolutionary strategy search for Go, over the Wickra C ABI hub via cgo.**

Wickra Darwin evolves trading strategies with a seeded genetic search — population,
mutation and fitness folded once in a Rust core, so the report is byte-identical
across every language for a given seed. This package is the Go binding; it consumes
the C ABI hub through cgo and drives the search over the same JSON protocol as
every other binding.

## Install

Use the published **`wickra-darwin-go`** module, which bundles the prebuilt
C ABI library for every platform, so `go get` + `go build` works with no extra
steps (a C compiler is still required, as the binding uses cgo):

```bash
go get github.com/wickra-lib/wickra-darwin-go
```

```go
import wickra "github.com/wickra-lib/wickra-darwin-go"
```

`wickra-darwin-go` is generated from the [`bindings/go`](https://github.com/wickra-lib/wickra-darwin/tree/main/bindings/go)
directory by the release pipeline: it mirrors the Go sources, the vendored C ABI
header (`include/wickra_darwin.h`) and the prebuilt libraries under
`lib/<goos>_<goarch>/`. On Linux/macOS the library path is baked in via rpath; on
Windows the DLL must be discoverable at run time (next to the executable or on
`PATH`).

### Building from this repository (contributors)

The `bindings/go` directory in the [wickra-darwin](https://github.com/wickra-lib/wickra-darwin)
repository is the development source. To build it directly, compile the C ABI and
stage the library into the per-platform directory cgo links against:

```bash
cargo build -p wickra-darwin-c --release
mkdir -p lib/linux_amd64                              # match your GOOS_GOARCH
cp target/release/libwickra_darwin.so    lib/linux_amd64/    # Linux
cp target/release/libwickra_darwin.dylib lib/darwin_arm64/   # macOS (arm64)
cp target/release/wickra_darwin.dll      lib/windows_amd64/  # Windows
```

## Quick start

```go
package main

import (
	"fmt"

	wickra "github.com/wickra-lib/wickra-darwin-go"
)

func main() {
	spec := `{"seed":1,"population":8,"generations":3,` +
		`"mutation_rate":0.2,"crossover_rate":0.6,"fitness":"sharpe",` +
		`"search_space":{"indicators":[{"name":"rsi","param_ranges":[{"min":2,"max":30,"step":1}]}],` +
		`"rules":"single_threshold","max_conditions":1},"elitism":1,"top":5}`

	darwin, err := wickra.New(spec)
	if err != nil {
		panic(err)
	}
	defer darwin.Close()

	data := `{"BTCUSDT":[{"time":1700000000,"open":100,"high":101,"low":99,"close":100.5,"volume":10}]}`
	resp, err := darwin.Command(`{"cmd":"evolve","data":` + data + `}`)
	if err != nil {
		panic(err)
	}
	fmt.Println(resp)
}
```

The search's PRNG lives only in the Rust core, so a given seed produces the
byte-identical report here and in every other binding. Every handle owns native
memory freed by `Close()`; a finalizer is wired as a backstop, but call `Close()`
(e.g. with `defer`) to release it promptly.

## Documentation

The full guides, quickstarts, and API reference live in the main repository and
documentation site:

- **Repository:** <https://github.com/wickra-lib/wickra-darwin>
- **Docs:** <https://wickra.org>
- **Runnable examples:** [`examples/go/`](https://github.com/wickra-lib/wickra-darwin/tree/main/examples/go)

Wickra ships native bindings for Python, Node.js, WASM and Rust, plus a
C ABI hub that any C-capable language (C, C++, C#, Go, Java, R) links against —
all exposing the same core from the shared, `unsafe`-forbidden Rust core.

## Security

Found a security issue? **Please don't open a public issue.** Report it privately
via the affected repository's *Security* tab (*"Report a vulnerability"*) or email
**support@wickra.org** with a subject line starting `[wickra security]`. Full
policy: <https://github.com/wickra-lib/wickra-darwin/blob/main/SECURITY.md>.

## Disclaimer

Wickra Darwin is analytics software, not a trading system. The strategies it
searches are deterministic transforms of the input data — they are not financial
advice and do not predict the market. Any use in a live trading context is at your
own risk. The library is provided **as is**, without warranty of any kind.

## License

Licensed under either of [Apache-2.0](https://github.com/wickra-lib/wickra-darwin/blob/main/LICENSE-APACHE)
or [MIT](https://github.com/wickra-lib/wickra-darwin/blob/main/LICENSE-MIT) at your option.
