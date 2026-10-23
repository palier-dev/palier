# Instructions for coding agents

Read this before working in the repository.

## What this project is
Palier is a public, educational project: building an AI-native solution for French copropriété (condominium) management, documented for engineers. The *way it is built* is the product, so clarity and traceability matter more than speed.

## Ground rules
- **Never commit real data.** No real names, addresses, lot numbers, amounts or documents. Real documents live only in `private/` (git-ignored). Use `data/synthetic/`.
- **Use the domain vocabulary** in `docs/domain/glossary.md`. Keep French domain terms (e.g. `appel_de_fonds`, `tantiemes`) in code and docs rather than inventing English equivalents; the glossary gives translations.
- **Record significant decisions** as an ADR in `docs/adr/` (copy the format of existing ones).
- **Legal content must cite its source** (law, article, version date). Never state a legal rule without a reference.
- Docs are Markdown, written for humans and agents alike: short sections, explicit headings, no unexplained jargon.

## Where things are
See the layout table in `README.md`.
