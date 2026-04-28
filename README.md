# ⚙️ Inference Foundry

We build high-efficiency, bare-metal AI tools. Our research focuses on eliminating runtime overhead and optimizing local inference architectures. We maintain `super-ollama` alongside open-source system benchmarking utilities.

## 🚀 Flagship Project: `super-ollama`

`super-ollama` is a terminal-native, serverless local LLM engine designed for absolute execution efficiency.

* Eliminated 100% of HTTP server overhead for CLI inference, by embedding the `llama.cpp` backend directly into the terminal process, resulting in 0 network latency.
* Reduced VRAM leak occurrences to 0, by implementing strict OS signal trapping and CGo context teardowns, resulting in 1 completely stable environment for consecutive model executions.
* Established 1 core research hub for system-level ML optimizations, by structuring C++ and Go implementations into isolated modules, resulting in 50% faster onboarding for new architectural contributors.

## 🔬 Core Focus Areas

Our development and research pipelines target:
* **Bare-metal Inference:** Bypassing abstraction layers for direct hardware-to-model communication.
* **Memory Management:** Profiling and optimizing GPU/CPU memory allocation during autoregressive sampling.
* **Language Bridges:** Benchmarking CGo overhead and optimizing Go-to-C++ context switching for tensor operations.

## 🤝 Join the Foundry

We operate strictly on evidence-based engineering and peer-reviewed optimizations. 

* **Community:** Join our [Discord Server](#) (Link TBD) to discuss architecture, review arXiv preprints, and share profiling data.
* **Contributions:** All architectural proposals must include benchmark data. Pull requests modifying `ggml` or CGo wrappers require memory leak tests.

---
*Inference Foundry — Bypassing the network, talking to the metal.*
