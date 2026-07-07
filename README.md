# Yamei Pets Clinic — HubSpot CMS Theme

A fully bilingual (Traditional Chinese / English) HubSpot CMS theme built for
a pet clinic, developed as the FINAL project for a HubSpot CMS Developer
course. The theme ships with 18 reusable custom modules (including
HubDB-connected modules and repeaters), page templates with drag-and-drop
areas, a global header/footer with a working language switcher, and 6
theme-level settings that are live-editable in the Theme Editor.

> 專案簡介：梅梅寵物診所（Yamei Pets Clinic）的 HubSpot CMS 主題，中文為主語言
> （乾淨網址 `/yamei/<slug>`），英文為語言變體（`/en/yamei/<slug>`）。用來練習
> HubSpot CMS 主題開發：自訂模組、HubDB、Theme Settings、Global Partial 與多語系。

## Features

- **Page templates with drag-and-drop areas**: a standard website page
  template (`home.html`, extends `base.html`, full header/footer + 5 dnd
  areas) and a dedicated landing page template (`landing-page.html`, also
  extends `base.html` so it keeps the global header/footer and language
  switcher, with a drag-and-drop area pre-built from the `campaign-hero`,
  `why-choose-us` and `cta-banner` modules — all bilingual).
- **18 custom modules** covering Fields, Style Fields, a Repeater pattern,
  and HubDB integration (see [Modules](#modules) below).
- **Global header & footer** as global partials, editable once and reflected
  site-wide.
- **6 theme settings**, live in the Theme Editor (see
  [Theme Settings](#theme-settings)).
- **EN / zh-TW multilingual demo** with a working language switcher in the
  header, clean bilingual URLs, and a "fill both languages once, translate
  zero times" content workflow (see
  [Multilingual Architecture](#multilingual-architecture)).

## Repository Structure

```
your-pets-clinic-hubspot-theme/
├── README.md
├── package.json                     # hs CLI convenience scripts
└── yamei-pets-clinic-theme/         # the HubSpot theme folder
    ├── theme.json
    ├── fields.json                  # 6 theme settings
    ├── templates/                   # page templates (incl. book.html, landing-page.html)
    │   └── layouts/base.html        # HubSpot boilerplate layout (unused; landing pages use base.html)
    ├── partials/
    │   ├── global-header.html       # global partial: nav + language switcher
    │   └── global-footer.html       # global partial: footer
    ├── modules/                     # 18 custom modules
    └── css/theme-overrides.css
```

## Setup Instructions

### Prerequisites

- Node.js (for the HubSpot CLI)
- A HubSpot account/portal with CMS Hub access (this project targets a
  training portal, account ID `51203041`)

### 1. Install the HubSpot CLI

```bash
npm install -g @hubspot/cli
```

### 2. Authenticate the CLI against the target portal

```bash
hs init
```

Follow the prompt to paste a Personal Access Key for portal `51203041`. This
creates a local, gitignored auth config — it is **not** committed to this
repo.

### 3. Upload the theme

Run from the **repository root** (one level above the theme folder):

```bash
hs cms upload yamei-pets-clinic-theme yamei-pets-clinic-theme --account=51203041
```

Or via npm script:

```bash
npm run upload
```

### 4. Local development loop

```bash
npm run watch     # auto-uploads on file save
npm run preview   # opens a live local preview
```

### 5. Create a page from a template

In HubSpot: **Marketing → Website → Website Pages → Create → select this
theme → pick `home.html` (website page) or `landing-page.html` (landing
page)** → build with the drag-and-drop areas → **Publish**.

### 6. Publish global content

Header/footer are **global partials** — after editing them (in Design
Manager or via a page's global content editor), you must **Publish** the
global content itself, not just the page, for the change to go live
everywhere.

### 7. Create a language variant (zero-translation workflow)

See [Multilingual Architecture](#multilingual-architecture) — fill in both
the `_zh` and `_en` fields on the **source (Chinese) page** first, then use
**Create variation** to generate the English page. No further translation
is needed; just Publish.

## Multilingual Architecture

- Chinese is the primary language with clean URLs (`/yamei/<slug>`); English
  is a language variant (`/en/yamei/<slug>`).
- Every custom module exposes paired `_zh` / `_en` fields, and templates use
  a shared macro:
  ```jinja
  {% macro t(zh, en) %}{% if html_lang == "zh-tw" and zh %}{{ zh }}{% else %}{{ en }}{% endif %}{% endmacro %}
  ```
- HubDB-backed modules (`service-cards`, `service-listing`, `doctor-grid`,
  `patient-testimonials`) read the `_zh` columns from HubDB (e.g. `name_zh`,
  `description_zh`, `category_zh.label` for select columns) through the same
  `t()` pattern.
- The global header/footer switch language directly via
  `{% if html_lang == "zh-tw" %}`.
- **Golden rule**: fill in both the Chinese and English fields on the source
  (Chinese) page before creating the variation. HubSpot's "Create variation"
  copies the module instances as-is, so the English variant already has its
  `_en` content ready — publish it and you're done, no translation step.
- **Dynamic language switcher**: the switcher lives in the global header
  ([partials/global-header.html](yamei-pets-clinic-theme/partials/global-header.html))
  and links to the *current page's* counterpart in the other language, not a
  fixed URL. It reads HubSpot's built-in `content.translated_content` (each
  linked language variant, exposing `.absoluteUrl` and `.published`) and uses
  `content.absolute_url` for the language you're already on:
  ```jinja
  {% set en_variant = content.translated_content['en'] %}
  {% if html_lang != "zh-tw" %}
    {% set en_href = content.absolute_url %}
  {% elif en_variant and en_variant.published %}
    {% set en_href = en_variant.absoluteUrl %}
  {% else %}
    {% set en_href = "/en/yamei/home" %}   {# fallback if no variant is linked #}
  {% endif %}
  ```
  Each link also carries the `hs-skip-lang-url-rewrite` class so HubSpot serves
  the exact URL instead of applying its own language redirect.
- **Language groups are a prerequisite**: `content.translated_content` is only
  populated when the Chinese and English pages are connected as *language
  variants* in HubSpot. Every page in this project (Home, Services, Our
  Doctors, Contact, Booking, Campaign) is linked as a zh-TW / EN group, which
  is what lets the switcher resolve correctly on every page. Always build the
  English page with **Create variation** from the Chinese source so the group
  link is automatic — a standalone page has no `translated_content`, so the
  switcher would fall back to the homepage.

## Theme Settings

Defined in [fields.json](yamei-pets-clinic-theme/fields.json), editable live
in the Theme Editor (Marketing → Website → Themes → Yamei Pets Clinic Theme
→ Edit theme settings):

| Setting | Type | Used in |
|---|---|---|
| 主色調（品牌色） `primary_color` | color | header, footer links, buttons, carousel, service cards |
| 輔助色 `secondary_color` | color | hover states (header, footer, buttons, carousel) |
| 標題文字色 `heading_color` | color | appointment form, service cards headings |
| Footer 背景色 `footer_bg_color` | color | global footer background |
| Footer 文字色 `footer_text_color` | color | global footer text |
| 按鈕圓角大小 `button_border_radius` | number (0–50px) | header CTA, buttons, carousel, service cards |

## Modules

18 custom modules, covering Fields + Style Fields, a Repeater pattern, and
HubDB integration:

| Module | Repeater | Style Fields | HubDB |
|---|---|---|---|
| appointment-form | | | |
| button | | ✅ | |
| card | ✅ | ✅ | |
| carousel-banner | ✅ | ✅ | |
| campaign-hero | | | |
| contact-info | | ✅ | |
| cta-banner | | ✅ | |
| doctor-grid | | ✅ | ✅ (`yamei_clinicdoctors`) |
| map-embed | | | |
| menu | | ✅ | |
| page-hero | | | |
| patient-testimonials | | ✅ | ✅ (`yamei_clinicreviews`) |
| pricing-card | ✅ | ✅ | |
| service-cards | | ✅ | ✅ (`yamei_clinicservices`) |
| service-listing | | | ✅ (`yamei_clinicservices`) |
| social-follow | ✅ | ✅ | |
| trust-badges | ✅ | ✅ | |
| why-choose-us | ✅ | | |

## Page Templates

- `home.html` — website page template, extends `base.html`, 5 dnd areas
  (hero / trust / services / carousel / cta).
- `landing-page.html` — landing page template, extends `base.html` (keeps the
  global header/footer + language switcher). Its drag-and-drop area ships with
  a bilingual `campaign-hero`, `why-choose-us` (repeater) and `cta-banner`,
  and is fully editable in the page editor.
- `book.html` — dedicated appointment booking page template.
- Plus `services.html`, `contact.html`, `our-doctors.html`, `campaign.html`,
  `about.html`, `pricing.html`, and blog templates.

## Demo Video

[Add the 5–8 minute demo video link here — page creation, module editing,
theme settings change, language switching]

## Author

Built by Ya-Mei for a HubSpot CMS Developer course FINAL project.
