# Leon's Agent Skills

Skills by [Leon van Zyl](https://github.com/leonvanzyl) that take a web app from idea to production in three steps: start it, review it, deploy it. They work with Claude Code, Cursor, Codex, OpenCode, and any other agent supported by the [`skills` CLI](https://github.com/vercel-labs/skills).

## Install

One command installs all four skills:

```bash
npx skills add leonvanzyl/skills
```

Or skip the terminal and paste this straight into your coding agent:

> Please install the start, review, and deploy an app skills by running `npx skills add leonvanzyl/skills`.

You can also browse first with `npx skills add leonvanzyl/skills --list`, or install a single skill with `npx skills add leonvanzyl/skills --skill <skill-name>`.

## The three steps

### 1. Start an app

Always your first step. Describe the app you want to build in plain language. The agent asks clarifying questions until it understands what you actually need, then picks a tech stack that is scalable, easy to deploy, and secure. You end up with a working Next.js app that is yours from the first commit, with sign-in, a database, a landing page, a dashboard, and the legal pages an app of its kind owes its users. The build ends by proving itself with commands that pass or fail.

> Use the start-an-app skill. I want to build...

### 2. Review an app

Once the app exists, have it checked. This skill reviews the whole app as it stands today and reports on three things: security against the OWASP Top 10, SEO and AI discoverability (is your sitemap, robots.txt, and llms.txt still true?), and drift, meaning anything your landing page, docs, or privacy policy claim that the app no longer does. Read-only by default, and it re-checks anything uncertain with a command instead of guessing.

> Use the review-an-app skill on this project.

### 3. Deploy an app

Takes your project and puts it live on services like Vercel. It provisions the database, file storage, and every other prerequisite automatically, sets all environment variables before the first build so there is one deploy instead of a fix-and-redeploy loop, and ends by checking the live site with real commands.

> Use the deploy-an-app skill to take this app live.

## Everything is customizable

Each skill has a `references` folder. Every component of the app and its tech stack lives there as a separate file: auth, database, email, payments, storage, and so on. Each file can be modified.

Want a different auth provider? Ask your coding agent to swap it out in `references/auth.md`. Don't want to deploy to Vercel? Open deploy-an-app's references and swap Vercel for Cloudflare Workers, Hostinger, or whatever you prefer. As long as the service has a CLI the agent can drive, you should be fine.

## Bonus: create a brand kit

Need a logo? The create-brand-kit skill runs a full brand-identity process from any starting point, even just a name. You get a logomark, wordmark, lockups, a design system, icons, social icons, favicons, app icons, and a generated guidelines page. Useful right after starting an app, before you show it to anyone.

> Use the create-brand-kit skill for my app.

## Adding a new skill

Copy [`TEMPLATE.md`](TEMPLATE.md) to `skills/<skill-name>/SKILL.md` (lowercase, hyphens), fill in the frontmatter, write the instructions, and push. The `skills` CLI discovers everything under `skills/` automatically, so there is no registry to update. A skill can also ship supporting files (scripts, references, examples) in the same folder.
