# Anderson Mill Gun Club — Website

Community firearms club website for northwest Austin, TX. Built with Hugo Extended, hosted on GitHub Pages, deployed automatically via GitHub Actions.

**Live site:** https://andersonmillgunclub.org
**Repo:** https://github.com/caseymhunt/anderson-mill-gun-club

---

## Stack

| Layer | Tool |
|---|---|
| Static site generator | Hugo Extended v0.162.1 |
| Theme | Custom (`themes/amgc`) |
| Hosting | GitHub Pages |
| CI/CD | GitHub Actions (`.github/workflows/deploy.yml`) |
| Newsletter | Buttondown (`buttondown.com/amgc`) |
| Contact form | Formspree (form ID `xlgvyjrk`) |
| Waiver signing | Google Forms (`https://forms.gle/2hcdPLZFwVnsbzPR8`) |
| Domain registrar | IONOS |
| Club email | `andersonmillgunclub@gmail.com` |

---

## Local Development

Hugo is installed via WinGet and is **not in the system PATH for Bash**. Use the full path or PowerShell:

```powershell
& "C:\Users\Owner\AppData\Local\Microsoft\WinGet\Packages\Hugo.Hugo.Extended_Microsoft.Winget.Source_8wekyb3d8bbwe\hugo.exe" server -D --source "C:\Users\Owner\anderson-mill-gun-club"
```

Then open http://localhost:1313/ in your browser. The dev server live-reloads on file save and automatically overrides `baseURL`, so all links resolve correctly at localhost.

The `-D` flag includes draft pages (e.g. the waiver, which is hidden until attorney review).

---

## Deployment

Push to `main` → GitHub Actions builds the site with Hugo Extended and deploys to GitHub Pages automatically. Build typically completes in under 2 minutes.

**Do not commit the `public/` directory** — it is the local build output and is ignored by the CI build. GitHub Actions generates its own clean build.

---

## Project Structure

```
content/          # Page content (Markdown)
static/           # Copied as-is to the build output (favicon, etc.)
docs/             # Non-published documents (waiver draft, etc.)
themes/amgc/
  layouts/        # HTML templates
    _default/
      baseof.html         # Base HTML shell (head, nav, footer)
      single.html         # Default interior page layout
      newsletter.html     # Custom layout for /newsletter/
    index.html            # Homepage template
    partials/
      nav.html
      footer.html
      newsletter-form.html  # Buttondown embed form
  assets/css/
    main.css              # All site styles (single file, minified at build)
hugo.yaml           # Site config, menus, params
```

---

## Pages

| URL | Content file | Notes |
|---|---|---|
| `/` | `content/_index.md` | Homepage |
| `/about/` | `content/about.md` | |
| `/get-involved/` | `content/get-involved.md` | Email hardcoded (see Quirks) |
| `/safety/` | `content/safety.md` | |
| `/newsletter/` | `content/newsletter.md` | Custom layout injects Buttondown form |
| `/contact/` | `content/contact.md` | Formspree form |
| `/subscribed/` | `content/subscribed.md` | Post-newsletter-signup thank-you |
| `/privacy/` | `content/privacy.md` | |
| `/waiver/` | `content/waiver.md` | **draft: true** — hidden until attorney review |

---

## Services & Configuration

### Buttondown (newsletter)
- Account: `buttondown.com/amgc` — log in with `andersonmillgunclub@gmail.com`
- **The username `amgc` is hardcoded** in `themes/amgc/layouts/partials/newsletter-form.html`, not in `hugo.yaml`. Check this file first when debugging newsletter issues.
- Form posts to `https://buttondown.com/api/emails/embed-subscribe/amgc` (embed API, no CSRF issues)
- Double opt-in is **enabled** — this is intentional and must not be disabled
- Post-subscription redirects are configured in Buttondown Settings → Subscribing:
  - "Redirect unconfirmed subscribers" → `https://andersonmillgunclub.org/subscribed/`
  - "Redirect confirmed subscribers" → `https://andersonmillgunclub.org/subscribed/`
- **If the domain ever changes**, these two Buttondown URLs must be updated in the settings panel in addition to `baseURL` in `hugo.yaml`

### Formspree (contact form)
- Form ID `xlgvyjrk` is set in `content/contact.md`
- Free tier; submits are emailed to the club Gmail

### Google Forms (participant waiver)
- Form URL: `https://forms.gle/2hcdPLZFwVnsbzPR8`
- Responses stored in Google Sheets under `andersonmillgunclub@gmail.com`
- Linked from `content/waiver.md` — currently hidden (`draft: true`) pending attorney review
- To publish the waiver page: remove `draft: true` from the front matter and push

### DNS (IONOS)
- Apex domain: four A records pointing to GitHub Pages IPs (185.199.108–111.153)
- `www` CNAME → `caseymhunt.github.io`
- HTTPS enforced via GitHub Pages (Let's Encrypt, auto-renews)

---

## Design System

**Palette**

| Token | Hex | Usage |
|---|---|---|
| `--bark` | `#2C1F0E` | Nav background, headings |
| `--earth` | `#3D2B1F` | Footer background |
| `--rust` | `#8B3A1A` | Primary CTA buttons, accents |
| `--gold` | `#C8873A` | Secondary accents, hover states |
| `--tan` | `#C4A882` | Nav links, subtle text |
| `--cream` | `#FDF8F3` | Page background |
| `--peach` | `#F0D5B0` | Cards, borders |

**Typography**
- Headings: Lora (serif)
- Labels / buttons / nav: Oswald (condensed sans)
- Body: Nunito (sans-serif)

All three loaded from Google Fonts in `baseof.html`.

---

## Known Quirks & Lessons Learned

**Hugo not in Bash PATH**
Hugo is installed via WinGet, which adds it to the Windows PATH but not the Bash PATH. Always use the full `.exe` path in Bash, or use PowerShell.

**URL handling under GitHub Pages subpath**
The old GitHub Pages URL had a subpath (`/anderson-mill-gun-club/`). All internal links use Hugo URL functions (`relURL`, `absURL`, `site.Home.RelPermalink`) rather than root-relative paths. This is still the correct pattern even on the custom domain — don't hardcode `/` prefixed paths in templates.

**Template syntax does not render in Markdown**
Hugo template tags (`{{ }}`) in `.md` content files are output as literal text. Workarounds used in this project:
- Email address in `get-involved.md` is hardcoded as plain text
- Newsletter form is injected into the newsletter layout via `replaceRE` on a `<!-- newsletter-form -->` comment placeholder (see `themes/amgc/layouts/_default/newsletter.html`)

**Raw HTML in Markdown**
Enabled via `markup.goldmark.renderer.unsafe: true` in `hugo.yaml`. Required for inline HTML (buttons, divs, forms) in content files.

**CSS specificity: buttons inside page body**
The rule `.page-body-inner a { color: var(--rust); }` overrides button text color. Any `.btn-rust` or `.btn-gold` inside `.page-body-inner` needs an explicit `color: #fff` override. This is already in `main.css` — be aware if adding new button styles.

**Buttondown embed API vs. web UI**
The form must post to `https://buttondown.com/api/emails/embed-subscribe/amgc`. Posting to `https://buttondown.com/amgc` (the web UI URL) returns a 403 CSRF error.

**Taxonomies disabled**
`disableKinds: [taxonomy, term]` in `hugo.yaml` suppresses Hugo's default tag/category pages. Do not add tags or categories to content front matter — they will silently generate broken pages.

**`public/` directory**
The `public/` folder is local build output and should not be committed. It is generated fresh by GitHub Actions on every deploy.

---

## Pending Before Launch

- [ ] **Participant waiver** — attorney review required before publishing. Once approved: remove `draft: true` from `content/waiver.md`, add footer link back in `themes/amgc/layouts/partials/footer.html`, and push.
- [ ] **Hero image** — current image is a Pexels stock photo (Tima Miroshnichenko, photo ID 6090798). Replace with original photography by updating `params.heroImage` in `hugo.yaml`.
- [ ] **Custom email** — optional: set up Zoho Mail for an `@andersonmillgunclub.org` address.

---

## Legal

The participant liability waiver draft is at `docs/waiver-draft.md`. It has not been reviewed by a licensed Texas attorney and **must not be used at any live event** until that review is complete.
