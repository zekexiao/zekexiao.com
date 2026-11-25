# zekexiao.com

Personal homepage + blog built with Astro 7 (static output), deployed to GitHub Pages.

## Commands

```bash
npm run dev      # local dev server (http://localhost:4321)
npm run build    # static build to dist/
npm run preview  # preview the production build
```

## Where to edit

- **Personal info** (name, role, bio, email, links): `src/site.config.ts` — look for `TODO` comments.
- **Blog posts**: `src/content/blog/*.md` — frontmatter schema is defined in `src/content.config.ts` (`title`, `description`, `pubDate`, `updatedDate?`, `tags[]`, `draft`). Posts with `draft: true` are excluded from production builds.
- **Global styles / design tokens**: `src/styles/global.css` — strict Bauhaus black/white palette (`--black`, `--white`, `--steel` for secondary text, `--rule` for borders). No other colors, no dark mode. Fonts: IBM Plex Sans Variable (body, never heavier than weight 500; large display uses 300) + IBM Plex Mono (labels/meta).
- **Layout & chrome**: `src/layouts/BaseLayout.astro`, `src/components/Header.astro`, `src/components/Footer.astro`.
- **Images / files**: `public/uploads/` (migrated from WordPress `wp-content/uploads`; reference as `/uploads/...`).
- **Legacy URLs**: `src/pages/[year]/[slug].astro` emits static redirect pages from the old WordPress permalink format `/%year%/%postname%/` to `/blog/<slug>/`.
- `.wp-export/` (gitignored): raw dump + converter used for the WordPress migration.

## UI conventions

Follow Vercel's [Web Interface Guidelines](https://github.com/vercel-labs/web-interface-guidelines) when touching UI code: semantic HTML + `aria-label`/`aria-current`/`aria-hidden` where needed, `:focus-visible` styles, animate `transform`/`opacity` only under `prefers-reduced-motion: no-preference`, `touch-action: manipulation` on interactive elements, explicit `width`/`height` on `<img>`, prefer muted `<video>` over animated GIFs, `Intl.*` for dates/numbers, `translate="no"` on brand names and code.

## Deploy

- Push to `main`; `.github/workflows/deploy.yml` builds with `withastro/action` and deploys to GitHub Pages.
- One-time setup: repo **Settings → Pages → Source: GitHub Actions**.
- Custom domain `zekexiao.com` is served via `public/CNAME` (also set the domain in Settings → Pages and point DNS at GitHub Pages).
