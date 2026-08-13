# Leon's Agent Skills

A collection of agent skills by [Leon van Zyl](https://github.com/leonvanzyl). Skills work with Claude Code, Cursor, Codex, OpenCode, and any other agent supported by the [`skills` CLI](https://github.com/vercel-labs/skills).

## Install

Install all skills from this repo:

```bash
npx skills add leonvanzyl/skills
```

Browse what's available first:

```bash
npx skills add leonvanzyl/skills --list
```

Install a specific skill:

```bash
npx skills add leonvanzyl/skills --skill <skill-name>
```

## Skills

| Skill | Description |
| ----- | ----------- |
| [start-an-app](skills/start-an-app) | Interview-driven app scaffolder: asks what you're building in plain language, recommends the right options (SQLite vs Postgres in Docker, sign-in, transactional email, file uploads, payments, AI features, background jobs), and builds a working Next.js app that's yours from the first commit — not a template. Every app ships with account settings and a system page for logs and debugging, plus whatever legal pages an app of its kind actually owes (decided rather than asked, with a cookie banner only where something genuinely tracks people). The build ends by proving itself — commands that pass or fail, then a fresh set of agents checking the result against what you agreed. |
| [deploy-an-app](skills/deploy-an-app) | Takes a Next.js app that runs on your machine and puts it live on Vercel — provisioning the database and file storage, wiring email, background jobs, payments and agent access, and setting every environment variable before the first build so there's one deploy rather than a fix-and-redeploy loop. Asks for the handful of things only a human can do (OAuth callback URLs, DNS records) once, in a single sitting, instead of one failed build at a time. Ends by checking the live site with commands, and says plainly what it couldn't check. |
| [review-an-app](skills/review-an-app) | Reviews an app that already exists — the whole thing as it stands today, not a pull request and not a diff, so it catches what was already wrong before the current branch. Gathers evidence once, then runs three independent lenses over it in parallel: security against the OWASP Top 10 grounded in what this stack actually gets wrong, discoverability (is the sitemap, robots.txt and llms.txt still true?), and drift (does the landing page, the docs and the privacy policy still describe the product?). The bar is the app's own claims rather than what a bigger app would have, so a finding is a contradiction and never an absence. Read-only by default, re-checks anything uncertain with a command, and never concludes that an app is secure. |

## Adding a new skill

1. Copy [`TEMPLATE.md`](TEMPLATE.md) to `skills/<skill-name>/SKILL.md` (folder name should match the skill name: lowercase, hyphens).
2. Fill in the frontmatter (`name`, `description`) and write the instructions.
3. Commit and push — that's it. The `skills` CLI discovers everything under `skills/` automatically; no registry or manifest to update.

A skill can also ship supporting files (scripts, references, examples) alongside its `SKILL.md` in the same folder.
