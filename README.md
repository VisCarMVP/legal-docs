# Viscar Legal Documents

Static HTML pages for Viscar's binding legal documents — Privacy Notice and Beta Tester Agreement — served via GitHub Pages.

**Live URL:** https://viscarmvp.github.io/legal-docs/
(Will be moved to `https://legal.viscar.com/` once the DNS CNAME is configured — see "Custom domain" below.)

## Repository layout

```
legal-docs/
├── CNAME                              # custom-domain pointer (empty until DNS set up)
├── versions.json                      # source of truth for current versions
├── index.html                         # landing page with the list of documents
├── assets/
│   └── legal.css                      # shared stylesheet for all legal pages
├── privacy-notice/
│   ├── v1.0.0.md                      # markdown source
│   └── v1.0.0.html                    # rendered HTML page served to users
└── terms/
    ├── v1.0.0.md                      # markdown source for the Beta Tester Agreement
    └── v1.0.0.html                    # rendered HTML page served to users
```

## `versions.json`

The main Viscar site fetches `https://viscarmvp.github.io/legal-docs/versions.json` on page load, compares the versions against the user's `viscar_accepted_versions` cookie, and shows a re-acceptance modal if any document with `material_change: true` has a newer version than what the user has accepted.

Schema:

```json
{
  "documents": {
    "<doc_type>": {
      "current": "<semver>",
      "url": "<full URL to the HTML page>",
      "material_change": <bool>,
      "effective_at": "<YYYY-MM-DD>"
    }
  },
  "updated_at": "<ISO-8601 timestamp>"
}
```

Currently published document types: `privacy_notice`, `terms`.

## Publishing a new version

1. Edit the existing markdown source (or copy `v1.0.0.md` to `v1.1.0.md` if the changes are substantive).
2. Re-render to HTML (in v1.0.0 this was done by hand — keep both files in sync).
3. Decide whether the change is **material**:
   - **Material** — new categories of data collected, new sub-processors, new processing purposes, new international transfers, new jurisdictions, change to liability terms or governing law, etc.
   - **Cosmetic** — typo fixes, formatting, contact-address change, clarifications without substance.
4. **Compute the SHA-256 of the new HTML file** and write it into the manifest:
   ```bash
   sha256sum privacy-notice/v1.1.0.html
   ```
   This hash is recorded on every user acceptance (in `user_consents.document_hash`)
   so we can later prove which exact bytes the user agreed to. Forgetting it
   leaves the field NULL on every new accept — silent loss of audit value.
5. Update `versions.json`:
   - Bump `current` (e.g. `1.0.1` for cosmetic, `1.1.0` for material additions, `2.0.0` for full rewrites).
   - Update `url` to the new file.
   - Update `hash` to the freshly computed SHA-256 (lower-case hex, 64 chars).
   - Set `material_change: true` only for material updates — this triggers the re-acceptance modal on the main site.
   - Set `effective_at`:
     - **Material updates** — must be at least 14 days after publication date (GDPR notice requirement).
     - **Cosmetic updates** — set to publication date.
   - Update `updated_at` to the current timestamp.
6. Commit and push. GitHub Pages will redeploy within a few minutes.

**Old versions are never deleted.** They stay reachable by direct URL for audit purposes.

## TODOs left in the v1.0.0 documents

The following placeholders are inside both documents and should be filled before public sign-up opens. They are intentionally left visible to avoid silently shipping incorrect text:

| Placeholder | Where | Current value |
|---|---|---|
| `[Polish development company — TBD]` | Section 2 of Terms, Section 6 (sub-processors) of Privacy Notice | placeholder — needs the registered Polish LLC name |
| `legal@viscar.com` | Both | placeholder address — needs to point to a real mailbox |

When the placeholders are filled, publish a new version (`v1.0.1` if it's just the company name plus address, `material_change: false` — this is administrative, not substantive).

## Custom domain (later)

Once a DNS record is added at the registrar for `viscar.com`:

```
Type:  CNAME
Name:  legal
Value: viscarmvp.github.io
```

put the desired hostname (e.g. `legal.viscar.com`) into the `CNAME` file in this repo, push, then enable the custom domain under repository **Settings → Pages**. After GitHub finishes the DNS check, enable "Enforce HTTPS".

When that switch happens, update:

- All `url` fields in `versions.json` to use the new hostname.
- `FALLBACK_VERSIONS` and `VERSIONS_URL` constants on the main site (`/wwwroot/js/legal.js`).
- The hardcoded `https://viscarmvp.github.io/legal-docs/...` links inside the markdown / HTML of v1.0.0 documents — either by republishing as v1.0.1, or accept that v1.0.0 keeps the GitHub Pages URL for historical reference and only newer versions use the custom domain.

## Local preview

```bash
python -m http.server 4000
# then open http://localhost:4000/
```

or any other static file server. No build step.

## Contact

`legal@viscar.com`
