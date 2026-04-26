# hugo-module-jsonld

A Hugo module that automatically generates [JSON-LD](https://json-ld.org/) structured data (`schema.org`) for your site, output as a single `@graph` per page.

## Requirements

Hugo `0.123.0` or newer (no extended build required).

## Quick start

**1. Add the module to your `hugo.toml`:**

```toml
[module]
  [[module.imports]]
    path = "github.com/zeced/hugo-module-jsonld"
```

**2. Call the entry point from your `<head>` partial:**

```html
{{ partial "jsonld/head.html" . }}
```

**3. Configure your organisation in `params.toml`:**

```toml
[jsonld.organization]
  name         = "My Organisation"
  url          = "https://example.com/"
  email        = "hello@example.com"
  telephone    = "+1-800-000-0000"
  foundingDate = "2010"
  logo         = "images/logo.png"
  sameAs       = ["https://twitter.com/myorg"]
```

That's it. Every page now gets `Organization`, `WebSite` (home), `BreadcrumbList`, and `WebPage` nodes automatically.

---

## Configuration reference

All settings live under `params.jsonld` in your Hugo config.

```toml
[jsonld]
  # Site-level schemas emitted on every page.
  # Add "Place" only if you have data/jsonld/places.json.
  schemas = ["Organization", "WebSite", "BreadcrumbList", "Place"]

  # Maps Hugo .Type values to JSON-LD schema names.
  # Pages not matched here default to WebPage.
  [jsonld.typeMap]
    blog   = "Article"
    events = "Event"
    team   = "Person"
    shop   = "Product"

  # .Type values where global FAQs (data/jsonld/faqs.json) are injected.
  faqSections = ["services"]

  # Fallback image when a page has no image in its frontmatter.
  defaultImage = "images/og-image.png"

  # Enable SearchAction on the WebSite node (home page only).
  # searchUrlPattern = "/search?q={search_term_string}"

  [jsonld.organization]
    name         = ""   # required; falls back to site.Title
    legalName    = ""
    url          = ""   # falls back to site.BaseURL
    email        = ""
    telephone    = ""
    foundingDate = ""
    logo         = ""   # relative or absolute image path
    sameAs       = []
```

---

## Built-in schemas

| Schema | Auto-detected when | Required frontmatter |
|---|---|---|
| `Organization` | always (site-level) | config only |
| `WebSite` | home page | config only |
| `Place` | always (if `"Place"` in `schemas`) | `data/jsonld/places.json` |
| `BreadcrumbList` | non-home pages | none |
| `WebPage` | all pages (default) | none |
| `CollectionPage` | section pages | none |
| `Article` / `BlogPosting` / `NewsArticle` | via `typeMap` | none |
| `Event` | via `typeMap` | `jsonld.startDate` |
| `FAQPage` | page has `jsonld.faqs` or `.Type` in `faqSections` | `jsonld.faqs` or `data/jsonld/faqs.json` |
| `Person` | via `typeMap` | none |
| `Product` | via `typeMap` | none |

WebPage subtypes (`AboutPage`, `ContactPage`, `ItemPage`, `ProfilePage`) can be forced via `jsonld.type` in frontmatter.

---

## Frontmatter reference

All fields are optional unless noted.

### Any page

```yaml
jsonld:
  type:        "Event"        # override auto-detected schema
  name:        "Custom Title" # falls back to .Title
  description: "…"            # falls back to .Description / .Summary
```

### Event

```yaml
jsonld:
  startDate: "2024-06-15T10:00:00+02:00"   # required — ISO 8601
  endDate:   "2024-06-15T18:00:00+02:00"
  location:                                  # falls back to administrative place
    name: "Venue Name"
    address:
      streetAddress:   "1 Example St"
      addressLocality: "London"
      postalCode:      "EC1A 1BB"
      addressCountry:  "GB"
  organizer:                                 # falls back to configured Organization
    name: "Co-organiser"
    url:  "https://coorganiser.example/"
  offers:
    - name:          "Standard"
      price:         "25"
      priceCurrency: "GBP"
      availability:  "InStock"  # any schema.org ItemAvailability name
      url:           "https://tickets.example/"
```

### Article / BlogPosting / NewsArticle

```yaml
jsonld:
  type: "BlogPosting"   # optional Article subtype
  author:
    name: "Jane Smith"
    url:  "https://example.com/team/jane-smith/"
  datePublished: "2024-03-15T09:00:00+01:00"  # falls back to .Date
  dateModified:  "2024-04-01T12:00:00+01:00"  # falls back to .Lastmod
```

### FAQPage

```yaml
jsonld:
  faqs:
    - question: "How do I get started?"
      answer:   "Follow the Quick Start above."
    - question: "Is it free?"
      answer:   "Yes, MIT licensed."
```

### Product

```yaml
jsonld:
  sku:  "SKU-001"
  gtin: "0614141999996"
  offers:
    - price:         "49.99"
      priceCurrency: "EUR"
      availability:  "InStock"
  aggregateRating:
    ratingValue: "4.8"
    reviewCount: "127"
    bestRating:  "5"
```

### Person

```yaml
# These can be top-level params or nested under jsonld:
jobTitle: "Lead Engineer"
email:    "jane@example.com"
sameAs:
  - "https://github.com/janesmith"
  - "https://linkedin.com/in/janesmith"
```

---

## Data files

### `data/jsonld/places.json`

Required when `"Place"` is in `params.jsonld.schemas`. The entry with `"administrative": true` is used as the default event location and the organisation's `location`.

```json
[
  {
    "slug": "headquarters",
    "name": "My Office",
    "administrative": true,
    "address": {
      "streetAddress":   "1 Example Street",
      "addressLocality": "London",
      "postalCode":      "EC1A 1BB",
      "addressCountry":  "GB"
    },
    "telephone": "+44-20-1234-5678",
    "openingHours": [
      {
        "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
        "opens":  "09:00",
        "closes": "18:00"
      }
    ]
  }
]
```

### `data/jsonld/faqs.json`

Global FAQ pool. Entries with `"active": false` are always skipped.
- `pinned: true` → also appears on pages that have their own `jsonld.faqs`, regardless of `faqSections`
- `pinned: false` → only appears when the page's `.Type` is in `params.jsonld.faqSections`

```json
[
  {
    "question": "Where are you based?",
    "answer":   "London, UK.",
    "active":   true,
    "pinned":   true,
    "category": "general"
  }
]
```

---

## Custom schemas

To add a schema not included in the module:

1. Create `layouts/partials/jsonld/nodes/myschema.html` in your project. The partial receives the page context (`.`) and must `return` a dict (or `dict` to emit nothing).
2. Register it in `typeMap`:
   ```toml
   [jsonld.typeMap]
     recipes = "MySchema"
   ```

Hugo's template lookup order means your project partial automatically overrides or extends the module.

---

## Domain-specific schemas (local override pattern)

For complex schemas tied to your project (e.g. recurring activity schedules with `Service` + `Schedule` nodes), keep the logic in your project and reuse the module's helpers:

```html
{{/* layouts/partials/jsonld/nodes/service-recurring.html */}}
{{- $dateValid := partial "jsonld/helpers/validate-date.html" (dict "date" .Params.startDate "page" .) -}}
{{- $image     := partial "jsonld/helpers/resolve-image.html" (dict "page" .) -}}
{{- $nodeID    := partial "jsonld/helpers/make-id.html"       (dict "base" .Permalink "type" "service") -}}
```

Wire it up by copying `layouts/partials/jsonld/aggregator.html` into your project and adding your section dispatch.

---

## License

MIT — see [LICENSE](LICENSE).
