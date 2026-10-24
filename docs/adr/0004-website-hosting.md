# ADR 0004: Website hosting, fonts and DNS for palier.dev

- **Status:** Accepted
- **Date:** 2026-10-24

## Context
`palier.dev` was registered (ADR 0002) but served nothing. An empty or parked domain looks abandoned to anyone who finds the project, and the blog and documentation site does not exist yet. The project needs a public page now, at no cost and with no server to run, without blocking later choices for the real site.

The page uses the Chakra Petch typeface from the Palier logo. Loading fonts from Google Fonts sends each visitor's IP address to Google. A German court ruled that this, without consent, breaches the GDPR (LG München I, judgment of 20 January 2022, Az. 3 O 17493/20).

## Decision
- **Holding page:** publish a single static "under construction" page now (`site/`: logo, one-line pitch, link to `github.com/palier-dev`, contact `contact@palier.dev`), so `palier.dev` is never empty. It is replaced by the documentation and blog site when that exists.
- **Hosting:** **GitHub Pages**, served from a dedicated repository **`palier-dev/site`** (branch `main`, root). The source stays in this monorepo under `site/`; a workflow (`.github/workflows/sync-site.yml`) copies it to `palier-dev/site` on every change under `site/` on `main`. `site/CNAME` (`palier.dev`) sets the custom domain.
- **Fonts:** **self-hosted** in `site/fonts/` (Chakra Petch, SIL Open Font License, licence shipped as `site/fonts/OFL.txt`). The page makes no requests to third-party font, script or tracking services; everything it loads comes from `palier.dev`.
- **DNS, proxy and analytics:** **Cloudflare** (the registrar, Free plan) manages DNS for `palier.dev` and proxies web traffic to GitHub Pages, which provides its basic analytics without a tracking script.
- **Email obfuscation:** Cloudflare's *Email Address Obfuscation* stays **on**. It rewrites the `mailto:contact@palier.dev` link and adds a small decoding script, served from `palier.dev` itself, to hide the address from scrapers. The script does not track visitors.

## Rationale
- GitHub Pages is free, needs no server, gives HTTPS on a custom domain and sits next to the code in the same organization.
- Self-hosting the font removes the IP transfer to Google, so the page needs no consent banner for fonts. It also makes the page independent of a third-party CDN.
- Keeping DNS at Cloudflare avoids moving the domain. Proxying through Cloudflare gives traffic analytics at the edge, so the page needs no analytics script, and adds caching and HTTPS redirection for free.
- Email obfuscation reduces spam to a public address at no privacy cost, because its script is first-party and collects nothing.

## Consequences
- `site/` in this monorepo is the only place to edit the website. `palier-dev/site` is a build output: anything committed there directly is overwritten by the next sync.
- The sync pushes with an SSH deploy key that has write access to `palier-dev/site` only, stored as the `SITE_DEPLOY_KEY` Actions secret in this repository. Setup steps are at the top of the workflow file.
- A dedicated repository keeps Pages separate from the monorepo (which uses `/docs` for Markdown documentation) and is where the future docs and blog site can be built and published.
- GitHub Pages on a private repository needs a paid GitHub plan; on the Free plan, `palier-dev/site` must be public. This matches the project's public-repo policy.
- `site/.nojekyll` tells Pages to serve the files as they are, without a Jekyll build.
- DNS setup (as configured on 2026-10-24):
  - `palier.dev`: four `A` records (`185.199.108.153` to `185.199.111.153`) and four `AAAA` records (`2606:50c0:8000::153` to `2606:50c0:8003::153`) pointing to GitHub Pages, **proxied** through Cloudflare.
  - `www.palier.dev`: `CNAME` to `palier-dev.github.io`, proxied.
  - Cloudflare SSL/TLS mode **Full**, *Always Use HTTPS* **on**.
  - Mail for `contact@palier.dev` is hosted by iCloud (`MX`, SPF, DKIM, DMARC records). Mail records stay **DNS only**: Cloudflare cannot proxy mail, and a proxied DKIM record breaks signature checks.
- Because traffic is proxied, the page served to visitors is not byte-for-byte the file in `site/`: Cloudflare injects the email-decoding script. Turning *Email Address Obfuscation* off (Scrape Shield settings) restores the exact page.
- GitHub Pages renews its certificate by validating the domain; if a renewal fails behind the proxy, switch the `A`/`AAAA` records to DNS only until the certificate is issued, then proxy again.
- If analytics move beyond Cloudflare's basic, cookieless figures, the consent requirements must be checked against the CNIL guidance on audience measurement before adding anything.
- The future documentation and blog site keeps the same rules: static hosting, self-hosted fonts, no third-party trackers by default.
