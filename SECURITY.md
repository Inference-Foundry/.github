# Security Policy

## Supported Versions

Security fixes are applied to the latest tagged release and the `main` branch. We do not backport security patches to older releases.

| Branch / Tag | Supported |
|---|---|
| `main` | ✅ Yes |
| Latest release tag | ✅ Yes |
| Older release tags | ❌ No |

---

## Threat Model

`super-ollama` is a **local CLI tool**. It has no network server, no listening socket, and no HTTP/gRPC interface. It embeds `llama.cpp` directly into the terminal process via CGo. The attack surface is therefore local and input-driven.

### In Scope

The following classes of vulnerability are relevant to `super-ollama` and should be reported:

| Category | Description |
|---|---|
| **Malicious GGUF model execution** | A crafted `.gguf` file triggers arbitrary code execution, stack/heap overflow, or controlled memory corruption during model loading or tensor deserialization. |
| **Unauthorized memory access via CGo boundary** | A bug in the CGo bridge allows a Go caller to read or write memory outside the bounds of a C-allocated buffer, potentially exposing process memory or enabling code execution. |
| **Signal handler exploitation** | A crafted input sequence or timing attack manipulates the SIGINT/SIGTERM teardown path to cause use-after-free, double-free, or controlled corruption of C-side GPU buffers. |
| **Integer overflow in tensor dimension arithmetic** | Arithmetic on tensor dimensions (during `ggml_context` allocation or quantization unpacking) overflows and leads to under-allocated buffers with subsequent out-of-bounds writes. |
| **Privilege escalation via IPC or filesystem** | Any code path that writes to a privileged file, creates a SUID binary, or communicates with a privileged process in an exploitable way. |
| **Dependency vulnerability** | A known CVE in a vendored dependency (`llama.cpp`, `ggml`, or a Go module) that is directly exploitable through `super-ollama`'s exposed interface. |

### Explicitly Out of Scope

The following are **not** security vulnerabilities in `super-ollama` and will be closed without action:

- Network-based attacks (DDoS, SSRF, request smuggling, CORS misconfigurations). `super-ollama` has no network server. These attack surfaces do not exist.
- Authentication or authorization bypasses. There is no authentication layer.
- Issues that require physical access to the machine running `super-ollama` and are not meaningfully different from any other local process attack.
- Performance degradation caused by unusually large or malformed inputs that does not cross into exploitable memory corruption.
- Social engineering attacks.

---

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.** Public disclosure before a fix is available puts all users at risk.

### Preferred Channel

Use GitHub's private [Security Advisory](https://github.com/Inference-Foundry/super-ollama/security/advisories/new) reporting feature. This creates a private, encrypted thread visible only to maintainers.

Alternatively, email **security@inference-foundry.dev** with the subject line `[SECURITY] super-ollama — <brief description>`. Encrypt the message with our PGP key (key ID and fingerprint published on our [Discord server](https://discord.gg/inference-foundry) and Keybase).

### What to Include

Provide enough detail for a maintainer to reproduce and assess the vulnerability independently:

- A precise description of the vulnerability class (see In Scope table above).
- The exact CLI command, model file (or minimal reproducer), and environment required to trigger it.
- The observed behavior (crash, memory corruption, arbitrary execution, etc.) and why it is exploitable rather than merely a bug.
- Your assessment of the severity (CVSS score if you have one, or a plain-language impact statement).
- Any proof-of-concept code or crafted GGUF file (attach as an encrypted archive).

Incomplete reports will be triaged more slowly. Thorough reports get faster fixes.

---

## Response Process

| Step | Target Timeline |
|---|---|
| Acknowledgment of receipt | 48 hours |
| Initial severity assessment | 5 business days |
| Fix development and internal testing | Depends on severity; critical issues prioritized |
| Coordinated disclosure | Agreed with reporter; default 90 days from receipt |
| Public CVE assignment (if applicable) | At or after public disclosure |

We follow coordinated disclosure. We will work with you on the disclosure timeline and credit you in the security advisory unless you prefer to remain anonymous.

---

## Known Non-Issues

If you are unsure whether something is in scope, err on the side of reporting it privately. We would rather triage a non-issue confidentially than have a real vulnerability disclosed publicly before a patch is ready.
