# Contributing to Inference Foundry

## The Foundry Philosophy

Code in this organization is judged by three metrics and three metrics only: **inference latency (ms/token)**, **memory footprint (VRAM/RSS in MiB)**, and **runtime stability** (clean exits, zero leaked handles). Architectural elegance is secondary to measurable efficiency. If a proposed change cannot demonstrate improvement — or at minimum a deliberate, documented trade-off — in one of these dimensions, it will not merge.

We do not accept contributions driven by aesthetic preference, framework familiarity, or vague performance intuition. Bring data or don't bring a PR.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [CGo and Memory Rules](#cgo-and-memory-rules)
3. [Profiling Requirements](#profiling-requirements)
4. [Testing Requirements](#testing-requirements)
5. [Commit Standards](#commit-standards)
6. [Pull Request Process](#pull-request-process)

---

## Getting Started

### Build Prerequisites

| Dependency | Minimum Version | Purpose |
|---|---|---|
| Go | 1.22 | CGo build pipeline |
| Clang | 15 | C++17 support for `ggml` |
| CUDA Toolkit | 12.x | NVIDIA backend (optional) |
| ROCm | 6.x | AMD backend (optional) |
| Xcode / Metal SDK | 15+ | Apple backend (macOS only) |
| `valgrind` | Current | Heap profiling on Linux |
| `pprof` | Bundled with Go | CPU/memory profiling |

```bash
git clone --recurse-submodules https://github.com/Inference-Foundry/super-ollama.git
cd super-ollama
go build ./...
go test ./... -race -count=1
```

Confirm a clean baseline before writing a single line of code.

---

## CGo and Memory Rules

`super-ollama` embeds `llama.cpp` directly into the terminal process via CGo. There is no isolation boundary. An un-freed C pointer is your leak. A missed signal trap is a VRAM hole that outlives the process. These are not style issues — they are correctness issues, and **PRs violating the rules below will be rejected without further review.**

### The Hard Rules

**1. Every `C.malloc` has a `C.free`.**

Ownership of all C-heap allocations must be explicit and documented. If a function allocates, it either frees before returning or transfers documented ownership to the caller. There is no third option.

```go
// CORRECT: explicit free in defer
func loadModel(path string) (*C.llama_model, error) {
    cPath := C.CString(path)
    defer C.free(unsafe.Pointer(cPath))
    return C.llama_load_model_from_file(cPath, C.llama_model_default_params()), nil
}

// REJECTED: CString leaks on every call
func loadModel(path string) (*C.llama_model, error) {
    return C.llama_load_model_from_file(C.CString(path), C.llama_model_default_params()), nil
}
```

**2. CGo contexts are freed with `defer`, not inline.**

All `llama_context_p` and `ggml_backend_t` handles must be freed in a `defer` block registered immediately upon successful allocation. This guarantees cleanup on panic paths and early returns.

**3. Go objects passed to C are pinned.**

Any Go-managed memory passed across the CGo boundary must be pinned with `runtime.Pinner` for the full duration of the C call. Passing unpinned Go pointers to C is undefined behavior under the Go memory model. The GC is not your friend here.

**4. OS signal traps are non-negotiable.**

`super-ollama` installs handlers for `SIGINT`, `SIGTERM`, `SIGHUP`, and `SIGPIPE`. These handlers drain the context registry and release all GPU-side buffers before exit. **Any modification to the signal handling path must preserve this guarantee.** Do not use `os.Exit()` or `C.exit()` — both bypass deferred deallocations.

**5. `ggml_backend_buffer_t` is freed before `ggml_backend_free`.**

GPU buffers must be released in reverse-allocation order before the backend is torn down. A backend free that precedes its buffer frees is a use-after-free waiting to be triggered by a driver scheduler flush.

**6. No `ggml_tensor` freed outside its `ggml_context`.**

Tensors are owned by their context. Call `ggml_free(ctx)` to release the context and all tensors it owns. Individual tensor frees are not part of the `ggml` API contract.

---

## Profiling Requirements

### When Profiling Is Mandatory

Any PR that touches the inference path, the CGo bridge, the `ggml` backend, the memory allocator, or the signal teardown path **must include before/after profiling data.** This is not optional.

### Accepted Profiling Tools

| Tool | Use Case | Output Format |
|---|---|---|
| `go test -bench` | Go-layer latency and allocation count | `.txt` benchmark output |
| `pprof` | CPU flamegraph, heap allocation profile | `.prof` file + flamegraph SVG |
| `valgrind --tool=massif` | Heap growth over time | `massif.out.*` |
| ASAN (`-fsanitize=address`) | Memory safety, use-after-free | Build log |
| `nvidia nsight compute` (`ncu`) | CUDA kernel efficiency | `.ncu-rep` file |
| `rocprof` | ROCm kernel profiling | `.csv` / `.json` |
| `perf record` | CPU cycle attribution | `perf.data` |

### Required Metrics

All architectural PRs must report these metrics in a before/after table:

| Metric | Measurement Method |
|---|---|
| Inference latency P50/P95 (ms/token) | `go test -bench=BenchmarkInference -count=10` |
| Time-to-first-token (ms) | `go test -bench=BenchmarkTTFT -count=10` |
| Peak VRAM utilization (MiB) | `nvidia-smi --query` / `rocm-smi` / Metal Instruments |
| Go heap allocations per inference | `go test -bench=. -benchmem` |
| RSS at steady state (MiB) | `/proc/<pid>/status` VmRSS field |

Attach raw profiler output as a PR artifact. Screenshots of profiler UIs are supplementary, not primary evidence.

---

## Testing Requirements

| Change Type | Required Test |
|---|---|
| New / modified CGo wrapper | Unit test + `go test -race`; `valgrind --leak-check=full` or ASAN clean |
| `ggml` kernel modification | Numerical correctness test vs. reference; edge cases: zero-length tensor, max-dim tensor, non-contiguous layout |
| Signal handler / teardown | Integration test: spawn subprocess → send signal mid-inference → assert clean exit code + zero leaked file descriptors |
| Allocator / memory path | `go test -memprofile` before/after; heap delta reported in PR |
| Inference scheduler | P50/P95/P99 latency regression ≤ 2% of baseline |

```bash
# Full suite with race detector
go test ./... -race -count=1 -timeout 120s

# Benchmarks (minimum 10 iterations)
go test -bench=. -benchmem -count=10 ./...

# CGo memory safety
CGO_CFLAGS="-fsanitize=address,undefined" \
CGO_LDFLAGS="-fsanitize=address,undefined" \
  go test ./internal/llama/... -count=1

# Valgrind (Linux)
go build -o /tmp/super-ollama ./cmd/super-ollama
valgrind --leak-check=full --error-exitcode=1 \
  /tmp/super-ollama --model /path/to/model.gguf --prompt "test"
```

---

## Commit Standards

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Use the scopes below — do not invent new ones without maintainer approval.

| Scope | Target subsystem |
|---|---|
| `feat(ggml):` | New functionality in the `ggml`/C++ layer |
| `feat(cgo):` | New CGo wrapper functions or bridge changes |
| `feat(scheduler):` | Inference scheduling or batching logic |
| `fix(cgo):` | CGo memory safety, handle lifecycle bugs |
| `fix(signal):` | Signal trap or teardown path corrections |
| `fix(allocator):` | Memory allocator defects |
| `perf(ggml):` | Compute kernel performance improvements |
| `perf(allocator):` | Allocation strategy optimization |
| `perf(cgo):` | Reducing CGo boundary crossing frequency |
| `refactor(bridge):` | Internal restructuring without behavior change |
| `test(cgo):` | Tests for the CGo layer |
| `docs:` | Documentation only |

Breaking changes must include `BREAKING CHANGE:` in the commit footer:

```
perf(cgo): batch embedding calls to reduce boundary crossings by 40%

BREAKING CHANGE: EmbedTokens() signature changed — caller now passes a
slice; individual token embedding via the old single-token API is removed.
```

---

## Pull Request Process

1. Run `go test ./... -race -count=1` locally. Zero failures, zero races.
2. Run benchmarks and populate the performance table in the PR template.
3. Fill every section of the [PR template](PULL_REQUEST_TEMPLATE.md). Incomplete sections — especially the performance impact table — result in the PR being returned without review.
4. Request a review from a maintainer with expertise in the modified subsystem.
5. Do not force-push during an active review. Use fixup commits; squash on final approval.
6. Merge requires: one approving review + green CI matrix (all backends).
