# Bergen County Math League

Informational website for the Bergen County Math League (BCML), serving ~30 high schools in Bergen County, NJ.

**Stack:** Astro · Bootstrap 5 · Decap CMS · Netlify

## Development

```bash
npm install
npm run dev
```

## Content

Content is managed via the CMS at `/admin` (requires Netlify Identity) or by editing files directly:

- `src/content/announcements.json` — home page announcements
- `src/content/archive.json` — contest archive by year
- `src/content/schools.json` — participating schools by group

Contest and solutions PDFs live in `public/files/`, named `BCML[YY][YY][c|s][N].pdf` (e.g. `BCML2526c1.pdf`).

## Deployment

Deployed automatically on Netlify on push to `main`. See `CLAUDE.md` for full setup instructions.
