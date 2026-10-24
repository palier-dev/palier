# ADR 0004: Website hosting, fonts and DNS for palier.dev

- **Status:** Accepted
- **Date:** 2026-10-24

## Context
`palier.dev` was registered (ADR 0002) but served nothing. An empty or parked domain looks abandoned to anyone who finds the project, and the blog and documentation site does not exist yet. The project needs a public page now, at no cost and with no server to run, without blocking later choices for the real site.

The page uses the Chakra Petch typeface from the Palier logo. Loading fonts from Google Fonts sends each visitor's IP address to Google. A German court ruled that this, without consent, breaches the GDPR (LG München I, judgment of 20 January 2022, Az. 3 O 17493/20).

## Decision
- **Holding page:** publish a single static "under construction" page now (`site/`: logo, one-line pitch, link to `github.com/palier-dev`, contact `contact@palier.dev`), so `palier.dev` is never empty. It is replaced by the documentation and blog site when that exists.
- **Hosting:** **GitHub Pages**, served from a dedicated repository **`palier-dev/site`** (branch `main`, root). The source stays in this monorepo under `site/`; a workflow (`.github/workflows/sync-site.yml`) copies it to `palier-dev/site` on every change under `site/` on `main`. `site/CNAME` (`palier.dev`) sets the custom domain.
- **Fonts:** **self-hosted** in `site/fonts/` (Chakra Petch, SIL Open Font License, licence shipped as `site/fonts/OFL.txt`). The page makes no requests to third-party font or script services.
- **DNS and analytics:** **Cloudflare** (the registrar) manages DNS for `palier.dev` and provides basic analytics.

## Rationale
- GitHub Pages is free, needs no server, gives HTTPS on a custom domain and sits next to the code in the same organization.
- Self-hosting the font removes the IP transfer to Google, so the page needs no consent banner for fonts. It also makes the page independent of a third-party CDN.
- Keeping DNS at Cloudflare avoids moving the domain, and its basic analytics are enough to see whether anyone visits, without adding a tracking script.

## Consequences
- `site/` in this monorepo is the only place to edit the website. `palier-dev/site` is a build output: anything committed there directly is overwritten by the next sync.
- The sync pushes with an SSH deploy key that has write access to `palier-dev/site` only, stored as the `SITE_DEPLOY_KEY` Actions secret in this repository. Setup steps are at the top of the workflow file.
- A dedicated repository keeps Pages separate from the monorepo (which uses `/docs` for Markdown documentation) and is where the future docs and blog site can be built and published.
- GitHub Pages on a private repository needs a paid GitHub plan; on the Free plan, `palier-dev/site` must be public. This matches the project's public-repo policy.
- `site/.nojekyll` tells Pages to serve the files as they are, without a Jekyll build.
- Cloudflare DNS must point `palier.dev` to GitHub Pages, and "Enforce HTTPS" is enabled in the repository's Pages settings once the certificate is issued.
- If analytics move beyond Cloudflare's basic, cookieless figures, the consent requirements must be checked against the CNIL guidance on audience measurement before adding anything.
- The future documentation and blog site keeps the same rules: static hosting, self-hosted fonts, no third-party trackers by default.
