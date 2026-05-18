# Static Site Build Templates

Collection of build instructions for AI agents (Claude Code, Cursor, etc.) to produce production-ready static websites through an interactive interview flow.

## Quickstart

1. Pick a template that matches your project type (see Available Templates below).
2. Create an empty folder for the new project.
3. Open the folder with Claude Code (or your preferred agent).
4. Attach the template file (drag-and-drop the `.md`) and send the agent the prompt found in the template's "How to Use" section.
5. Answer the agent's interview questions.
6. The agent fetches a reference site, then builds your site, then validates it.

## Available Templates

| Template | Use Case | Reference Site |
|---|---|---|
| [static-site-template.md](static-site-template.md) | Professional service firms (law, consulting, engineering, accounting, healthcare, architecture) | [karabogahukuk.com](https://karabogahukuk.com) |

## What You Get

Every site produced from these templates ships with:

- Pure HTML + CSS + Vanilla JS — no framework, no build step, no `node_modules`
- Dark/light mode toggle with FOUC (flash of unstyled content) prevention
- Fully responsive (320px to 1920px+)
- SEO-complete: meta tags, Open Graph, Twitter Card, schema.org JSON-LD, sitemap.xml, robots.txt
- Accessibility: WCAG AA contrast, ARIA, semantic HTML, focus states
- Lighthouse targets: Performance ≥90, Accessibility ≥95, SEO 100
- Form backend via Web3Forms — no server needed
- Bot protection via hCaptcha + honeypot
- Glass-effect sticky navbar with backdrop blur
- Static hosting compatible: GitHub Pages, Netlify, Vercel, Cloudflare Pages

## Philosophy

- **Templates describe, don't dictate.** Each template tells the agent what to build and which patterns to follow, then points to a live reference site for implementation details. The agent inspects the reference, reproduces the architecture, and adapts content to the new client.
- **Interactive interview, not a form.** The agent asks the user one question at a time, validates each answer, and confirms critical inputs before proceeding to build.
- **Tested patterns are protected.** Each template's "Protected Patterns" section is the accumulated result of real production fixes (Windows Defender phishing detection, FOUC prevention, grid overflow on mobile, Core Web Vitals). Agents must not deviate from these.
- **No vendor lock-in.** Generated sites are pure static files that work on any host. No proprietary runtime, no required CMS.

## Repository Structure

```
/
├── README.md                       # this file
├── static-site-template.md         # professional service firms
└── (future templates)
```

Each template is self-contained — there are no shared imports between templates. You can use any one independently.

## Contributing a New Template

1. Create `your-template-name.md` at the repository root, modeled on existing templates.
2. Identify (or build) a public reference site that demonstrates the architecture.
3. Include the required sections:
   - How to Use
   - Reference Site
   - Interview Protocol
   - Architecture Overview
   - Task List
   - SEO Requirements (mandatory)
   - Protected Patterns
   - Code Conventions
   - Validation Checklist
   - FAQ
   - References
4. Add a row to the "Available Templates" table above.

## License

MIT (or your preferred license — replace this section accordingly).
