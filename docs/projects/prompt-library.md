# Lexicon

> *The right words for the right model, versioned and open.*

**Planned** initiative: a curated, open collection of **fine-tuned and task-specific prompts** the community can reuse, together with honest **analysis** of when and why they work, how to adapt them, and what to watch out for.

## Status

Repository **TBD**—this page is the org-level placeholder until a public repo is created.

## Background

Prompt engineering sits at an awkward intersection: it feels informal yet it has real, measurable impact on output quality. Most prompts are written once, never shared, and lost when a project moves on. Lexicon treats prompts as first-class artifacts—versioned, attributed, evaluated, and openly licensed—so that knowledge can accumulate across the community instead of being rediscovered in private.

## Intended scope

### Catalog structure
- Prompts organized by **task category**: summarization, reasoning chains, code generation, retrieval augmentation, classification, creative writing, safety evaluation, and more
- Each prompt entry carries: purpose, target model family, version tag, license, authorship, and known caveats
- Versioned with semantic tags so downstream consumers can pin to a stable revision

### Analysis and evaluation
- Qualitative analysis: failure modes, model-specific quirks, sensitivity to minor rewording
- Quantitative benchmarks where applicable (e.g. accuracy on public datasets, latency vs. output length)
- Prompt injection and adversarial robustness notes—what the prompt is and is not hardened against
- Comparisons across model families to surface which patterns generalize vs. which are model-specific

### Prompt engineering patterns
- Few-shot and zero-shot templates and when to use each
- Chain-of-thought (CoT) and scratchpad patterns
- System-prompt design: tone, persona, constraint setting
- Structured output prompts (JSON schemas, Markdown tables, etc.)
- Retrieval-augmented generation (RAG) prompt scaffolding

### Integration
- Optional links to specific models or runtimes (e.g. local inference via [super-ollama](super-ollama.md))
- Machine-readable metadata format so tooling can index and search the catalog programmatically

## Key open questions

1. What is the minimum evaluation harness that gives a meaningful signal without being expensive to run?
2. How should the catalog handle prompts that are highly model-version-sensitive?
3. What licensing model best balances openness with attribution for community contributions?

## How to help

When the repository exists, contribute prompts and analysis through its workflow. Until then, coordinate via [members](https://github.com/Inference-Foundry/.github/blob/main/docs/members/README.md) (roster: [`.github-private`](https://github.com/Inference-Foundry/.github-private)) or open an issue in [Inference-Foundry/.github](https://github.com/Inference-Foundry/.github/issues).

Useful backgrounds: NLP evaluation, prompt engineering experience across multiple model families, technical writing, and comfort working with YAML/JSON metadata schemas.
