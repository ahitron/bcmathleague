# Bergen County Math League — Rebuild

## Project overview

Informational website for the Bergen County Math League (BCML), a competitive math league
serving ~30 high schools in Bergen County, NJ. The site is static, deployed on Netlify, and
must remain free to host.

**Three pages:**
- **Home** — announcements (cards, newest first)
- **Archive** — historical contest PDFs organized by school year
- **About** — participating schools organized by group

**Previous stack (being replaced):** React 16 / Create React App with hardcoded data.

---

## Architecture

| Concern | Choice | Rationale |
|---|---|---|
| Framework | **Astro** (static output) | Fast builds, excellent for content-driven sites, no client JS by default |
| CSS | **Bootstrap 5** (CDN) | Matches existing look, no build step needed for styles |
| CMS | **Decap CMS** | Git-based, free, Netlify-native, browser admin UI at `/admin` |
| Auth | **Netlify Identity** | Free tier, required by Decap CMS |
| Content | **JSON files** in `src/content/` | Simple, version-controlled, editable via CMS or directly |
| PDFs | `public/files/` | Same location as old site; copy directory from old repo |

The CMS admin panel (`/admin`) lets a non-technical user log in with their GitHub-connected
Netlify Identity account, add announcements or update archive data via a form, and save —
which auto-commits to Git and triggers a Netlify redeploy. No terminal required.

---

## File structure

```
/
├── public/
│   ├── files/                  # All contest/solutions PDFs (copy from old repo)
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
│   ├── components/
│   │   ├── Header.astro
│   │   ├── AnnouncementCard.astro
│   │   └── ArchiveYear.astro
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

### `src/content/announcements.json`

Ordered array, newest first. The `date` field is for sorting/display only.

```json
[
  {
    "date": "2025-09-01",
    "title": "Contest Archive",
    "body": "The contest archives have been updated to include all contests through the 2025-2026 school year."
  },
  {
    "date": "2024-09-01",
    "title": "2023-2024 Contests",
    "body": "The contest archives have been updated to include all contests through the 2023-2024 school year."
  },
  {
    "date": "2024-08-01",
    "title": "2024-2025 Advisor's Meeting",
    "body": "The 2024-2025 BCML Advisor's Meeting will be held in September 2024. It will be held virtually on Zoom. Details and link will arrive via email to all registered advisors."
  }
]
```

### `src/content/archive.json`

One entry per school year, reverse chronological order. `contestRounds` and `solutionRounds`
are the number of available rounds for that year (usually 6). The archive page generates PDF
links automatically from these counts using the naming convention below.

```json
[
  { "startYear": 2025, "label": "2025-2026", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2024, "label": "2024-2025", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2023, "label": "2023-2024", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2022, "label": "2022-2023", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2021, "label": "2021-2022", "contestRounds": 5, "solutionRounds": 5 },
  { "startYear": 2020, "label": "2020-2021", "contestRounds": 4, "solutionRounds": 4 },
  { "startYear": 2019, "label": "2019-2020", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2018, "label": "2018-2019", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2017, "label": "2017-2018", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2016, "label": "2016-2017", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2015, "label": "2015-2016", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2014, "label": "2014-2015", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2013, "label": "2013-2014", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2012, "label": "2012-2013", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2011, "label": "2011-2012", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2010, "label": "2010-2011", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2009, "label": "2009-2010", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2008, "label": "2008-2009", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2007, "label": "2007-2008", "contestRounds": 6, "solutionRounds": 6 },
  { "startYear": 2006, "label": "2006-2007", "contestRounds": 6, "solutionRounds": 6 }
]
```

> Note: The 2006-2007 entry is included for completeness but some or all PDFs for that year
> may be missing. Verify against the actual files in `public/files/` and adjust counts if needed.

### `src/content/schools.json`

Rarely changes. Full data:

```json
[
  {
    "group": 1,
    "schools": [
      "Indian Hills High School",
      "Mahwah High School",
      "Midland Park High School",
      "Park Ridge High School",
      "Ramapo High School",
      "Ramsey High School"
    ]
  },
  {
    "group": 2,
    "schools": [
      "Fair Lawn High School",
      "Glen Rock High School",
      "Paramus High School",
      "Ridgewood High School"
    ]
  },
  {
    "group": 3,
    "schools": [
      "Bergenfield High School",
      "Dumont High School",
      "New Milford High School",
      "River Dell High School",
      "Tenafly High School"
    ]
  },
  {
    "group": 4,
    "schools": [
      "Dwight Englewood High School",
      "Fort Lee High School",
      "Ridgefield Memorial High School",
      "Ridgefield Park High School"
    ]
  },
  {
    "group": 5,
    "schools": [
      "Becton Regional High School",
      "Garfield High School",
      "Lodi High School",
      "Rutherford High School"
    ]
  },
  {
    "group": 6,
    "schools": [
      "Cresskill High School",
      "Northern Valley Regional High School (Demarest)",
      "Northern Valley Regional High School (Old Tappan)",
      "Pascack Hills High School",
      "Westwood High School"
    ]
  }
]
```

---

## PDF naming convention

All contest and solutions PDFs follow this pattern:

```
BCML[YY][YY][c|s][N].pdf
```

- `BCML` — constant prefix
- `[YY][YY]` — two-digit year pairs for the school year (e.g., `2526` for 2025-2026)
- `[c|s]` — `c` for contest, `s` for solutions
- `[N]` — round number, 1 through 6

**Examples:**
- `BCML2526c1.pdf` — 2025-2026, Contest round 1
- `BCML2526s3.pdf` — 2025-2026, Solutions round 3
- `BCML0708c6.pdf` — 2007-2008, Contest round 6

**Link construction** (used in archive page):
```js
// given a year entry like { startYear: 2025, label: "2025-2026" }
const yy1 = String(startYear).slice(2);        // "25"
const yy2 = String(startYear + 1).slice(2);    // "26"
const prefix = `BCML${yy1}${yy2}`;             // "BCML2526"
const contestUrl = `/files/${prefix}c${round}.pdf`;
const solutionsUrl = `/files/${prefix}s${round}.pdf`;
```

---

## Decap CMS setup

### `public/admin/index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>BCML Admin</title>
  </head>
  <body>
    <script src="https://unpkg.com/decap-cms@^3.0.0/dist/decap-cms.js"></script>
  </body>
</html>
```

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
              - { name: date, label: Date, widget: date, format: YYYY-MM-DD }
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
              - { name: label, label: Label (e.g. 2025-2026), widget: string }
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

> Important: The JSON files above use a flat array at the root. Decap CMS wraps list widgets
> under a key when using the file collection format. Adjust field names to match exactly, or
> use a wrapper object (`{ "items": [...] }`) in each JSON file and update the Astro imports
> accordingly. Keep it consistent.

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

### Dashboard steps (one-time, done manually)

1. Connect the GitHub repo to Netlify
2. Go to **Site settings → Identity** → Enable
3. Go to **Identity → Services → Git Gateway** → Enable
4. Go to **Identity → Registration** → set to **Invite only**
5. Invite the authorized editor(s) via email from the Identity tab

---

## Bootstrap order

Run these steps in order in a fresh empty directory:

1. **Scaffold Astro:**
   ```bash
   npm create astro@latest . -- --template minimal --no-install --no-git
   npm install
   ```

2. **Create content files** — write the three JSON files above into `src/content/`.

3. **Build Layout and Header** — `src/layouts/Layout.astro` with Bootstrap 5 CDN link in
   `<head>`, navbar with links to Home, Archive, About.

4. **Build the three pages:**
   - `index.astro` — import `announcements.json`, render Bootstrap Cards
   - `archive.astro` — import `archive.json`, render Bootstrap Accordion; generate PDF links
     using the naming convention function above
   - `about.astro` — import `schools.json`, render Bootstrap Accordion by group

5. **Add Decap CMS** — create `public/admin/index.html` and `public/admin/config.yml`.

6. **Add `netlify.toml`** at the root.

7. **Copy PDFs** — copy `public/files/` from the old repo into this one.

8. **Verify locally:**
   ```bash
   npm run dev
   ```
   Check all three pages, confirm archive accordion renders and PDF links are correctly formed.

9. **Push and deploy** — push to GitHub, connect to Netlify, follow dashboard steps above.

---

## Adding a new year's archive (ongoing maintenance)

1. Copy the 12 new PDF files (6 contests + 6 solutions) into `public/files/`, named per the
   convention (e.g., `BCML2627c1.pdf` … `BCML2627s6.pdf`).
2. Add a new entry to `src/content/archive.json` at the top of the array:
   ```json
   { "startYear": 2026, "label": "2026-2027", "contestRounds": 6, "solutionRounds": 6 }
   ```
   Or do both via the `/admin` CMS panel (media upload + archive entry form).
3. Commit and push — Netlify redeploys automatically.

## Adding an announcement

Via `/admin`: open Announcements → add entry at top → Save.
Or directly: prepend an object to `src/content/announcements.json` → commit → push.
