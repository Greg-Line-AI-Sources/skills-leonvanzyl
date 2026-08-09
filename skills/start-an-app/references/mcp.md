# Agent access (MCP)

Last verified: 2026-08-09

**Purpose:** Let AI agents — Claude, Claude Code, ChatGPT, Cursor, anything that speaks MCP — do the app's real work on behalf of a signed-in user. The app gains a second front door: the same actions, the same ownership rules, the same log, reached over an authenticated protocol instead of a browser.

> **Hard rule: every tool takes the user from the token and scopes every query to them. Never trust an id the model passed you.** A tool call arrives carrying the user's full authority, but the thing that *triggered* it may be text the user never wrote — a web page the agent read, an email it summarised, another tool's output. In the browser a person can only click what they already own; a tool argument is a string somebody else may have chosen. `logged()` below exists to make this mechanical rather than something to remember, and every tool goes through it.
>
> If Better Auth's or Anthropic's documentation and this file disagree on *how to structure this*, this file wins. If they disagree on a method name, an option, or an endpoint path, their docs win — update this file afterwards, because both of these move quickly.

**Prerequisite: sign-in must exist.** Tools act as somebody, and the whole authorisation flow hangs off the auth config, so there is nowhere to put this otherwise. If the user asked for agent access but said no to accounts, set up `references/auth.md` first and explain it in a sentence ("an agent has to sign in as *you*, so the app knows whose data it's touching") rather than treating it as a blocker.

> **Almost every example online is wrong, in four specific ways.** This area churned hard and the search results have not caught up — Step 2's research is not optional here. (1) Better Auth's own `mcp()` plugin is **deprecated** in favour of the OAuth provider plugin used below; do not reach for it because a tutorial does. (2) `@vercel/mcp-adapter` is dead — it was renamed `mcp-handler`, and the rename came with breaking changes: `server.tool()` became `registerTool`, `inputSchema` takes a whole schema object rather than a bare shape, and `basePath`, `maxDuration` and `redisUrl` are gone. (3) Better Auth's *own* MCP documentation page is still written against the dead adapter and will not compile. (4) Anything telling you to provision Redis is describing the old adapter. If something here doesn't compile, check the current docs — never a blog post.

## What you're building

Three parts, and it helps to say them out loud to the user in this order:

1. **An authorisation server.** Already there — it's Better Auth. The plugin adds the OAuth endpoints an agent needs to ask permission.
2. **A resource server.** One route, `/api/mcp`, that checks the token and runs the tool.
3. **The tools.** The app's own verbs, the same ones the buttons call.

The agent never sees a password. It gets sent to the app's own sign-in page, the user approves it on a consent screen, and the agent walks away with a scoped token the user can revoke.

## Install

```bash
pnpm add better-auth @better-auth/oauth-provider
pnpm add mcp-handler @modelcontextprotocol/server
```

If `better-auth` is already installed from `references/auth.md`, upgrade it in the same command rather than leaving it behind — `@better-auth/oauth-provider` declares it as a peer dependency, and a version skew between the two fails later, at a confusing moment, rather than at install time.

Two reasons this file insists on the current release rather than whatever is already in the lockfile: the OAuth provider plugin is comparatively new and its surface is still settling, and the plugins it replaces carried a refresh-token replay flaw in older versions. This is security-relevant code. Do not build it on a stale install.

**Nothing new goes in `.env`.** `BETTER_AUTH_URL` now does three jobs — the app's origin, the OAuth issuer, and the audience stamped into every token — so in production it has to be the real public URL. Set it to anything else and the flow completes right up to the first tool call, then fails on an audience mismatch with no useful error. That one variable is the whole go-live switch.

## Configure

### One file for the strings that must agree

Four separate things have to be character-identical: the plugin's audience, the discovery document's `resource`, the audience checked at verify time, and the URL the user types into Claude. Put the strings in one place so they cannot drift.

`src/lib/mcp/resource.ts`:

```ts
const BASE_URL = process.env.BETTER_AUTH_URL ?? "http://localhost:3000";

export const ISSUER = `${BASE_URL}/api/auth`;
export const MCP_RESOURCE = `${BASE_URL}/api/mcp`;

// Two scopes, named after the app. Not one per table.
export const MCP_SCOPES = ["hikes:read", "hikes:write"] as const;
```

No trailing slash, ever — the spec prefers it absent and a stray one is a mismatch like any other. `ISSUER` includes `/api/auth`: Better Auth issues from its own base path, not from the bare origin, and pointing discovery at the origin is the most common way this fails silently.

Two scopes, not nine. A consent screen listing `hikes:read`, `hikes:write`, `photos:read`, `photos:write`… is a consent screen nobody reads, which defeats the point of having one.

### The OAuth provider

Extend `src/lib/auth.ts` — this adds a plugin alongside whatever is already there:

```ts
import { oauthProvider } from "@better-auth/oauth-provider";
import { MCP_RESOURCE, MCP_SCOPES } from "@/lib/mcp/resource";

export const auth = betterAuth({
  baseURL: process.env.BETTER_AUTH_URL,
  // ...existing config

  plugins: [
    // ...existing plugins
    oauthProvider({
      loginPage: "/sign-in",
      consentPage: "/oauth/consent",

      // Claude has no account here in advance, so it registers itself on first connect.
      allowDynamicClientRegistration: true,
      allowUnauthenticatedClientRegistration: true,

      validAudiences: [MCP_RESOURCE],
      scopes: ["openid", "profile", "email", "offline_access", ...MCP_SCOPES],
      clientRegistrationDefaultScopes: ["openid", "profile", "email"],
      clientRegistrationAllowedScopes: ["offline_access", ...MCP_SCOPES],

      accessTokenExpiresIn: "1h",
      refreshTokenExpiresIn: "30d",
    }),
  ],

  rateLimit: {
    enabled: true,
    storage: "database",
    customRules: {
      // `references/settings.md` adds its own rules to this object later.
      "/oauth2/register": { window: 60, max: 5 },
      "/oauth2/token": { window: 60, max: 30 },
    },
  },
});
```

`allowUnauthenticatedClientRegistration` sounds alarming and is required: the agent registers *before* anybody has signed in, because signing in is what it is about to ask for. Registration creates a client record, not access — nothing can be read until a human approves it on the consent screen. The rate limit is there because registration is the one endpoint an anonymous caller can reach, and each fresh connection makes a new client row.

This changes the schema, so regenerate and migrate:

```bash
pnpm dlx @better-auth/cli@latest generate --config src/lib/auth.ts --output src/lib/db/auth-schema.ts -y
pnpm db:generate
pnpm db:migrate
```

### Discovery

An agent finds all of this by fetching two well-known documents from the **domain root**. These need their own route handlers — the `/api/auth/[...all]` catch-all does not serve anything outside its own path, and expecting it to is a half-hour of confusion.

`src/app/.well-known/oauth-authorization-server/route.ts`:

```ts
import { oauthProviderAuthServerMetadata } from "@better-auth/oauth-provider";
import { auth } from "@/lib/auth";

export const GET = oauthProviderAuthServerMetadata(auth, {
  headers: { "Access-Control-Allow-Origin": "*" },
});
```

The second document describes the *resource*, and Better Auth has historically not served it — an authorisation server generally doesn't know what it is protecting. **Check what the installed version does before writing this by hand**; serving it natively is on the plugin's roadmap, and if it has landed, use that instead of the routes below. If it hasn't, write it once and mount it twice, because Claude probes the path-suffixed form first and the bare form second.

`src/lib/mcp/protected-resource.ts`:

```ts
import { createAuthClient } from "better-auth/client";
import { oauthProviderResourceClient } from "@better-auth/oauth-provider/resource-client";
import { auth } from "@/lib/auth";
import { ISSUER, MCP_RESOURCE, MCP_SCOPES } from "./resource";

const client = createAuthClient({ plugins: [oauthProviderResourceClient(auth)] });

const cors = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, OPTIONS",
  "Access-Control-Allow-Headers": "*",
};

export async function GET() {
  const metadata = await client.getProtectedResourceMetadata({
    resource: MCP_RESOURCE,
    authorization_servers: [ISSUER],
    scopes_supported: [...MCP_SCOPES],
    bearer_methods_supported: ["header"],
  });

  return Response.json(metadata, {
    headers: { ...cors, "Cache-Control": "public, max-age=15, stale-while-revalidate=15" },
  });
}

export const OPTIONS = () => new Response(null, { headers: cors });
```

Then two one-line routes, both re-exporting it:

```ts
// src/app/.well-known/oauth-protected-resource/api/mcp/route.ts
// and src/app/.well-known/oauth-protected-resource/route.ts
export { GET, OPTIONS } from "@/lib/mcp/protected-resource";
```

`authorization_servers` takes exactly one entry and Claude uses only the first — it does not try the others. `scopes_supported` here lists the app's two scopes and deliberately leaves out `offline_access`: that belongs to the authorisation server, and Claude adds it by itself when it wants a refresh token.

Leave a comment on `protected-resource.ts` saying it exists only until Better Auth serves this document itself, and that it should be deleted then. Without the comment it is still here in two years, quietly shadowing the built-in.

### Verifying a token

`src/lib/mcp/verify.ts`:

```ts
import "server-only";
import { createAuthClient } from "better-auth/client";
import { oauthProviderResourceClient } from "@better-auth/oauth-provider/resource-client";
import { auth } from "@/lib/auth";
import { ISSUER, MCP_RESOURCE } from "./resource";

const client = createAuthClient({ plugins: [oauthProviderResourceClient(auth)] });

export async function verifyBearer(token: string) {
  const payload = await client.verifyAccessToken(token, {
    verifyOptions: { issuer: ISSUER, audience: MCP_RESOURCE },
  });

  return {
    userId: String(payload.sub),
    clientId: String(payload.client_id ?? payload.azp ?? ""),
    scopes: String(payload.scope ?? "").split(" ").filter(Boolean),
    expiresAt: payload.exp,
  };
}
```

Log one decoded payload to the terminal the first time this runs and check the field names against what you actually got before moving on. The shape is stable but it is the one thing here worth confirming with your own eyes, and a wrong field name gives you `undefined` as a user id rather than an error.

### The endpoint

`src/app/api/mcp/route.ts`:

```ts
import { createMcpHandler, withMcpAuth } from "mcp-handler";
import { registerTools } from "@/lib/mcp/tools";
import { verifyBearer } from "@/lib/mcp/verify";

export const runtime = "nodejs";

const mcp = createMcpHandler(registerTools, {
  serverInfo: { name: "traillog", version: "1.0.0" },
});

const handler = withMcpAuth(
  mcp,
  async (_req, bearerToken) => {
    if (!bearerToken) return undefined;
    try {
      const { userId, clientId, scopes, expiresAt } = await verifyBearer(bearerToken);
      return { token: bearerToken, clientId, scopes, expiresAt, extra: { userId } };
    } catch {
      return undefined; // -> 401 with the challenge header
    }
  },
  {
    required: true,
    requiredScopes: ["hikes:read"],
    resourceMetadataPath: "/.well-known/oauth-protected-resource/api/mcp",
  },
);

export { handler as GET, handler as POST, handler as DELETE };
```

**`required: true` is not optional, despite the name.** It defaults to `false`, and with the default a request carrying no token runs the tools anyway. Nothing looks broken — the app serves unauthenticated traffic, Claude never receives a 401, and so it never starts the sign-in flow at all. This is the single most expensive line to leave out.

Returning `undefined` is what produces the 401. Do not catch the failure and return a friendly error object instead: a `200` carrying `isError: true` is an ordinary tool failure, so Claude hands the text to the model and moves on. Only a real 401 makes it stop and ask the user to sign in.

## The tools

### They call the same code the buttons call

Put the app's reads and writes in `src/lib/<domain>.ts` — `listHikes({ userId, limit })`, `createHike({ userId, ... })` — and have both the server actions and the tools call those. A tool that writes its own query is a tool whose ownership check drifts from the one the UI enforces, and nobody notices until the drift is a leak. One function, two callers, one `where`.

### The wrapper that enforces the hard rule

`src/lib/mcp/log.ts` — every tool goes through this, which is what makes "take the user from the token" structural instead of a thing to remember:

```ts
import "server-only";
import { db } from "@/lib/db";
import { mcpCallLog } from "@/lib/db/schema";

type Ctx = {
  http?: { authInfo?: { clientId?: string; scopes?: string[]; extra?: { userId?: string } } };
};

export function logged<A>(
  tool: string,
  scope: string,
  run: (args: A, userId: string) => Promise<unknown>,
) {
  return async (args: A, ctx: Ctx) => {
    const userId = ctx.http?.authInfo?.extra?.userId;
    if (!userId) throw new Error("No user on this token");
    if (!ctx.http?.authInfo?.scopes?.includes(scope)) {
      throw new Error(`This connection was not granted ${scope}`);
    }

    const clientId = ctx.http?.authInfo?.clientId ?? null;
    const startedAt = Date.now();

    try {
      const result = await run(args, userId);
      await db.insert(mcpCallLog).values({
        tool,
        userId,
        clientId,
        ok: true,
        durationMs: Date.now() - startedAt,
        rowCount: Array.isArray(result) ? result.length : null,
      });
      return { content: [{ type: "text" as const, text: JSON.stringify(result) }] };
    } catch (error) {
      await db.insert(mcpCallLog).values({
        tool,
        userId,
        clientId,
        ok: false,
        durationMs: Date.now() - startedAt,
        error: error instanceof Error ? error.message : String(error),
      });
      throw error;
    }
  };
}
```

The table goes in `src/lib/db/schema.ts` alongside the app's own — Postgres branch shown, SQLite uses `text` ids and `integer` timestamps per `references/database.md`:

```ts
export const mcpCallLog = pgTable("mcp_call_log", {
  id: uuid("id").primaryKey().defaultRandom(),
  userId: text("user_id").references(() => user.id, { onDelete: "set null" }),
  clientId: text("client_id"),
  tool: text("tool").notNull(),
  ok: boolean("ok").notNull(),
  durationMs: integer("duration_ms"),
  rowCount: integer("row_count"),
  error: text("error"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});
```

`pnpm db:generate` then `pnpm db:migrate` — this one is the app's own table, so it does not need the Better Auth CLI.

`onDelete: "set null"` rather than `cascade`, for the same reason `references/ops.md` gives: the record of what an agent did outlives the account it did it to.

**This log records reads as well as writes**, which is the one place this skill departs from `references/ops.md`'s "log writes, not reads". When the caller is a person, a read is noise. When the caller is an agent, a read *is* the event worth seeing — it is how data leaves the app. Tools that write should additionally call the existing `logActivity()` with `{ via: "mcp" }` in the detail, so the ordinary activity log keeps telling the truth about who changed what.

### Writing a tool

```ts
// src/lib/mcp/tools.ts
import "server-only";
import { z } from "zod";
import type { McpServer } from "@modelcontextprotocol/server";
import { listHikes, createHike } from "@/lib/hikes";
import { logged } from "./log";

export function registerTools(server: McpServer) {
  server.registerTool(
    "list_hikes",
    {
      title: "List hikes",
      description:
        "List the signed-in person's hikes, newest first. Use to answer questions about what they have walked, or to find a hike before changing it.",
      inputSchema: z.object({
        since: z.string().optional().describe("Only hikes on or after this date (YYYY-MM-DD)."),
        limit: z.number().int().min(1).max(50).default(20),
      }),
      annotations: { readOnlyHint: true },
    },
    logged("list_hikes", "hikes:read", ({ since, limit }, userId) =>
      listHikes({ userId, since, limit }),
    ),
  );

  server.registerTool(
    "log_hike",
    {
      title: "Log a hike",
      description: "Record a hike the person has been on. Ask them for the trail and date if either is missing.",
      inputSchema: z.object({
        trail: z.string().min(1).max(200),
        date: z.string().describe("YYYY-MM-DD"),
        distanceKm: z.number().positive().optional(),
        notes: z.string().max(2000).optional(),
      }),
      annotations: { destructiveHint: false },
    },
    logged("log_hike", "hikes:write", (input, userId) => createHike({ userId, ...input })),
  );
}
```

`inputSchema` takes a whole `z.object({...})`, not a bare shape, and the registration function is `registerTool` — `server.tool()` is from the retired adapter. Both are things half the examples online still get wrong.

**The scope argument is per-tool for a reason.** `requiredScopes` on the handler decides who may *connect*; it cannot decide who may write, because it sees the same value for every call. Without the check inside `logged()`, a connection the user approved for reading only can call every write tool in the app, and the consent screen they read becomes a lie.

### The rules that make tools good rather than merely present

- **Task-shaped, not table-shaped.** Name them after what a person would ask for — `log_hike`, `weekly_summary` — not `create_row` or `query_table`. A mechanical CRUD-per-table mapping technically works and produces a tool list an agent uses badly, because no single tool matches anything anyone actually wants.
- **Reads and writes are separate tools.** Never one tool with a `method` or `action` argument. Anthropic rejects catch-all tools outright in connector review, and it makes the next rule impossible.
- **Every tool declares `readOnlyHint: true` or `destructiveHint`.** These drive whether Claude runs a tool without asking. Label a write as read-only and it will fire without confirmation.
- **The description says what it does and when to use it.** It does not contain instructions aimed at the model, and it does not oversell — a description that claims more than the tool does is how an agent picks the wrong one.
- **Every list tool takes a `limit` with a hard maximum.** Results are capped near 150,000 characters on Claude.ai and around 25,000 tokens in Claude Code. An uncapped list either truncates mid-JSON into something unparseable, or hands a single call the entire account. Cap it in the schema *and* in the query, and return a cursor if there is more.
- **Destructive tools need a reason to exist.** Deleting is nearly always better done by the person, in the app. If the interview genuinely called for it, require an id the agent had to read first, mark it `destructiveHint: true`, and say so in the description.
- **Names stay under 64 characters** and read as `verb_noun`.

Pick the four or five tools that cover what the user said the app is *for* in Step 1a. A short list of tools that match real intentions beats a complete list that mirrors the schema.

## What the user can see and control

An agent working out of sight is exactly the thing this skill says an app must never have. Three pieces, and none of them is optional.

### The consent screen

`src/app/oauth/consent/page.tsx`, styled like the rest of the app — this is where somebody decides whether Claude gets their data, and a page that looks nothing like the app they just signed into reads as a phishing attempt.

Read the pending request through the plugin's client (`oauthProviderClient` on `authClient`, backed by `/api/auth/oauth2/get-consent`), then show:

- **Who is asking**, by the client name it registered with, and a plain warning that this name is chosen by whoever is connecting.
- **What it will be able to do**, in the app's own words. "Read your hikes" and "Add and edit hikes" — never the raw scope strings.
- **Whose account it will act as** — their email, visible, so a signed-in-as-the-wrong-person mistake is caught here.
- Two buttons of equal weight. Approve is not the primary-coloured one.

Accepting posts to `/api/auth/oauth2/consent`. Do not auto-approve, and do not skip the screen for "trusted" clients — with dynamic registration any client can call itself anything.

### Connected apps

`references/settings.md` builds this section: what has access, what it can do, when it was granted, and a Revoke button. Revoking has to stop the *next* tool call, not just remove a row from a list.

### The call log

`references/ops.md` renders `mcp_call_log` on `/settings/system`: which tool, which client, worked or failed, how long, how many rows went out. This is the panel that answers "what did Claude actually do?" and "why did it say it couldn't find anything?"

## Testing locally

**Claude Code talks to localhost.** No tunnel, no deploy — the whole flow works the moment the routes exist:

```bash
claude mcp add --transport http traillog http://localhost:3000/api/mcp
```

Then `/mcp` inside Claude Code to run the sign-in. The browser opens the app's own sign-in page, then the consent screen, and the tools appear. This is the loop to iterate in.

For tool shapes and schemas without the auth round trip, `pnpm dlx @modelcontextprotocol/inspector` against the same URL.

**Claude.ai and Claude Desktop cannot reach localhost** — they connect from Anthropic's servers, not from the user's machine. To test as a real connector, tunnel:

```bash
cloudflared tunnel --url http://localhost:3000
```

Set `BETTER_AUTH_URL` to the tunnel's public URL and restart the dev server, or every token comes out stamped with the wrong audience. Then add the tunnel URL plus `/api/mcp` under **Settings → Connectors → Add custom connector** on claude.ai.

If discovery fails, check it by hand before guessing — these three must line up exactly:

```bash
curl -s http://localhost:3000/.well-known/oauth-protected-resource/api/mcp | jq
curl -s http://localhost:3000/.well-known/oauth-authorization-server | jq
curl -si -X POST http://localhost:3000/api/mcp | head -20
```

The third must be a `401` carrying a `WWW-Authenticate: Bearer ... resource_metadata="..."` header. A `200` means `required: true` is missing.

One thing this file could not confirm: whether Better Auth matches Claude Code's loopback redirect port-agnostically. Claude Code listens on a fresh ephemeral port each time and registers `http://localhost/callback`. If sign-in from Claude Code fails at the redirect step with a mismatch, that is the cause — check the plugin's current redirect-matching options rather than working around it.

## Going to production

- `BETTER_AUTH_URL` becomes the real public URL on the host. That is the entire change; everything else derives from it.
- Give the user the connector URL — their domain plus `/api/mcp` — and show them where it goes in Claude. They will not find it on their own.
- If the host sits behind a proxy that rewrites the origin, pass `resourceUrl` to `withMcpAuth` explicitly. On Vercel the forwarded headers are already right.
- Dynamic registration creates a client row per fresh connection. Fine for one person; if the app gets popular, mention that old unused clients are worth pruning.
- Two things on Better Auth's roadmap remove code from this file: serving the protected-resource document natively, and client metadata documents, which replace dynamic registration outright. Both are worth taking when they land, and both need a database migration — so take them deliberately, not by accident during an unrelated upgrade.

## Harnesses that can't do OAuth

Some automation tools — n8n, a self-hosted script, a home-grown harness — only send a static header. The temptation is to add a personal access token, and it is worth saying plainly what that costs: a long-lived credential that grants everything the user can do, that lives in another system's settings screen, that never expires, and that is the one people paste into a chat message. Every agent that matters here — Claude, Claude Code, ChatGPT, Cursor — does OAuth properly.

Don't build it unless the user asks for it specifically and understands that. If they do, four rules keep it survivable: issue it from `/settings` so it is visible where access is managed; verify it in the same function that verifies OAuth tokens so the tool layer never learns there are two kinds of caller; give it the same two scopes, not a bypass; and list it in Connected apps with a revoke button beside the others.

## Verify

- An unauthenticated `POST /api/mcp` returns `401` with a `WWW-Authenticate` header containing `resource_metadata`, not a `200`.
- Both `.well-known` documents return JSON, and two pairs match character for character: the protected-resource document's `resource` against `validAudiences[0]` in `src/lib/auth.ts`, and its `authorization_servers[0]` against the `issuer` in the authorisation-server document. No trailing slash on either.
- `offline_access` appears in the authorisation server document and does not appear in the protected-resource document.
- Adding the server in Claude Code opens the app's own sign-in page, then a consent screen wearing the app's design, and the tools appear afterwards.
- A read tool returns only the signed-in user's rows, and asking for more than the schema's maximum is rejected rather than clamped silently.
- **Second account test:** signed in as a different user, ask the agent for the first account's record by its id. It fails server-side — an empty result is not good enough, because that means the query ran.
- A write tool creates a real row that shows up in the app's own UI on refresh, and the activity log records it as the app's verb with `via: "mcp"`.
- Approving read access only, then asking the agent to write, is refused by the tool — not merely absent from the consent screen.
- `/settings/system` lists the calls that were just made, including the reads, with the tool name and the client.
- Revoking the connection in Connected apps makes the very next tool call fail, without restarting the server.
- Every tool has a `title`, a `readOnlyHint` or `destructiveHint`, and a description that names a real reason to call it — no `create_row`, no catch-all with a `method` argument.
- With `BETTER_AUTH_URL` absent the app still starts, and the system page reports the MCP endpoint as not configured rather than crashing.
