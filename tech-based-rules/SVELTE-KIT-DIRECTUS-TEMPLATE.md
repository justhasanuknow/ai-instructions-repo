# SvelteKit + Directus Site — Build Instructions

This document provides build instructions for an AI agent (Claude Code, Cursor, etc.)
to produce a content-managed website using a hand-written SvelteKit frontend with
Directus as the headless CMS.

The architecture is industry-agnostic: law, consulting, engineering, architecture,
healthcare, restaurants, agencies, portfolios. The interview determines the
project-specific content model; the core model is identical in every project.

---

## Core Principle

The panel never controls layout. Directus stores content only — text, media,
relations, ordering. Every visual decision lives in SvelteKit components written by
hand. The client can change **what** the site says; they cannot change **how** it
looks.

This is the deliberate trade-off against page builders. Do not weaken it by adding
layout, styling, or section-ordering fields to the schema unless the user explicitly
asks for them.

---

## How to Use

1. Create an empty folder and start the agent there.
2. Provide this entire document to the agent along with the prompt below.
3. The agent interviews the user, then builds the project.

### Prompt to Give the Agent

> Build a SvelteKit + Directus website following this document.
>
> Workflow:
>
> **Phase 1 — Interview**: Use the INTERVIEW PROTOCOL section as your script. Ask
> questions one at a time, in the order presented. Validate answers. Confirm
> critical inputs by repeating them back. Save all collected answers to
> `_project-info.md` in the project root so progress survives an interruption.
> Do not start building until the interview is complete.
>
> **Phase 2 — Content model**: From the interview answers, derive the
> project-specific collections and write the full schema plan into
> `_schema-plan.md`. Present it to the user and get explicit approval before
> creating anything in Directus.
>
> **Phase 3 — Build**: Apply the TASK LIST sequentially. Follow PROTECTED PATTERNS
> exactly. Apply every item in SEO REQUIREMENTS. Commit at the end of each major
> step.
>
> **Phase 4 — Validation**: Execute every item in the VALIDATION CHECKLIST. Report
> items you cannot verify yourself and ask the user to confirm them manually.
>
> **Phase 5 — Cleanup**: Delete `_project-info.md`, `_schema-plan.md`, and this
> instruction document from the project folder before final deploy.

### Command Execution

You cannot run `npm`, `node`, `deno`, `docker`, or the Directus CLI yourself
(see `GENERAL-RULES.md`). Whenever a command must be executed, write it out and ask
the user to run it, then wait for their confirmation and output before continuing.
This applies to project scaffolding, dependency installation, schema snapshot and
apply, and all container operations.

---

## Architecture Overview

### Stack

- **SvelteKit** with `adapter-node`, server-side rendering
- **Directus** as headless CMS, one isolated instance per project
- **PostgreSQL** as the Directus database
- **Docker Compose**, deployed through Coolify (Traefik handles domains and TLS)
- **Nodemailer** with a user-supplied SMTP server for form notifications
- No page builder, no component library imposed by this document

### Topology

```text
Directus + PostgreSQL          SvelteKit (adapter-node)
cms.example.com        <-----  example.com
        |                              |
        +--------- Coolify / Docker ---+
```

- All Directus access happens server-side with a static token.
- The token must never reach the client bundle.
- Two environments: `development` and `production`, each with its own Directus
  instance, its own database, and its own token.
- Local development connects to the development Directus over the network.

### Environment Variables

```env
DIRECTUS_URL=https://cms.example.com
DIRECTUS_TOKEN=static_token_here

PUBLIC_SITE_URL=https://example.com

SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASS=
SMTP_FROM=
CONTACT_TO=
```

Commit `.env.example`. Never commit `.env`. Never print a real token into chat, a
committed file, or a log line.

---

## Interview Protocol (Question Script)

Ask each item in the order presented. Mark each as [REQUIRED] or [OPTIONAL]; accept
"skip" for optional items. Re-ask when validation fails.

### Project Basics [REQUIRED — ask first]

- **Project name** — used for slugs, container names, storage key prefix
- **Industry** — determines JSON-LD type and section naming ("Practice Areas" vs.
  "Services" vs. "Projects")
- **Domain** — bare domain, no protocol. Validate format.
- **CMS subdomain** — default `cms.<domain>`
- **Client-facing panel language** — the language of the panel labels the client
  will read

### Languages [REQUIRED — ask early, it changes every collection]

- Ask: "Will the site launch single-language or multilingual?"
- Ask: "Should the multilingual structure be built in even if you launch with one
  language?" Recommend yes — retrofitting translations later means rewriting every
  collection.
- If multilingual or structure-ready: ask for the default language code and any
  additional language codes (BCP-47 style, e.g. `tr-TR`, `en-US`).
- Confirm the URL strategy: default language unprefixed (`/about`), other languages
  prefixed (`/en/about`). Offer prefixing all languages as an alternative.

### Pages [REQUIRED]

- Ask which pages the site needs. For each page, classify it into exactly one
  pattern and confirm the classification with the user:
  - **Pattern A — fixed-field singleton**: one page, fixed design, one field per
    content slot. Home, About, Contact.
  - **Pattern B — list collection**: repeating records rendered by a card
    component, with an optional detail route. Services, Projects, Team, FAQ.
  - **Pattern C — free-form page**: a single rich-text body. Privacy policy, terms,
    legal notices. Handled by the core `pages` collection with no code change.
- If a requested page fits none of these, stop and ask the user how it should
  behave rather than inventing a fourth pattern.

### Home Page Sections [REQUIRED]

- Ask which sections the home page needs, from: hero, intro, highlights, featured
  list, latest posts, gallery, CTA band.
- For each selected section, confirm whether the client should be able to hide it
  (adds a `show_*` boolean) or whether it is always visible.
- Do not add fields for sections the user did not select.

### List Collections [REQUIRED if any Pattern B page exists]

For each list collection, ask:

- **Collection name** and the client-facing label
- **Fields needed** beyond the standard set (title, slug, summary, content, image)
- **Does it need a detail page?** If no, omit the `[slug]` route and the `content`
  field
- **Ordering** — manual (`sort`) or by date

### Blog [OPTIONAL]

- Ask: "Do you want an articles/blog section?"
- If yes, ask whether categories are needed, and whether article pages should show
  an author.
- Comment systems are out of scope. If the user asks for comments, tell them it
  requires a third-party service and ask them to choose one before you proceed.

### Contact and Forms [REQUIRED]

- **Form fields** — default is name, email, phone, subject, message
- **Recipient address** for notifications
- **SMTP details** — host, port, user, sender address. Tell the user to put the
  password in `.env` themselves; never ask them to paste it into the conversation.
- **Map** — ask for latitude and longitude if a map is wanted
- **Spam protection** — honeypot plus a time-based check by default. Ask whether a
  captcha service is also wanted.

### Theming [REQUIRED]

- **Light/dark mode?** Default yes
- **Brand colors** — primary, secondary, optional accent, for each mode. Validate
  contrast against the text color at 4.5:1 (WCAG AA). Warn and re-ask on failure.
- **Typography** — heading font and body font, or ask the agent to propose a pairing
- **Logo assets** — light-mode logo, dark-mode logo, favicon. If the user has only
  one logo, warn that a dark logo disappears on a dark background.

### Compliance and Content Restrictions [REQUIRED]

Ask: "Is this industry subject to advertising, disclosure, or content restrictions
that should be enforced at the schema level?"

Some professions are restricted in ways the schema should reflect. Examples the user
may raise: regulated professions where testimonials, success statistics, fee
information, or comparative claims are prohibited or restricted; sectors with
mandatory disclosure pages; sectors with data-protection notice requirements.

If the user identifies restrictions, record them in `_project-info.md` and omit the
corresponding fields from the schema entirely, so restricted content cannot be
entered from the panel. State the omission explicitly in `_schema-plan.md`. Do not
research or interpret the regulation yourself; take the user's answer as given, and
tell them to confirm the final content with their own compliance source.

### Deployment [REQUIRED]

- **Coolify or plain Docker Compose?**
- **Separate development and production Directus instances?** Recommend yes
- **Who runs the containers** — the agent writes the compose file; the user deploys

---

## Schema Conventions

- Collection and field names: **English, snake_case** (`practice_areas`,
  `cover_image`).
- Panel labels, notes, and option labels: **the client's language**.
- All code, identifiers, and file content: English.

### Standard Fields on Every Content Collection

| Field | Type | Note |
| --- | --- | --- |
| `status` | dropdown | `draft` / `published` / `archived` |
| `sort` | integer | manual drag ordering in the panel |
| `date_created` | timestamp | Directus special field |
| `date_updated` | timestamp | Directus special field |
| `user_created` | uuid | Directus special field |

Every read from the frontend filters `status = published`.

### SEO Fields

Language-specific SEO fields live in the translations table; language-independent
ones stay on the parent.

| Field | Location | Type |
| --- | --- | --- |
| `meta_title` | translations | string |
| `meta_description` | translations | text |
| `og_image` | parent | file |
| `no_index` | parent | boolean |

Frontend fallback chain, applied in this order:

1. `meta_title` → `title` → `site_settings.site_name`
2. `meta_description` → `excerpt` → first 160 characters of `content`, tags stripped
3. `og_image` → `cover_image` → `site_settings.default_og_image`

---

## Multilingual Model

Build this structure whenever the user chose multilingual or structure-ready, even
if only one language is seeded.

### `languages`

| Field | Type | Note |
| --- | --- | --- |
| `code` | string, primary key | `tr-TR`, `en-US` |
| `name` | string | display name |
| `direction` | string | `ltr` / `rtl` |
| `is_default` | boolean | exactly one row is true |

Adding a language later is a new row plus translation entries — never a schema
change. Verify this holds before finishing the schema.

### Split Rule

**Goes into `*_translations`**: `title`, `slug`, `excerpt`, `content`, `meta_title`,
`meta_description`, and every free-text field a visitor reads.

**Stays on the parent table**: `status`, `sort`, timestamps, files, relations,
booleans, colors, icon names, coordinates, `no_index`.

`slug` belongs in the translations table so each language can have its own URL.

### Not Translated

`form_submissions`, `redirects`, `languages`, and any internal or operational
collection.

### Routing

- Default language: no prefix (`/about`)
- Other languages: prefixed (`/en/about`)
- SvelteKit route parameter: `[[lang]]`
- Resolve the language once in `+layout.server.js` and pass it down

Query pattern:

```javascript
readItems('pages', {
  fields: ['*', { translations: ['*'] }],
  deep: {
    translations: {
      _filter: { languages_code: { _eq: lang } }
    }
  },
  filter: { status: { _eq: 'published' } }
});
```

The language switcher needs every translation of the current record to build
alternate URLs, so fetch translations unfiltered on that component's data path.

---

## Core Collections (Identical in Every Project)

### `site_settings` — singleton

| Field | Type | Translated |
| --- | --- | --- |
| `site_name` | string | yes |
| `site_description` | text | yes |
| `logo_light` | file | no |
| `logo_dark` | file | no |
| `favicon` | file | no |
| `default_og_image` | file | no |
| `default_theme` | dropdown (`light` / `dark` / `system`) | no |
| `phone` | string | no |
| `email` | string | no |
| `address` | text | yes |
| `map_lat` | float | no |
| `map_lng` | float | no |
| `working_hours` | text | yes |
| `social_links` | repeater (`platform`, `url`) | no |
| `footer_text` | text | yes |
| `analytics_id` | string | no |

### `navigation`

| Field | Type | Note |
| --- | --- | --- |
| `label` | string | translated |
| `type` | dropdown | `page` / `url` |
| `page` | m2o to `pages` | shown when `type = page` |
| `url` | string | shown when `type = url` |
| `location` | dropdown | `header` / `footer` |
| `parent` | m2o to `navigation` | submenus |
| `sort` | integer | |

Use conditional field visibility so `page` and `url` appear only for the relevant
`type`.

### `pages` — free-form content pages (Pattern C)

| Field | Type | Translated |
| --- | --- | --- |
| `slug` | string, unique per language | yes |
| `title` | string | yes |
| `content` | WYSIWYG | yes |
| `status` | dropdown | no |
| `og_image` | file | no |
| `no_index` | boolean | no |
| SEO fields | see above | yes |

One route renders all of them: `/[[lang]]/[slug]`. The client can add a new legal
or informational page from the panel with no code change. Register this route last
so it does not shadow more specific routes.

### `posts` and `categories` — include only if the user asked for a blog

`posts`: `slug`, `title`, `excerpt`, `content`, `cover_image`, `published_at`,
`category` (m2o), `status`, `no_index`, SEO fields.

`categories`: `slug`, `name`, `description`.

### `form_submissions`

| Field | Type |
| --- | --- |
| `name` | string |
| `email` | string |
| `phone` | string |
| `subject` | string |
| `message` | text |
| `is_read` | boolean |
| `date_created` | timestamp |

Written by the SvelteKit form action using the server token. The public role never
writes here. The client role reads and may update `is_read` only.

### `redirects`

| Field | Type |
| --- | --- |
| `from_path` | string |
| `to_path` | string |
| `status_code` | dropdown (`301` / `302`) |

Resolved in `hooks.server.js` before routing. Always include this collection —
replacing an existing site is the common case and old URLs must keep working.

---

## Project-Specific Layer

Everything derived from the interview goes here. This is the only layer that
changes between projects.

### Pattern A — Fixed-Field Singleton

One collection per page, singleton mode, one field per content slot, grouped in the
panel by section. Example shape for a home page:

| Group | Fields |
| --- | --- |
| Hero | `hero_title`, `hero_subtitle`, `hero_image`, `hero_cta_label`, `hero_cta_url` |
| Intro | `intro_title`, `intro_text`, `intro_image` |
| Highlights | `highlights` repeater (`icon`, `title`, `text`) |
| Featured | `featured_items` (m2m), `show_latest_posts`, `latest_posts_count` |
| CTA | `cta_title`, `cta_text`, `cta_button_label`, `cta_button_url` |

Include only the sections the user selected. Add `show_*` booleans only where the
user asked for hideable sections.

### Pattern B — List Collection

| Field | Type | Translated |
| --- | --- | --- |
| `slug` | string | yes |
| `title` | string | yes |
| `summary` | text | yes |
| `content` | WYSIWYG (omit if no detail page) | yes |
| `icon` | string | no |
| `cover_image` | file | no |
| `status` | dropdown | no |
| `sort` | integer | no |
| SEO fields | see above | yes |

Routes: `/[[lang]]/<collection>` and, when a detail page exists,
`/[[lang]]/<collection>/[slug]`.

### Pattern C — Free-Form Page

Uses the core `pages` collection. No new collection, no new route.

---

## Frontend Structure

```text
src/
├── lib/
│   ├── directus.js           # SDK client, server-only
│   ├── queries/              # one module per collection
│   ├── components/
│   │   ├── layout/           # Header, Footer, Nav, LanguageSwitcher, ThemeToggle
│   │   ├── sections/         # Hero, Intro, Highlights, FeaturedList, Cta
│   │   └── ui/               # Button, Card, Container, Icon
│   ├── seo/                  # meta builder, JSON-LD builders
│   └── utils/                # image url builder, date format, slug helpers
├── routes/
│   ├── +layout.server.js     # site_settings, navigation, languages
│   ├── [[lang]]/
│   │   ├── +page.svelte      # home
│   │   ├── <list-routes>/
│   │   └── [slug]/           # free-form pages, registered last
│   ├── sitemap.xml/+server.js
│   └── robots.txt/+server.js
├── hooks.server.js           # redirects, language resolution
└── app.html                  # theme flash-prevention script
```

Route segments use the site's own language, since they are user-facing. When
additional languages exist, localized path segments come from the translations
table.

### Data Loading Rules

- All Directus calls live in `+page.server.js` or `+layout.server.js`.
- Never import the Directus client into a `.svelte` file.
- `+layout.server.js` loads `site_settings`, `navigation`, and `languages` once and
  shares them through the layout data.
- Request only the fields actually rendered. Do not use `fields: ['*']` on list
  endpoints that carry large relations.
- Handle a Directus outage explicitly: a failed fetch must render an error page,
  not a blank layout.

### Images

Use the Directus transform endpoint:

```text
{DIRECTUS_URL}/assets/{id}?width=800&format=webp&quality=80
```

Write a `getImageUrl(fileId, options)` helper in `lib/utils` and build `srcset` from
it. Always set explicit `width` and `height` attributes to prevent layout shift. Use
`loading="lazy"` for everything below the fold, never for the hero image.

---

## Protected Patterns (Do Not Modify)

### 1. Token Never Reaches the Client

The Directus token is read from private environment variables and used only in
server modules. Never expose it through a `PUBLIC_` variable, never pass it in
returned data, never call Directus from a browser-side `fetch`.

### 2. Theme Flash Prevention

Put a blocking inline script in `app.html` before the closing head tag that sets the
theme attribute before first paint:

```html
<script>
  (function () {
    try {
      var t =
        localStorage.getItem("STORAGE_KEY") ||
        (window.matchMedia &&
        window.matchMedia("(prefers-color-scheme: dark)").matches
          ? "dark"
          : "light");
      document.documentElement.setAttribute("data-theme", t);
    } catch (e) {}
  })();
</script>
```

Replace `STORAGE_KEY` with a project-specific slug. Resolution order at runtime:
stored user choice, then `site_settings.default_theme`, then the system preference.

### 3. Design Tokens Only

All colors, spacing, radii, and shadows come from CSS custom properties defined once
for the light theme and overridden under the dark theme attribute. No hardcoded
color or spacing values in components, except social-brand colors and error states.

### 4. Form Handling

- Use a SvelteKit form action with progressive enhancement, not a client-side POST.
- Validate on the server. Client-side validation is a convenience, never the gate.
- Honeypot field plus a minimum-elapsed-time check against bots.
- Write the record to `form_submissions` first, then send mail.
- If the mail send fails, the record must still be saved and the user must still see
  success. Log the mail failure separately.
- Rate-limit by IP.

### 5. Published Filter

Every public-facing query filters `status = published`. Never rely on the panel to
keep drafts out of the site.

### 6. Schema Snapshot Is the Source of Truth

`schema.yaml` is committed and updated whenever the schema changes. A schema change
that exists only in a running Directus instance does not exist.

---

## SEO Requirements (Mandatory)

- Unique `<title>` and meta description per page, using the fallback chain above.
- Canonical link on every page.
- `hreflang` for every available translation plus `x-default`, when multilingual.
- Open Graph and Twitter Card tags, image 1200x630.
- `sitemap.xml` generated from Directus, covering every language, excluding
  `no_index` and non-published records.
- `robots.txt` with a sitemap reference.
- Semantic HTML: `header`, `nav`, `main`, `section`, `article`, `footer`. Exactly
  one `h1` per page. No skipped heading levels.
- Descriptive `alt` text on every image. Add an `alt` field to file records where
  the text must be editable.
- JSON-LD, matched to the industry from the interview:
  - Home: `Organization` plus the industry type (`LegalService`,
    `ProfessionalService`, `AccountingService`, `MedicalBusiness`, `Restaurant`,
    and so on)
  - Detail pages: `BreadcrumbList`
  - Articles: `Article` or `BlogPosting`
  - FAQ sections: `FAQPage`
- `Cache-Control` headers on server responses; long TTL for static assets.

### Lighthouse Targets

| Category | Target |
| --- | --- |
| Performance (mobile) | 90 or above |
| Accessibility | 95 or above |
| Best Practices | 95 or above |
| SEO | 100 |

---

## Roles and Permissions

**Public role**: read-only where used at all, always filtered to
`status = published`, no access to `form_submissions` or `directus_users`. Since the
frontend uses the static token, public access can stay disabled entirely.

**Client role** (name it `editor`):

| Collection | Permission |
| --- | --- |
| Content collections and `pages` | full create, read, update, delete |
| Singleton page collections | read and update |
| `site_settings` | read and update |
| `navigation` | full create, read, update, delete |
| `form_submissions` | read, and update `is_read` only |
| `redirects`, `languages` | no access |
| Settings, schema, users | no access |

Set the client user's panel language to their own language in their user profile.

`directus schema apply` does not reliably carry roles and permissions. Create the
client role manually once per environment and note this in the handover document.

---

## Schema Management

Directus uses snapshot-based migration, not incremental migration files.

```bash
npx directus schema snapshot ./schema.yaml
npx directus schema diff ./schema.yaml
npx directus schema apply ./schema.yaml
```

Add to `package.json`:

```json
{
  "scripts": {
    "schema:pull": "directus schema snapshot ./schema.yaml",
    "schema:push": "directus schema apply ./schema.yaml"
  }
}
```

Rules:

- `schema.yaml` is committed. Every schema change is a reviewable diff.
- Always run `diff` before `apply` on production. Never use `-y` against production.
- `apply` drops fields removed from the snapshot, **including their data**. Read the
  diff before approving.
- Snapshots carry schema, flows, and panel layouts. They do not carry content, and
  they do not reliably carry roles and permissions.

---

## Docker Compose

```yaml
services:
  database:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_DATABASE}
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  directus:
    image: directus/directus:latest
    volumes:
      - directus_uploads:/directus/uploads
      - directus_extensions:/directus/extensions
    depends_on:
      database:
        condition: service_healthy
    environment:
      SECRET: ${DIRECTUS_SECRET}
      DB_CLIENT: pg
      DB_HOST: database
      DB_PORT: 5432
      DB_DATABASE: ${DB_DATABASE}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD}
      PUBLIC_URL: ${PUBLIC_URL}
      CORS_ENABLED: "true"
      CORS_ORIGIN: ${CORS_ORIGIN}

volumes:
  postgres_data:
  directus_uploads:
  directus_extensions:
```

Tell the user that both `postgres_data` and `directus_uploads` must be backed up. A
database backup alone loses every uploaded image.

---

## Task List (Apply Sequentially)

### Step 1: Interview and record

Run the interview protocol. Write answers to `_project-info.md`.

### Step 2: Content model plan

Derive the collections from the answers. Write `_schema-plan.md` listing every
collection, every field with its type, translation status, and which pattern each
page uses. Name any fields deliberately omitted for compliance reasons. Get explicit
user approval before proceeding.

### Step 3: Directus environment

Write `docker-compose.yml` and `.env.example`. Ask the user to deploy the
development instance and confirm the panel is reachable.

### Step 4: Core schema

Create `languages`, `site_settings`, `navigation`, `pages`, `form_submissions`,
`redirects`, and — if requested — `posts` and `categories`. Ask the user to run
`schema:pull` and commit `schema.yaml`.

### Step 5: Project schema

Create the singleton page collections and list collections from `_schema-plan.md`.
Snapshot again.

### Step 6: Roles

Write the permission matrix for the client role as a checklist the user applies in
the panel. Ask the user to confirm when done.

### Step 7: SvelteKit skeleton

Scaffold the project, install dependencies, wire the Directus client, build
`+layout.server.js`, the header, the footer, the theme system, and the language
plumbing.

### Step 8: Design system

Define the CSS custom properties for both themes from the interview's brand colors
and typography. Build the `ui/` primitives before any page.

### Step 9: Sections and routes

Build the section components, then the routes that compose them. Register the
free-form `[slug]` route last.

### Step 10: Forms

Build the contact form action, the mail sending, and the anti-spam measures.

### Step 11: SEO layer

Meta builder, JSON-LD builders, `sitemap.xml`, `robots.txt`, redirects hook.

### Step 12: Seed and review

Ask the user to enter real content in the panel, then review the rendered site
together.

### Step 13: Production

Deploy the production Directus, apply the schema, recreate the client role, deploy
SvelteKit, configure DNS.

---

## Validation Checklist

### Security

- [ ] `DIRECTUS_TOKEN` does not appear in any client bundle
- [ ] No Directus import in any `.svelte` file
- [ ] `.env` is git-ignored; `.env.example` is committed with no real values
- [ ] Development and production use different tokens
- [ ] Every public query filters `status = published`
- [ ] Form action validates on the server and rate-limits by IP

### Functional

- [ ] Every core and project collection renders on the site
- [ ] Draft records do not appear anywhere on the public site
- [ ] Contact form saves a record and delivers mail
- [ ] Form still saves and still reports success when SMTP fails
- [ ] Honeypot and time check reject an automated submission
- [ ] Language switcher lands on the same page in the other language
- [ ] Redirect entries resolve with the correct status code
- [ ] A 404 renders correctly for an unknown slug
- [ ] The client role can edit content and cannot reach settings or schema

### Visual

- [ ] Theme toggle works and no flash occurs on reload
- [ ] Correct logo variant shows in each theme
- [ ] Layout holds from 320px to 1920px
- [ ] Dark-mode primary color meets 4.5:1 against its text color
- [ ] Images reserve space; no layout shift on load

### SEO

- [ ] Unique title and description per page, fallback chain working
- [ ] Canonical correct on every page
- [ ] `hreflang` bidirectional plus `x-default`, when multilingual
- [ ] `sitemap.xml` excludes drafts and `no_index` records
- [ ] `robots.txt` present and references the sitemap
- [ ] JSON-LD passes the schema.org validator
- [ ] One `h1` per page, no skipped heading levels
- [ ] Every image has descriptive alt text

### Handover

- [ ] `schema.yaml` committed and matching the running instances
- [ ] Client role documented, including that it must be recreated per environment
- [ ] Backup requirement for both volumes stated in writing
- [ ] `_project-info.md`, `_schema-plan.md`, and this document removed from the
      project

---

## FAQ

**The client wants to change the page layout from the panel.** That is outside this
architecture. Explain that layout lives in code and offer to add the specific change
as a new section component. Do not add layout or ordering fields to the schema to
work around it.

**The client wants a page that fits none of the three patterns.** Stop and ask.
Inventing a fourth pattern per project defeats the point of a reusable core.

**A field needs to be removed after launch.** Removing it from the snapshot and
applying will drop the column and its data. Confirm with the user that the data is
disposable, or export it first.

**Roles disappeared after `schema apply` on a new environment.** Expected. Roles and
permissions are not reliably carried by snapshots. Recreate the client role manually
using the documented permission matrix.

**Content changes are not visible on the site.** Check, in order: the record's
`status`, the language filter in the query, the requested `fields` list, and any
response caching.

**Directus is down and the whole site is down.** Expected with `adapter-node` and
live queries. If the user needs the site to survive a CMS outage, the options are a
response cache with stale-while-revalidate, or a build-time static export with a
rebuild webhook. Raise this before launch rather than after.

**Images load slowly.** Confirm the transform parameters are being used rather than
the original asset, that `format=webp` is set, and that `srcset` offers more than
one width.

**The user asks you to run npm, docker, or the Directus CLI.** You cannot. Write the
exact command, ask the user to run it, and wait for their output.

---

## References

- Directus documentation: <https://directus.io/docs>
- Directus SDK: <https://directus.io/docs/guides/connect/sdk>
- SvelteKit documentation: <https://svelte.dev/docs/kit>
- Schema.org validator: <https://validator.schema.org>
- PageSpeed Insights: <https://pagespeed.web.dev>
- WebAIM contrast checker: <https://webaim.org/resources/contrastchecker/>
