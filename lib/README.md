# Prebuilt libraries

The per-platform C ABI shared libraries live here under `<goos>_<goarch>/`
(`linux_amd64/`, `linux_arm64/`, `darwin_amd64/`, `darwin_arm64/`,
`windows_amd64/`, `windows_arm64/`). They are populated by the release
pipeline, which builds the C ABI hub for every target triple and commits the
binaries alongside this Go module so `go get` + `go build` works with no extra
steps (a C compiler is still required, as the binding uses cgo).

To build from this repository directly, compile the C ABI crate
(`cargo build -p wickra-darwin-c --release`) and stage the library into the
directory matching your `GOOS_GOARCH` — see the package README.
