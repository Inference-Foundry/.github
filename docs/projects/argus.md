# Argus

> *A hundred eyes, open.*

**Planned** initiative: algorithms and tooling to **detect AI-generated images**, using representation learning—particularly **JEPA (Joint Embedding Predictive Architecture)**—as a foundation for building reliable, model-agnostic detectors.

## Status

Repository **TBD**—this page is the org-level placeholder until a public repo is created.

## Background

Generative image models (diffusion, GAN, and autoregressive) have reached a quality level where human inspection is unreliable as a detection strategy. The problem is not purely perceptual; it is statistical and representational: AI-generated images leave systematic traces in frequency domains, semantic consistency, and learned feature spaces that are invisible to the eye but legible to well-trained detectors.

Argus is Inference Foundry's attempt to study and build these detectors openly—releasing code, datasets, and findings so the community has trustworthy, reproducible tools rather than black-box API calls.

The project takes its name from Argus Panoptes, the many-eyed giant of Greek myth: tireless, all-seeing, and impossible to fool while awake.

## Why JEPA

JEPA (Joint Embedding Predictive Architecture), introduced by Yann LeCun's group at Meta, learns to predict representations of image regions from context—without relying on pixel-level reconstruction. Key properties that make it attractive for detection:

- **Rich abstract representations:** I-JEPA and V-JEPA encoders learn features at a semantic level above low-level textures. Real and AI-generated images that look similar to the eye may diverge sharply in this representation space.
- **No generative bias:** Because JEPA is not a generative model itself, its feature space is not confounded by familiarity with specific generator architectures (unlike detectors trained against a fixed GAN or diffusion model).
- **Strong transfer:** Pre-trained JEPA encoders have demonstrated competitive transfer on recognition benchmarks with minimal fine-tuning, suggesting the representations are general enough to probe for unnatural statistical patterns.
- **Open weights:** Meta has released I-JEPA and V-JEPA checkpoints, making it practical to experiment without large-scale pre-training.

The hypothesis is that a lightweight probe (linear classifier or small MLP) on top of frozen or lightly fine-tuned JEPA representations will generalise across generator families better than detectors trained end-to-end on pixel inputs.

## Intended scope

### Research and theory
- Survey of existing detection approaches: frequency-domain methods (DCT artifact analysis, Fourier spectrum anomalies), GAN-specific fingerprint methods, CLIP-based semantic consistency tests, and diffusion-specific watermark/artifact studies
- Analysis of why detectors trained on one generator family fail on others (the generalisation problem)
- Theoretical grounding for why JEPA representations may be more generator-agnostic
- Literature review of competing architectures (DINO, MAE, CLIP) as detection backbones

### Experiments
- Baseline detectors: frequency-domain classifiers, fine-tuned CLIP probes, and a CNN baseline trained on raw pixels
- JEPA-based detector: linear probe and lightweight MLP on I-JEPA / V-JEPA frozen features; comparison of fine-tuned vs. frozen encoder performance
- Cross-generator generalisation tests: train on images from one generator family (e.g. Stable Diffusion), evaluate on another (e.g. DALL·E, Midjourney, Flux)
- Robustness evaluation: JPEG compression, resizing, colour jitter, social-media-style post-processing pipelines
- Dataset experiments on publicly available benchmarks: GenImage, ArtiFact, FakeSet, and RAISE

### Tooling
- Reproducible training and evaluation pipelines (Python / PyTorch)
- Lightweight inference wrapper so detection can run locally without a network call
- Integration notes for [super-ollama](super-ollama.md) if multimodal pipeline support matures
- Dataset curation guidelines and a documented augmentation strategy to avoid spurious correlations

### Responsible use
- Documentation of known failure modes and generalisation limits—Argus should not be presented as a ground-truth oracle
- Discussion of adversarial images crafted to evade detection and what that means for real-world use
- Guidance on ethical deployment: provenance claims, due-process considerations, and the difference between "likely AI-generated" and "definitively AI-generated"

## Key open questions

1. Do JEPA representations expose cross-generator artefacts that CLIP or DINO representations do not, and to what extent?
2. What is the minimum amount of labelled data required to train a probe that generalises reasonably across generator families?
3. How does detection accuracy degrade under common post-processing (compression, upscaling) and at what point is it no longer reliable?
4. Can the detector be distilled into a form small enough to run on-device for local-first privacy-preserving pipelines?

## How to help

When the repository exists, use its issues and contributing guide. Until then, coordinate with maintainers (see [members](https://github.com/Inference-Foundry/.github/blob/main/docs/members/README.md); roster in [`.github-private`](https://github.com/Inference-Foundry/.github-private)) or open an issue in [Inference-Foundry/.github](https://github.com/Inference-Foundry/.github/issues).

Useful backgrounds: computer vision, representation learning, PyTorch, experience with evaluation pipelines and dataset curation, and interest in media authenticity or AI safety at the applied layer.
