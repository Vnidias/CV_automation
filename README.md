# Professional CV — Single Page (Vue + Tailwind + Python)

A single-page, dark-mode résumé/CV that reads from a JSON file, renders with Vue 3 + Tailwind (via CDN), supports tasteful scroll animations, and exports to a PDF with the exact same dark theme. The print layout is opinionated for hiring managers: **Page 1** contains About → Education → Certificates. **Page 2+** contains Experience (timeline). Tech Stack is screen-only by default.

---

## Contents

* [Quick start](#quick-start)
* [Project structure](#project-structure)
* [How it works](#how-it-works)
* [Data schema (`vinicius.json`)](#data-schema-viniciusjson)
* [Customisation](#customisation)
* [Animations](#animations)
* [Exporting to PDF](#exporting-to-pdf)
* [Deployment](#deployment)
* [Troubleshooting](#troubleshooting)
* [Roadmap ideas](#roadmap-ideas)
* [License](#license)

---

## Quick start

1. **Requirements**

* Python 3.9+
* pip
* A modern browser (Chrome/Edge/Firefox/Safari)

2. **Install & run**

```bash
# from the project root
pip install -r requirements.txt

# run the tiny server
python app.py
```

Visit: [http://127.0.0.1:5000/](http://127.0.0.1:5000/)

3. **Provide assets**

* Keep `vinicius.json` in the project root.
* Put your avatar at `profile.jpg` (or set `"photo"` in the JSON).

---

## Project structure

```
.
├─ app.py                 # Minimal Flask server (serves static files)
├─ index.html             # App (Vue + Tailwind + animations + print CSS)
├─ vinicius.json          # Your CV data (see schema below)
├─ profile.jpg            # Avatar (round, 20×20 Tailwind units on screen)
└─ requirements.txt       # Flask
```

**Why Flask?**
Browsers block `fetch()` from `file://`. A tiny server avoids CORS/permissions problems and mirrors production.

---

## How it works

* **Vue 3 (global build)** powers data binding. It `fetch()`es `vinicius.json` on mount and renders the sections.
* **Tailwind (CDN)** gives utility classes. Dark mode is **forced** (`<html class="dark">`).
* **Custom `v-animate` directive** (IntersectionObserver) adds subtle, accessible scroll animations with optional staggering. Animations replay when items leave and re-enter the viewport; they’re disabled for print and for users who prefer reduced motion.
* **Print** uses `@page` and an inner `.sheet` margin to guarantee the order:

  * **Page 1**: About → Education → Certificates
  * **Page 2+**: Experience (timeline)
  * Tech Stack is hidden in print via `.print-hide` (easy to change).

---

## Data schema (`vinicius.json`)

Use these keys. Unknown keys are ignored.

```json
{
  "name": "Vinicius Dias Barros",
  "title": "Data Architect Consultant",
  "photo": "profile.jpg",
  "linkedin": "https://www.linkedin.com/in/your-handle",
  "about": "<p>Short HTML-supported summary. Keep it concise.</p>",
  "education": [
    { "degree": "B.Sc. in Computer Science", "institution": "University X", "years": "2012 – 2016" }
  ],
  "certificates": [
    { "name": "Databricks Certified Data Engineer Professional", "year": "2024" }
  ],
  "tech_stack": {
    "Programming Languages": [
      { "name": "Python", "years": "8 yrs" },
      { "name": "SQL", "years": "8 yrs" }
    ],
    "Cloud Technologies": [
      { "name": "Azure", "years": "6 yrs" }
    ],
    "Databases": [
      { "name": "SQL Server", "years": "8 yrs" }
    ]
  },
  "experience": [
    {
      "title": "Data Architect",
      "company": "WAES",
      "location": "Eindhoven, the Netherlands",
      "period": "May 2025 – Present",
      "technologies": ["Databricks","Python","SQL","Airflow","DBT","Docker","Apache Spark","Terraform"],
      "highlights": [
        "Segmented and tailored data & AI solution materials for sales and pre-sales teams.",
        "Built targeted demos on governance, lakehouse architecture, and cloud migration."
      ]
    }
  ]
}
```

**Notes**

* `about` accepts minimal HTML (e.g., `<p>…</p>`, `<strong>`).
* `photo` can be a local path (`profile.jpg`) or a full URL.
* `years` values are strings for display (“8 yrs”) to avoid locale issues.

---

## Customisation

### Dark/light mode

Dark mode is pinned by `<html class="dark">`. To switch to system preference:

```html
<html lang="en" class="scroll-smooth">
```

…and remove dark-only palette overrides if desired.

### Section order for print

The print order is determined by **two `.page` sections** in `index.html`:

```html
<section class="page">  <!-- Page 1 -->
  <!-- About / Education / Certificates (+ Tech Stack if you want) -->
</section>

<section class="page">  <!-- Page 2+ -->
  <!-- Experience timeline -->
</section>
```

To include **Tech Stack** in print, remove the `.print-hide` wrapper around that block.

### Margins for print

By default:

```css
@page { size: A4; margin: 0; }
:root { --sheet-margin: 1.5cm; } /* inner margin */
```

Adjust `--sheet-margin` to 1.2 cm for a denser layout.

### Timeline dot alignment

* The line is `border-s` on the `<ol>`.
* Each `<li>` has `ps-6` (padding-start) to create the gutter.
* The dot:

```html
<span class="absolute left-0 -translate-x-1/2 top-2 h-3 w-3 rounded-full bg-indigo-500"></span>
```

This centers the dot exactly over the line on all entries.

---

## Animations

A lightweight directive powers AOS-style animations with replay and staggering.

**Use it**

* Animate one element:

```html
<div v-animate data-animate="fade-up" data-delay="120">...</div>
```

* Stagger children:

```html
<div v-animate.group data-animate="fade-up" data-stagger="60">
  <div data-animate-child>Card A</div>
  <div data-animate-child>Card B</div>
</div>
```

**Effects**

* `fade-up` (default), `fade-left`, `fade-right`, `zoom-in`

**Modifiers**

* `.group` – animate `[data-animate-child]` descendants with optional `data-stagger`
* `.once` – reveal once; do not hide on leave

**Accessibility**

* Respects `prefers-reduced-motion: reduce`.
* Disabled during print.

---

## Exporting to PDF

1. Click **Export PDF** (top-right).
2. In the browser print dialog:

   * **Destination**: Save as PDF
   * **Pages**: All
   * **More settings → Background graphics**: **On** (crucial for dark theme)
   * **Paper size**: A4
   * **Margins**: Default (the inner sheet margin is handled by CSS)
3. Page 1 includes About → Education → Certificates.
   Page 2+ includes Experience.

**Why it matches the screen**
We set `print-color-adjust: exact` and avoid removing dark backgrounds in print.

---

## Deployment

This is a static app. Any static host works.

* **GitHub Pages / Netlify / Vercel**: Serve `index.html`, `vinicius.json`, and `profile.jpg`.
  If your host forbids `fetch` of sibling files, place assets in `/public` or configure static headers.
* **Docker / VM**: Any web server (nginx, Caddy) can serve the files in root.

For quick demos on a server:

```bash
# Python's built-in server (no Flask) if your host allows local fetches:
python -m http.server 8080
```

Some browsers disallow `fetch()` from `file://`. When testing locally, use `Flask` (`python app.py`) or any static server.

---

## Troubleshooting

**Blank sections after load**
Cause: elements added after the directive mounted were not observed.
Fix: the project includes a `MutationObserver` in the `v-animate` directive to register new children; ensure you’re using the provided `index.html` script.

**Content hidden after scrolling**
By design, animations **replay** on enter/leave. If you want a section to stay visible, use `v-animate.once` on that element.

**Timeline dot misaligned**
Ensure each experience `<li>` uses `relative ps-6`, the `<ol>` has `border-s`, and the dot span uses `left-0 -translate-x-1/2 top-2`.

**Avatar not showing**
Set `"photo": "profile.jpg"` in JSON or replace `profile.jpg` at root. Check path and case sensitivity.

**PDF looks light instead of dark**
Enable **Background graphics** in the print dialog.

**`fetch` error on local file**
Run `python app.py` instead of opening `index.html` directly from the filesystem.

---

## Roadmap ideas

* Shareable view state via URL query (e.g., `?page=experience&fx=fade-up`).
* ATS print theme toggle (light, monochrome, 11 pt, borderless).
* JSON validator + live editor for the profile.
* CSV/JSON export of the skills matrix.

---

## License

MIT. Do what you want; attribution appreciated.

---

**Q1:** Should Tech Stack also print on Page 1 when it fits (auto-spill to Page 2 otherwise)?
**Q2:** Do you want a compact print mode (smaller avatar, tighter paddings) to guarantee a strict one-page About/Education/Certificates?
**Q3:** Add an optional email/phone/location to the JSON and render matching icons in Contact?
