# caerus_analytics_website

The public marketing site for Caerus Analytics, served at https://caerusanalytics.com.

It is a single static page with no build step, no framework, and no dependencies to install.
Everything the browser needs is in `index.html` except the two web fonts, which load from
Google Fonts at runtime.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The entire site: markup, CSS, and JavaScript in one file. |
| `404.html` | Shown for any unmatched URL, wired up in `staticwebapp.config.json`. |
| `staticwebapp.config.json` | Azure Static Web Apps settings: the 404 rewrite, cache headers, and security headers. |
| `favicon.svg` | Browser tab icon. |
| `apple-touch-icon.png` | Home-screen icon when someone saves the site on iOS. |
| `og-image.png` | The preview card shown when the link is pasted into LinkedIn, WhatsApp, or Slack. |
| `robots.txt`, `sitemap.xml` | Search engine crawling directives. |
| `.github/workflows/azure-static-web-apps-*.yml` | The deploy pipeline. Do not rename or delete this file. |

## How deployment works

Azure Static Web Apps is already connected to this repository. Any push to `main` triggers the
GitHub Actions workflow in `.github/workflows/`, which uploads the repository root to the Azure
resource `ashy-meadow-0a8e97a10`. There is no build step because `output_location` is empty and
the site is plain HTML. A deploy normally finishes in one to two minutes; watch it under the
repository's Actions tab.

Pull requests get their own temporary preview URL, and the preview is torn down when the pull
request closes.

## Publishing a change

```bash
git pull
# edit index.html
git add -A
git commit -m "Describe the change"
git push
```

## Editing common things

**Contact details.** Both the email address and the phone number are defined once, at the top of
the `<script>` block near the bottom of `index.html`:

```js
var CONTACT_EMAIL = "renato@caerusanalytics.com";
var CONTACT_PHONE = "954-554-9293";
```

Changing those two lines updates the contact section, the footer, the mailto link the form builds,
and the clipboard fallback text. The phone number also appears once in human-readable form,
`(954) 554-9293`, in the same script.

**Bilingual copy.** Every translatable element exists twice, once with `lang="en"` and once with
`lang="es"`. CSS hides whichever language is not active, and the EN/ES buttons in the header set
`data-lang` on the root element. To change wording, find the English element and edit its Spanish
twin directly beneath it. Form placeholders, button labels, and the dropdown options are the
exception: those live in the `STRINGS` object in the script, because JavaScript writes them.

**Colors and type.** All colors are CSS custom properties defined in the `:root` block at the top
of the file, with a second set inside the `prefers-color-scheme: dark` media query. Change a value
in both places or the dark theme will drift from the light one.

## Custom domain setup (one time)

DNS for caerusanalytics.com is hosted in Azure DNS, which means Azure can create the records
itself rather than you typing them in.

1. Open the Azure portal and go to the Static Web App resource (`ashy-meadow-0a8e97a10`).
2. Under **Settings**, choose **Custom domains**, then **+ Add**, then **Custom domain on Azure DNS**.
3. Pick the `caerusanalytics.com` zone from the dropdown and select **Add**. Azure writes the
   validation TXT record and the ALIAS record for the apex domain automatically, usually within a
   minute.
4. Repeat for `www.caerusanalytics.com` so both addresses resolve. The www entry uses a CNAME.
5. Set the apex domain as the default so www redirects to it rather than serving a duplicate site.
6. Wait for the **Status** column to read validated. Certificate issuance and DNS propagation can
   take up to 72 hours, though it is usually much faster.

Azure Static Web Apps issues and renews the TLS certificate automatically, so the site will be
reachable over HTTPS with no further work. The apex zone's existing MX records are untouched by
this process, so email at the domain keeps working.

## Things to know before the site is public

- The contact form does not submit anywhere. It composes a message and opens the visitor's own
  email client, which means no server, no database, and no privacy obligations, but also no record
  of an inquiry if the visitor abandons the email. Moving to a real form handler (Azure Functions,
  Formspree, or similar) is the natural next step if volume justifies it.
- The page carries no analytics or tracking of any kind. Adding a privacy-respecting counter such
  as Plausible or Cloudflare Web Analytics would require one script tag in the `<head>` and a
  cookie notice only if the tool sets cookies.
- Both languages ship in the same HTML, with one hidden by CSS. Search engines index both. If
  Spanish-language search traffic becomes important, the better structure is a separate `/es/` page
  with `hreflang` tags pointing at each other.
