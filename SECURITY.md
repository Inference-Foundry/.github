# Security policy

## Reporting

**Do not** use public issues for unfixed, exploitable vulnerabilities. Report privately so maintainers can fix and coordinate disclosure.

**Preferred:** open a **private security advisory** on the affected repository (GitHub: *Security* → *Report a vulnerability*).

**Email:** **security@inference-foundry.dev** — include which product/repo, summary, reproduction if possible, and severity assessment.

For **super-ollama** specifically (local CLI, CGo / `llama.cpp`), use [super-ollama security reporting](https://github.com/Inference-Foundry/super-ollama/blob/main/SECURITY.md) for in-scope threat classes and advisory links.

## Supported versions

Unless a repository states otherwise, security fixes target the latest release tag and the default branch (`main`). Older tags are generally not backported.

## Product-specific policies

Detailed threat models and response expectations live with each product:

| Repository | Policy |
|------------|--------|
| **super-ollama** | [SECURITY.md in super-ollama](https://github.com/Inference-Foundry/super-ollama/blob/main/SECURITY.md) |
| **Others** | Each repo should add `SECURITY.md` when it gains maintainers and a release process. |

This file is the organization **default** for repositories that do not define their own `SECURITY.md`.
