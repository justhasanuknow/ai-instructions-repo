# AI Instructions

A curated collection of instruction documents for AI agents (Claude Code, Cursor, GitHub Copilot, etc.) covering recurring engineering and creative tasks.

Each instruction file is a self-contained document that an agent can follow to complete a specific task class — from building a static website, to reviewing a pull request, to refactoring legacy code, to producing design briefs.

## How It Works

Each instruction file in this repo is designed to be:

1. **Standalone** — pasted or attached to an agent without external dependencies
2. **Interactive when needed** — the agent collects required information from the user via interview before acting
3. **Reference-anchored** — points to a live reference (URL, repo, file) for grounding rather than describing everything inline
4. **Reproducible** — produces consistent, high-quality output across runs and across agents

## Quickstart

1. Browse the catalog below and find an instruction matching your task.
2. Open a working directory and start your preferred agent (Claude Code, Cursor, etc.).
3. Attach or paste the instruction file.
4. Follow the "How to Use" section inside the file (each instruction has its own tailored prompt).

## Catalog

| Instruction | Category | Use Case | Status |
|---|---|---|---|
| [static-site-template.md](static-site-template.md) | Web Development | Build a modern professional-service static website with dark/light mode, SEO, form, and bot protection | Stable |

Planned categories: code review, refactor, design brief, content writing, data analysis, devops runbooks.

## Anatomy of an Instruction

A well-formed instruction file in this repo typically contains:

- **Overview** — what the instruction is for, in one paragraph
- **How to Use** — concrete steps for the user + the prompt to give the agent
- **Reference** — URL or file the agent should study before acting
- **Interview Protocol** — questions to ask the user, in order, with validation rules
- **Task List** — sequential steps the agent executes
- **Requirements** — non-negotiable rules (e.g., SEO, accessibility, security)
- **Protected Patterns** — tested behaviors that must not be modified
- **Conventions** — naming, structure, style guidelines
- **Validation Checklist** — items to verify before reporting success
- **FAQ** — known edge cases and resolutions

Not every instruction needs every section, but this is the standard skeleton.

## Philosophy

- **Instructions describe, don't dictate.** Each file tells the agent what to produce and which patterns to follow, then points to a reference for implementation details. Agents inspect the reference, internalize patterns, and adapt to the specific task.
- **Interactive interview over forms.** When inputs are needed, the agent asks the user one question at a time, validates each answer, and confirms critical inputs before proceeding.
- **Tested patterns are protected.** The "Protected Patterns" section in each instruction is the accumulated result of real production fixes. Agents must not deviate.
- **No vendor lock-in.** Instructions are model-agnostic. They should work with any capable agent (Claude, GPT, Gemini, etc.). Test before committing to a specific agent.

## Repository Structure

```
/
├── README.md                       # this file
├── static-site-template.md         # web: professional-service site
└── (future instructions)
```

Each file is self-contained — no shared imports. You can use any instruction independently.

## Contributing a New Instruction

1. Pick a recurring task you'd like to standardize.
2. Identify a public reference (live site, GitHub repo, documentation page) that demonstrates the desired output quality.
3. Create `your-instruction-name.md` at the repository root, modeled on existing instructions.
4. Include the relevant sections from the "Anatomy" list. Skip what doesn't apply.
5. Test the instruction with a fresh agent at least once, end-to-end.
6. Add a row to the Catalog table above.

## License

No license. All rights reserved.

This repository is provided as-is, for personal/internal use by the author. Public use, redistribution, or derivative works are not permitted without explicit written permission.
