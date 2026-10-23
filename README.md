# Palier

**How to build a real AI-native solution from the ground up, using French condominium management (copropriété) as the case study.**

Palier is an open, non-commercial project. It is written for developers, including vibe coders. It documents every step of building a modern, mostly cloud-native solution that makes the property manager's (syndic) job easier, and gives owners and the conseil syndical more visibility and control over their building.

Why copropriété? It's a real business domain with law, budgets, meetings and many actors (owners, conseil syndical, gestionnaire, accountant, service providers, public services). That makes it a far better teaching case than yet another e-commerce demo.

> *Palier* (French): the landing where neighbours meet, and a step you climb one at a time.

## Repository layout

| Path | Content |
|---|---|
| `docs/domain/` | Domain primer for engineers: actors, yearly lifecycle, documents, glossary |
| `docs/discovery/` | Interviews, event storming, brainstorming, hypotheses |
| `docs/adr/` | Architecture Decision Records |
| `data/synthetic/` | Synthetic copropriété dataset (fake data only) |
| `data/legal/` | Legal corpus ingestion config (sources: Légifrance/PISTE, data.gouv.fr) |
| `evals/` | Evaluation sets and harnesses |
| `services/` | Application code (later) |
| `infrastructure/` | Local lab (Talos, Argo) and GCP |
| `site/` | Documentation website and blog (palier.dev) |

## Data policy

This repository contains **only synthetic or public data**. Real copropriété documents never enter the repo (`private/` is git-ignored).

## Status

Phase 1: domain discovery.

## License

- **Code**: [Apache License 2.0](LICENSE)
- **Documentation** (`docs/`, `site/` content) and **synthetic dataset** (`data/synthetic/`): [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE-CC-BY-4.0)
