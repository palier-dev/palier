# ADR 0004: Website hosting, fonts and DNS for palier.dev

- **Status:** Accepted
- **Date:** 2026-10-24

## Context
`palier.dev` was registered (ADR 0002) but served nothing. An empty or parked domain looks abandoned to anyone who finds the project, and the blog and documentation site does not exist yet. The project needs a public page now, at no cost and with no server to run, without blocking later choices for the real site.

The page uses the Chakra Petch typeface from the Palier logo. Loading fonts from Google Fonts sends each visitor's IP address to Google. A German court ruled that this, without consent, breaches the GDPR (LG München I, judgment of 20 January 2022, Az. 3 O 17493/20).

## Decision
- **Holding page:** publish a single static "under construction" page now (`site/`: logo, one-line pitch, link to `github.com/palier-dev`, contact `contact@palier.dev`), so `palier.dev` is never empty. It is replaced by the documentation and blog site when that exists.
- **Hosting:** **GitHub Pages**, from the `palier-dev` GitHub organization, with a `CNAME` file containing `palier.dev`.
- **Fonts:** **self-hosted** in `site/fonts/` (Chakra Petch, SIL Open Font License, licence shipped as `site/fonts/OFL.txt`). The page makes no requests to third-party font or script services.
- **DNS and analytics:** **Cloudflare** (the registrar) manages DNS for `palier.dev` and provides basic analytics.

## Rationale
- GitHub Pages is free, needs no server, gives HTTPS on a custom domain and sits next to the code in the same organization.
- Self-hosting the font removes the IP transfer to Google, so the page needs no consent banner for fonts. It also makes the page independent of a third-party CDN.
- Keeping DNS at Cloudflare avoids moving the domain, and its basic analytics are enough to see whether anyone visits, without adding a tracking script.

## Consequences
- GitHub Pages publishes from a repository root, `/docs`, or a GitHub Actions workflow. `/docs` already holds the project documentation, so for now `site/` is published from this monorepo by a workflow (`.github/workflows/pages.yml`), triggered by changes under `site/` on `main`. A separate repository remains an option if the site outgrows the monorepo.
- With workflow deployment, GitHub ignores the `site/CNAME` file: the custom domain is set in the repository's Settings → Pages (Source: GitHub Actions; Custom domain: `palier.dev`). The file is kept as documentation of the intended domain.
- Cloudflare DNS must point `palier.dev` to GitHub Pages, and "Enforce HTTPS" is enabled in the repository's Pages settings once the certificate is issued.
- If analytics move beyond Cloudflare's basic, cookieless figures, the consent requirements must be checked against the CNIL guidance on audience measurement before adding anything.
- The future documentation and blog site keeps the same rules: static hosting, self-hosted fonts, no third-party trackers by default.
