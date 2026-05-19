# Bergen County Math League

## Project overview

Informational website for the Bergen County Math League (BCML), a competitive math league
serving ~30 high schools in Bergen County, NJ. The site is static, deployed on Netlify, and
must remain free to host.

**Three pages:**
- **Home** — announcements (cards, newest first)
- **Archive** — historical contest PDFs organized by school year
- **About** — participating schools organized by group

---

## Architecture

| Concern | Choice | Rationale |
|---|---|---|
| Framework | **Astro** (static output) | Fast builds, excellent for content-driven sites, no client JS by default |
| CSS | **Bootstrap 5** (CDN) | Matches existing look, no build step needed for styles |
| CMS | **Decap CMS** | Git-based, free, Netlify-native, browser admin UI at `/admin` |
| Auth | **Netlify Identity** | Free tier, required by Decap CMS |
| Content | **JSON files** in `src/content/` | Simple, version-controlled, editable via CMS or directly |
| PDFs | `public/files/` | Served as static assets |

The CMS admin panel (`/admin`) lets a non-technical user log in, add announcements or update
archive data via a form, and save — which auto-commits to Git and triggers a Netlify redeploy.

---

## File structure

```
/
├── public/
│   ├── files/                  # All contest/solutions PDFs
│   ├── favicon.ico
│   └── admin/
│       ├── index.html          # Decap CMS entry point
│       └── config.yml          # Decap CMS collection schema
├── src/
│   ├── content/
│   │   ├── announcements.json
│   │   ├── archive.json
│   │   └── schools.json
│   ├── layouts/
│   │   └── Layout.astro        # Shared HTML shell, Bootstrap CDN, navbar
│   └── pages/
│       ├── index.astro         # Home — announcements
│       ├── archive.astro       # Archive — accordion by year
│       └── about.astro         # About — schools by group
├── astro.config.mjs
├── netlify.toml
└── package.json
```

---

## Content model

All three JSON files use a `{ "items": [...] }` wrapper object at the root. This is required
by Decap CMS file collections. Astro pages import the file and access `.items`.

### `src/content/announcements.json`

Ordered array, newest first. `date` is a string in `YYYY-MM-DD` format, displayed on the site
as "Month D, YYYY".

```json
{
  "items": [
    {
      "date": "2025-09-01",
      "title": "Contest Archive",
      "body": "The contest archives have been updated to include all contests through the 2025-2026 school year."
    }
  ]
}
```

### `src/content/archive.json`

One entry per school year, reverse chronological order. `contestRounds` and `solutionRounds`
are the number of available rounds for that year. The archive page generates PDF links
automatically from these counts using the naming convention below.

Only include years where PDF files actually exist in `public/files/`. Currently covers
2007-2008 through 2025-2026 (no files exist for 2006-2007 or 2008-2009).

```json
{
  "items": [
    { "startYear": 2025, "label": "2025-2026", "contestRounds": 6, "solutionRounds": 6 },
    { "startYear": 2007, "label": "2007-2008", "contestRounds": 6, "solutionRounds": 6 }
  ]
}
```

### `src/content/schools.json`

Rarely changes.

```json
{
  "items": [
    { "group": 1, "schools": ["Indian Hills High School", "..."] }
  ]
}
```

---

## PDF naming convention

```
BCML[YY][YY][c|s][N].pdf
```

- `[YY][YY]` — two-digit year pairs (e.g. `2526` for 2025-2026)
- `[c|s]` — `c` for contest, `s` for solutions
- `[N]` — round number

**Examples:** `BCML2526c1.pdf`, `BCML2526s3.pdf`, `BCML0708c6.pdf`

**Link construction:**
```js
const yy1 = String(startYear).slice(2);
const yy2 = String(startYear + 1).slice(2);
const prefix = `BCML${yy1}${yy2}`;
const contestUrl = `/files/${prefix}c${round}.pdf`;
const solutionsUrl = `/files/${prefix}s${round}.pdf`;
```

---

## Decap CMS

### Known gotchas

- **`date` widget is not available in Decap CMS v3.** Use `widget: string` for date fields.
- **Netlify Identity Widget in Astro requires `is:inline`** on both the CDN script tag and
  any inline script that references `window.netlifyIdentity`, otherwise Astro treats them as
  ES modules (adding `type="module"`) which triggers CORS errors and breaks token handling.
- The widget script must be in `<head>`, not `<body>`, to intercept invite/recovery tokens
  from the URL hash before the page renders.

### `public/admin/config.yml`

```yaml
backend:
  name: git-gateway
  branch: main

media_folder: public/files
public_folder: /files

collections:
  - name: announcements
    label: Announcements
    files:
      - name: announcements
        label: Announcements
        file: src/content/announcements.json
        fields:
          - name: items
            label: Announcements
            widget: list
            fields:
              - { name: date, label: "Date (YYYY-MM-DD)", widget: string }
              - { name: title, label: Title, widget: string }
              - { name: body, label: Body, widget: text }

  - name: archive
    label: Contest Archive
    files:
      - name: archive
        label: Archive Years
        file: src/content/archive.json
        fields:
          - name: items
            label: Years
            widget: list
            fields:
              - { name: startYear, label: Start Year, widget: number }
              - { name: label, label: "Label (e.g. 2025-2026)", widget: string }
              - { name: contestRounds, label: Contest Rounds Available, widget: number, default: 6 }
              - { name: solutionRounds, label: Solution Rounds Available, widget: number, default: 6 }

  - name: schools
    label: Participating Schools
    files:
      - name: schools
        label: Schools by Group
        file: src/content/schools.json
        fields:
          - name: items
            label: Groups
            widget: list
            fields:
              - { name: group, label: Group Number, widget: number }
              - name: schools
                label: Schools
                widget: list
                field: { name: name, label: School Name, widget: string }
```

---

## Netlify configuration

### `netlify.toml`

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Netlify dashboard setup (one-time)

1. Connect the GitHub repo to Netlify
2. **Site configuration → Identity** → Enable
3. **Identity → Services → Git Gateway** → Enable
4. **Identity → Registration** → set to **Invite only**
5. Invite authorized editor(s) via email from the Identity tab

---

## Development

```bash
npm install
npm run dev
```

---

## Adding a new year's archive

1. Copy the new PDFs into `public/files/` (e.g. `BCML2627c1.pdf` … `BCML2627s6.pdf`).
2. Prepend a new entry to the `items` array in `src/content/archive.json`:
   ```json
   { "startYear": 2026, "label": "2026-2027", "contestRounds": 6, "solutionRounds": 6 }
   ```
   Or do both via the `/admin` CMS panel (media upload + archive entry form).
3. Commit and push — Netlify redeploys automatically.

## Adding an announcement

Via `/admin`: open Announcements → add entry at top → Save.
Or directly: prepend an object to the `items` array in `src/content/announcements.json` → commit → push.
