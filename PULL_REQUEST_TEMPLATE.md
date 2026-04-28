## Summary

<!-- One precise paragraph: what changes, which subsystem, and why. Link the related issue. -->

Closes #

---

## Architecture Impact

- [ ] This PR crosses the CGo boundary (adds or modifies `import "C"` call sites)
- [ ] This PR modifies tensor allocation or `ggml_context` lifecycle
- [ ] This PR changes the OS signal handling or context teardown path
- [ ] This PR modifies the `ggml` / `llama.cpp` C++ layer
- [ ] This PR has no impact on the inference path, allocator, or CGo bridge

---

## Performance Metrics

**Required for any change touching the inference path, CGo bridge, ggml layer, allocator, or scheduler. Leave blank only for documentation-only PRs — and explain why below.**

### Benchmark Environment

| Field | Value |
|---|---|
| CPU | |
| GPU model | |
| VRAM capacity (GiB) | |
| OS + kernel version | |
| CUDA / Metal / ROCm driver version | |
| Go version | |
| Compiler (clang/gcc) version | |
| Benchmark model (name + quant) | |
| Context length | |

### Before / After Results

Run: `go test -bench=. -benchmem -count=10 ./...`

| Metric | Before | After | Δ |
|---|---|---|---|
| Time-to-First-Token — P50 (ms) | | | |
| Time-to-First-Token — P95 (ms) | | | |
| Generation Speed (tokens/sec) | | | |
| Inference Latency — P50 (ms/token) | | | |
| Inference Latency — P95 (ms/token) | | | |
| Peak VRAM Utilization (MiB) | | | |
| Go Heap Allocs per Inference | | | |
| RSS at Steady State (MiB) | | | |

Attach raw profiler output (`pprof` `.prof`, `ncu` report, `perf.data`, `valgrind` output) as a comment or CI artifact.

**If no performance measurement is required, justify here:**

<!-- e.g., "Documentation-only change with no production code path touched." -->

---

## Safety Checklist

- [ ] OS signal traps (`SIGINT`, `SIGTERM`, `SIGHUP`, `SIGPIPE`) tested — teardown drains all registered handles under this code path.
- [ ] Every new `C.malloc` / `C.CString` / `C.CBytes` call has a corresponding `C.free`, either in a `defer` or with documented ownership transfer.
- [ ] CGo contexts (`llama_context_p`, `ggml_backend_t`) are explicitly freed in a `defer` block registered immediately after successful allocation.
- [ ] Go objects passed across the CGo boundary are pinned with `runtime.Pinner` for the duration of all C calls.
- [ ] No `ggml_backend_buffer_t` is freed after its parent `ggml_backend_free` in any code path, including error paths.
- [ ] `os.Exit()` and `C.exit()` have **not** been introduced anywhere in this PR.
- [ ] No goroutine leaks — verified with `goleak` or `go test -race` showing no leaked goroutines after test teardown.
- [ ] Built and tested with `CGO_CFLAGS=-fsanitize=address,undefined` — zero ASAN/UBSan errors. *(Documentation PRs exempt.)*

---

## Test Coverage

- [ ] `go test ./... -race -count=1` passes with zero failures and zero race conditions.
- [ ] New/modified CGo wrapper functions have unit tests covering both the success path and error/nil-context edge cases.
- [ ] New/modified `ggml` kernels have numerical correctness tests (output compared against reference), including: zero-length tensors, max-dimension tensors, non-contiguous memory layouts.
- [ ] Signal/teardown path changes are covered by an integration test: spawn subprocess → deliver signal mid-inference → assert clean exit code + zero leaked file descriptors.

---

## Breaking Changes

<!-- Describe any breaking change to the CGo API, ggml ABI, CLI flags, or signal contract, and the migration path. Leave blank if none. -->

---

## Additional Context

<!-- Flamegraphs, `nsight` screenshots, links to relevant arXiv preprints, or any other reviewer context. -->
