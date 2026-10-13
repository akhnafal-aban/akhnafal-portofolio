# PLAN.md — Astro Portfolio

> Source of truth untuk pembangunan portfolio Astro Noor Akhnafal Aban.
> Update file ini saat scope berubah. Jangan mulai build tanpa re-read ini.

---

## Tujuan

Portfolio personal yang dikelola via `.md` — edit konten = commit markdown, build, deploy. Tanpa server, tanpa database, tanpa CMS. Cocok untuk profil backend-leaning yang ingin showcase tanpa overhead maintenance.

**Live target:** `https://akhnafal-aban.github.io/portfolio-astro/` (atau domain custom nanti)

---

## Stack

| Layer | Pilihan | Alasan |
|---|---|---|
| Framework | **Astro v5** | Zero JS default, content-first, Content Collections bawaan |
| Styling | **Tailwind CSS v4** (via `@astrojs/tailwind` atau manual) | CSS-first `@theme`, modern default 2026 |
| Content | **Content Collections** + `.md` files | Type-safe via Zod schema, build-time validasi |
| UI islands | React 19 (opsional, untuk komponen interaktif) | Pakai hanya kalau perlu (contact form, search) |
| Icons | Lucide atau Phosphor | Satu family, konsisten |
| Fonts | Self-host `@font-face` (Inter + JetBrains Mono) | No Google Fonts `@import` (block render) |
| Deploy | GitHub Pages (legacy branch) atau VPS Nginx | Static `dist/` — no runtime |
| Animasi | CSS-only (transitions, `animation-timeline: view()`) | Keep minimal, respect `prefers-reduced-motion` |

**Tidak dipakai:**
- ❌ Database (SQLite/Postgres/D1)
- ❌ Server functions / API endpoints (kecuali contact form via webhook)
- ❌ Next.js / TanStack Start (overkill untuk portfolio statis)
- ❌ GSAP / Motion library (portfolio recruiter tidak butuh cinematic)
- ❌ Google Fonts `<link>` di `<head>`

---

## Struktur Konten (`.md`)

Semua konten di `src/content/`. Satu file = satu entitas.

```
src/content/
  projects/
    akhnafin.md
    really-sport-center.md
    danantara-research.md
    asset-tracker.md
    jambidan.md
    task-reminder.md
  posts/
    topic-modeling-danantara.md      (blog/writing — opsional v1)
  about.md
  skills.md
  publications.md
  contact.md
```

### Schema (Zod) — `src/content.config.ts`

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const projects = defineCollection({
  loader: glob({ base: './src/content/projects', pattern: '**/*.md' }),
  schema: z.object({
    title: z.string(),
    period: z.string(),
    stack: z.array(z.string()),
    kind: z.string(),                    // "iOS · Personal", "Web · Production", "Research"
    live: z.string().url().optional(),
    repo: z.string().url().optional(),
    order: z.number().default(99),       // urutan tampil
    featured: z.boolean().default(false), // tampil di homepage
  }),
});

const posts = defineCollection({
  loader: glob({ base: './src/content/posts', pattern: '**/*.md' }),
  schema: z.object({
    title: z.string(),
    pubDate: z.coerce.date(),
    description: z.string(),
    tags: z.array(z.string()).optional(),
  }),
});

const pages = defineCollection({
  loader: glob({ base: './src/content', pattern: '*.md' }),
  schema: z.object({
    title: z.string(),
    body: z.string(),
  }),
});

export const collections = { projects, posts, pages };
```

### Contoh Frontmatter — `src/content/projects/akhnafin.md`

```markdown
---
title: "AkhnaFin"
period: "JUL 2026 - Present"
stack: ["Swift", "SwiftUI", "SwiftData", "CloudKit", "Foundation Models", "Vision", "Speech", "App Intents"]
kind: "iOS · Personal"
repo: "https://github.com/akhnafal-aban/AkhnaFin"
order: 1
featured: true
---

# AkhnaFin

Personal finance capture. Five input paths — Siri, natural language, voice, receipt OCR, manual — satu pipeline parse → draft → confirm.

## Teknis
- SwiftData + CloudKit persistence
- Foundation Models untuk natural language transaction parsing
- Vision OCR untuk receipt photo
- Speech untuk voice input
- App Intents untuk Siri shortcut

AI output selalu jadi editable draft sebelum commit. User confirm, baru save. Tidak pernah auto-commit tanpa konfirmasi.
```

### Contoh `really-sport-center.md` (dengan live link)

```markdown
---
title: "Really Sport Center"
period: "AUG 2025 - Present"
stack: ["Laravel", "MySQL", "Docker", "Nginx", "Linux VPS"]
kind: "Web · Production"
live: "https://reallysportcenter.com"
order: 2
featured: true
---

# Really Sport Center

End-to-end gym management platform — members, memberships, payments, check-in/out, dashboards, reporting, scheduled operations. Deployed dan di-maintain di server Linux saya sendiri via SSH.

## Lingkup
- Staging + production environments
- SSH remote admin, monitoring, troubleshooting
- Scheduled operational processes
```

---

## Halaman (Routes)

File-based routing di `src/pages/`:

| File | URL | Sumber konten |
|---|---|---|
| `index.astro` | `/` | Featured projects (dari collection) + bio ringkas + proof strip |
| `work.astro` | `/work` | Semua projects (dari collection, sort by `order`) |
| `work/[slug].astro` | `/work/akhnafin` | Single project (dari collection by slug) |
| `about.astro` | `/about` | `src/content/about.md` |
| `publications.astro` | `/publications` | `src/content/publications.md` |
| `skills.astro` | `/skills` | `src/content/skills.md` |
| `contact.astro` | `/contact` | `src/content/contact.md` + form (webhook) |
| `blog/index.astro` | `/blog` | All posts (dari collection) |
| `blog/[slug].astro` | `/blog/topic-modeling-danantara` | Single post (dari collection) |
| `404.astro` | * | Custom 404 |

### Layout

`src/layouts/Base.astro` — wrapper dengan `<head>`, nav, footer, slot konten. Semua halaman pakai layout ini.

```
<Base>
  <nav> (name, links: Work · About · Publications · Contact)
  <slot />
  <footer> (email, github, linkedin, last-updated)
</Base>
```

---

## Data Pribadi (Source)

Dari `/Users/aban/Profile/akhnaf-context.md` (verified 2026-10-12):

- **Nama:** Noor Akhnafal Aban
- **Headline:** Software Engineer | iOS | Backend Systems
- **Lokasi:** Jakarta, Indonesia (karir); asal Bojonegoro; pendidikan Yogyakarta
- **Email:** akhnafal03@gmail.com
- **GitHub:** github.com/akhnafal-aban
- **LinkedIn:** linkedin.com/in/akhnaf-aban
- **Pendidikan:** Bachelor of Informatics, UII — GPA 3.84/4.00, 2022-2026 (Graduated, Alumni)
- **Publikasi:** "Pemodelan Topik Cuitan tentang Danantara" — Rabit Jurnal Vol 11 No 1, Jan 2026, Garuda-indexed
- **Award:** GEMASTIK 2025 National Round finalist

### 6 Projects (featured di homepage)

1. **AkhnaFin** — iOS, finance capture 5-path pipeline
2. **Really Sport Center** — Laravel, production gym platform (live: reallysportcenter.com)
3. **Danantara Research** — Python BERTopic, published paper
4. **Asset Tracker** — iOS, REST API
5. **Jambidan Village System** — Laravel, village admin + LLM
6. **Task Reminder** — Laravel, Dockerized, project management

### Experience (singkat, di about.md)

- Apple Developer Academy @ UC Jakarta — Junior Developer iOS (FEB 2026 - Present)
- Really Sport Center — Software Engineer freelance (AUG 2025 - Present)
- FTI UII — Backend Developer (MAY 2025 - JAN 2026)
- Artiknesia — Backend Developer Intern (FEB 2025 - MAY 2025)

---

## Design Direction

### Aesthetic

**Dark terminal, restrained.** Bukan AI-default purple. Bukan brutalist. Editorial-meets-terminal.

- Background: `#0a0a0a` atau `#111111`
- Text: off-white `#e5e5e5` body, white headings
- Accent: emerald `#10b981` (satu warna, dipakai sparingly: links, section markers, accent words)
- Surface: `#1a1a1a` untuk card/section berbeda
- Border: `#2a2a2a` hairline
- Mono: `JetBrains Mono` untuk labels, nav, code, data
- Sans: `Inter` untuk body, headings

### Layout

- Single-column, `max-w-3xl` (narrow, editorial, fokus baca)
- Generous whitespace: `py-24` antar section
- Hairline dividers, no cards, no shadows
- Section labels sebagai small uppercase mono di margin kiri (bukan centered eyebrow)
- Projects sebagai numbered entries (01, 02, 03 — OK karena urutan = informasi)

### Motion

- CSS-only: `animation-timeline: view()` untuk fade-in on scroll (0.4s ease-out)
- View Transitions API (Astro built-in) untuk navigasi antar halaman — smooth, no flash
- `prefers-reduced-motion`: disable all, show static

### Yang Dilarang (unslop)

- ❌ Purple/indigo gradient hero
- ❌ "Seamless digital experiences" / "passionate crafter"
- ❌ Emoji bullets
- ❌ Skill rating bars / percentage
- ❌ 3-column equal feature cards
- ❌ Glassmorphism on everything
- ❌ Fake metrics ("10k+ users")
- ❌ Centered hero dengan gradient blob
- ❌ Eyebrow di atas setiap section (max 1 per 3 section)

---

## SEO + GEO (Wajib 2026)

- **JSON-LD** `Person` schema di `<head>` — `name`, `url`, `sameAs: [GitHub, LinkedIn]`
- **JSON-LD** `WebSite` schema dengan `SearchAction` (kalau ada search)
- **`llms.txt`** di root — emerging standard untuk AI discoverability
- **`robots.txt`** dengan directives: `GPTBot`, `ClaudeBot`, `PerplexityBot` (allow)
- **Sitemap.xml** via `@astrojs/sitemap` integration
- **Canonical tags** per halaman
- **Open Graph** + Twitter Card meta per halaman
- **Self-host fonts** — no Google Fonts `<link>`
- **Lighthouse target:** 90+ performance, 100 SEO

---

## Deploy

### Opsi A — GitHub Pages (paling simpel)

```bash
# Build
npm run build

# Push dist ke gh-pages branch
TMP=$(mktemp -d) && cp -R dist/* "$TMP/" && cd "$TMP" && \
  git init -b gh-pages && git add -A && git commit -m "Deploy" && \
  git remote add origin https://github.com/akhnafal-aban/portfolio-astro.git && \
  git push -f origin gh-pages && cd - && rm -rf "$TMP"

# Set Pages source (sekali)
gh api repos/akhnafal-aban/portfolio-astro/pages -X POST \
  -f "source[branch]=gh-pages" -f "source[path]=/" 2>/dev/null || true
gh api repos/akhnafal-aban/portfolio-astro/pages -X PUT \
  -f "build_type=legacy" -f "source[branch]=gh-pages" -f "source[path]=/"
```

URL: `https://akhnafal-aban.github.io/portfolio-astro/`

### Opsi B — VPS Nginx (punya kontrol)

```bash
npm run build
rsync -avz --delete dist/ user@vps:/var/www/portfolio/
```

Astro `astro.config.mjs` perlu `site: 'https://domain-kamu'` + `base` kalau subpath.

### Workflow update konten

```
1. Edit src/content/projects/*.md (atau tambah file baru)
2. git commit -m "update: project X"
3. npm run build
4. Deploy (Opsi A atau B)
5. Live dalam 30 detik
```

---

## Tahap Eksekusi

### Phase 1 — Scaffold (ini yang di-build pertama)
1. `npm create astro@latest portfolio-astro -- --template minimal --typescript strict`
2. `npx astro add tailwind` (Tailwind v4 integration)
3. Setup `src/content.config.ts` (schema Zod)
4. Buat 6 file `.md` di `src/content/projects/` (konten dari profile)
5. Buat `src/content/about.md`, `skills.md`, `publications.md`, `contact.md`
6. `src/layouts/Base.astro` (nav + footer + head)
7. `src/pages/index.astro` (homepage: featured projects + bio + proof)
8. `src/pages/work.astro` + `work/[slug].astro`
9. `src/pages/about.astro`, `publications.astro`, `skills.astro`, `contact.astro`
10. JSON-LD Person schema + sitemap + robots.txt + llms.txt
11. Build + deploy

### Phase 2 — Polish (setelah Phase 1 live)
- View Transitions
- Dark theme refinement
- Mobile audit
- Lighthouse 90+
- Image optimization (kalau ada screenshot proyek)

### Phase 3 — Blog (opsional, kalau mau nulis)
- `src/content/posts/` collection
- `/blog` index + `/blog/[slug]`
- MDX support kalau mau komponen interaktif di post

---

## Status

- [x] Riset stack (Astro vs TanStack vs Next.js)
- [x] Repo dibuat
- [x] PLAN.md ditulis
- [ ] Phase 1: Scaffold + build
- [ ] Phase 2: Polish
- [ ] Phase 3: Blog (opsional)

---

## Catatan

- Profile context verified: `/Users/aban/Profile/akhnaf-context.md` (GPA 3.84, graduated 2026, alumni)
- Project links asli: reallysportcenter.com, admin.reallysportcenter.com, tbe-fit.uii.ac.id
- GitHub Actions billing-locked di akun ini → deploy pakai legacy branch (build lokal → push `gh-pages`)
- Astro versi terbaru: v5 (Node 22+, `astro:content` Content Layer API)
