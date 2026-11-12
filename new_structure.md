# New project structure — the agent-operating foundation

> **Version 2026-11-12.** This structure was built and proven in a production SaaS repo (MultiAngle.ai). It adds
> 2025–26 best practice from the official Claude Code docs (checked against Claude Code 2.1.304), Anthropic
> engineering posts, Fowler, matklad, Google SRE, OWASP and the GitHub docs.
>
> **Who this is for:** a Claude Code session opened at the root of a project, new or existing.
>
> **How to start:**
> 1. Read §0–§17 once, then Appendix E and the Sources at the end. Appendices A–D are one page: the list of kit
>    scripts and the commands that copy them (§0.0). You never read or retype the scripts themselves.
> 2. Do §0.0. Check for an existing `chore/agent-setup-*` branch, then create or enter the branch (§0.2). Then do §2 Discovery.
> 3. Build Phase A, then Phase B, then report (§17).
>
> **If you are not the author.** This guide was written for one person's machine and says so in places. Read it this way:
> - **"The owner"** is whoever runs the setup prompt: you.
> - **"The global layer"** (§1, §12) is the author's private `~/.claude/` setup: personal working rules, review and
>   commit gates, a command proxy (`rtk`), skills and a subagent model table. You don't have it and don't need it.
>   §2 step 8 will report it missing; that is expected. Wherever the guide says the global layer already does
>   something (the 800-line Write limit, the `--no-verify` block, the `/code-review` push gate, format-on-edit for
>   Python), treat it as absent.
> - **§4's Working rules block** holds the author's own preferences. Replace the bullets with yours.
> - **`rtk`, `gsd-map-codebase` and the named global skills** are optional tools. Every step has a path without them.
>
> The kit, the file tree and the templates work on any machine with the tools listed under "Appendices A–D".

---

## 0. How to use this document

**Goal:** Claude Code should be able to work in this repo for years without:
- re-deriving the architecture every session;
- breaking invariants that were learned the hard way;
- losing the thread across sessions, compactions and worktrees;
- mixing this project up with the owner's other projects.

**Order of work**

| Step | What | Sections |
|---|---|---|
| 0.0 | Find the kit | below |
| Discovery | Read-only survey, then a mapping table | §2 |
| Phase A | Build the agent-operating structure | §3–§10, §12–§13 |
| Phase B | Audit against the foundation checklist. Build the DAY-1 items §0.3 allows. List the rest. | §11 |
| Finish | Verify, commit in groups, report | §16–§17 |

**Before you start:**
- Turn the §16 checklist into your task list.
- If a branch `chore/agent-setup-*` already exists, **resume** from where its §16 checklist stopped (see `.planning/agent-setup.md`) instead of starting over.
- The global hook refuses a single Write over 800 lines. Split a long file into several files, or write the first part and append the rest with `cat >> <file> <<'EOF'`.

### 0.0 Find the kit

The hook scripts (Appendices A–D) are **copied from the kit, never retyped**. The kit is the `kit/claude/` folder next to this document.

- **Default location:** `~/new-project-foundation/kit/claude`. Check it: `[ -d ~/new-project-foundation/kit/claude ] && echo kit`.
- **Somewhere else:** if you opened this document from another folder, the kit is `<that folder>/kit/claude`. Use that folder wherever this guide writes `~/new-project-foundation`.

If the kit is missing (the text was pasted from somewhere else), ask the owner where the foundation folder is. **That is the only question you ask before the end.** Then proceed.

### 0.1 Never

- **Never edit anything under `~/.claude/`.** That covers the global CLAUDE.md, hooks, skills, agents, plugins, settings, output styles and other projects' memory. Every project on this machine shares it. If something there is wrong for this repo, report it.
  - **The only exception:** during this setup you may create this repo's own memory folder with exactly two files, `MEMORY.md` and `reference_docs_index.md` (§10, E.7). Do it only if `MEMORY.md` doesn't exist yet. Never edit existing memory during setup.
- **Never install plugins or tools.** `claude plugin install`, `brew` and global `npm -g` all write outside the repo. List them in the report as owner actions.
- **Never copy content from another project** (§15). Copy the patterns, never the facts.
- **Never write an unverified fact** into CLAUDE.md, a rule, the map or a deep-dive.
  - "Verified" means checked in this repo's code, CI or tests.
  - A false rule is worse than no rule: when two sources disagree, Claude picks one arbitrarily.
  - Text you relocate verbatim from an existing file is exempt.
- **Never open a secret file or repeat a secret.**
  - Don't open `.env*` files, except `.env.example`.
  - Don't print or quote any secret value. This covers tokens in `.mcp.json` too.
  - Never copy values into docs. Name the variable only.
- **Never touch data during setup.** Don't connect to a database, and don't run migrations, seed or bulk scripts, or the test suite. `.env` may point at production. The only checks you run are:
  - `verify` (typecheck, lint, format);
  - `hooks.test.sh`;
  - census test files you added that only read source files.
- **Never `git add -A` or `git add .`.** Stage only files you created or deliberately changed. Other sessions may have uncommitted work in the same tree.
- **Never use `--no-verify`** (it is blocked globally anyway). Never force-push. Never push.
- **Never delete existing docs or instructions.** Relocate them (usually into `docs/`) and leave a one-line pointer.

### 0.2 Work on a branch

- **If the session has the `EnterWorktree` tool:**
  1. Call `EnterWorktree` (`name: "agent-setup"`) instead of `cd`-ing in each command.
  2. Rename its branch: `git branch -m chore/agent-setup-<YYMMDD>`.
  3. A fresh worktree has no `node_modules` and no `.env`. Restore the repo's already-declared dependencies from its lockfile (`npm ci`, `pnpm i --frozen-lockfile`, `uv sync --frozen`). Add `--ignore-scripts` if a `prepare` script would enable git hooks. Never add a dependency.
- **Otherwise:** run `git checkout -b chore/agent-setup-<YYMMDD>`.
- Commit in logical groups (§16). Push nothing.

### 0.3 What you may do alone, and what waits for the owner

**Allowed without asking:**
- Creating new files.
- Appending to or merging into existing files (CLAUDE.md, `.gitignore`, `settings.json`, docs).
- Relocating existing text verbatim, leaving a pointer behind.
- Writing config that takes effect **only after merge**: CI workflows, `dependabot.yml`, ruleset JSON. Do it on the branch and list it in the report.

**Not allowed without the owner.** Skip these, finish everything else, and ask about them once, in the §17 question block:
- pushing, opening or merging a PR, or deploying;
- replacing an existing file wholesale, or deleting a file you did not create;
- anything that acts immediately or outside the repo:
  - installing new dependencies or tools (restoring the lockfile's set in a worktree, §0.2, is fine);
  - enabling git hooks (husky, pre-commit, gitleaks);
  - GitHub settings or rulesets via the API;
  - plugins;
- adding a paid or external service (error tracker, flag platform, paid CI);
- changing the product's runtime behaviour, for example config validation at boot that could stop the app from booting;
- turning on strict modes that make an existing codebase fail `verify`. Report the error count instead;
- anything that would need a change in `~/.claude/`.

**This setup assumes a solo owner.** On a repo shared with other developers (check in §2), the owner's personal working rules (§4) go in `CLAUDE.local.md` (gitignored), and CLAUDE.md gets no Working rules section.

---

## 1. The three layers

| Layer | Where | When it loads | Who maintains it | On the owner's Mac |
|---|---|---|---|---|
| **Global** | `~/.claude/`: `CLAUDE.md` (worktrees, subagent table, **the owner's Working rules** since 2026-10-19), `RTK.md`, `settings.json` hooks, `skills/`, `agents/`, `output-styles/owner.md`, `statusline-command.sh` | Every session, every project | The owner only | **Already installed.** Never duplicate it project-side. |
| **Project** | This repo: `CLAUDE.md`, `.claude/` (settings, rules, hooks, skills, agents), `RECONSTRUCTION.md`, `.planning/`, `docs/`, CI guards | Sessions in this repo | You (this session), reviewed through a PR | **What you build.** |
| **Memory** | `~/.claude/projects/<encoded-repo-path>/memory/` (`MEMORY.md` plus topic files) | The first 200 lines or 25 KB of `MEMORY.md` at session start; topic files on demand | Claude, automatically | **Starts empty** for this repo. |

What this means in practice:
- **Global gates already act on this repo** (§12). Don't copy them.
  - When the review gate blocks a push, its message tells the session to register out-of-scope findings in `.planning/DEFERRED-BY-BLOCKER.md` → Section F.
  - The gate never reads that file. Create it anyway so the instruction has somewhere to land.
- **The owner's working rules come from the global CLAUDE.md.** Check that `~/.claude/CLAUDE.md` has a "Working rules" section; if it does, this repo's CLAUDE.md must not repeat them. §4's block is a fallback for a machine without the global layer.
- **Memory is keyed to the git repository.** All worktrees share it. Moving or renaming the repo folder starts a new, empty memory, so create the repo at its final path.

---

## 2. Step 0 — Discovery (read-only)

Run the survey in an Explore subagent (Sonnet, effort xhigh) so file dumps stay out of your context. **It must not open `.env*` files** (except `.env.example`). Collect:

1. **Stack:** languages, frameworks, package manager or task runner, runtime version pin (`.nvmrc`, `mise.toml`, `.python-version`, `engines`), and whether it is a monorepo.
2. **Commands that exist:** dev, build, typecheck, lint, format, test, e2e, migrate. Note which are missing.
3. **Source directories.** These become `scopedPaths` in §5, and each one must contain tracked files. Also note the test layout.
4. **CI:** the system, the workflows, required checks, branch protection or rulesets.
5. **Existing agent files:**
   - `CLAUDE.md` (with its line count), `CLAUDE.local.md`, `AGENTS.md`;
   - `.claude/` (settings, rules, hooks, skills, agents), `.mcp.json` server names only — read them with `python3 -c 'import json;print(list(json.load(open(".mcp.json")).get("mcpServers",{})))'` and never open the file itself, which can hold tokens — `.cursor/`, `.github/copilot-instructions.md`;
   - `docs/`, ADRs, incident notes, `.planning/`;
   - whether `.gitignore` ignores `.claude/` or `.planning/`.
6. **Git:**
   - the default branch name;
   - the branch or tag that production deploys from;
   - remotes;
   - whether `gh auth status` is OK;
   - contributors: `git shortlog -sn HEAD | head -5`. More than one human author means a shared repo (§0.3, last paragraph).
7. **Environments:** the deploy target; dev, staging and prod; which databases exist and whether dev and prod are separate. Infer these from config and docs, never by connecting.
8. **Global layer present?** Run this. Report anything missing; don't install it:
   ```bash
   ls ~/.claude/CLAUDE.md ~/.claude/RTK.md ~/.claude/output-styles/owner.md ~/.claude/statusline-command.sh ~/.claude/hooks/{code-review-gate.sh,code-review-lib.sh,code-review-mark.sh,code-review-clear.sh,citation-gate.sh,check-citation-drift.mjs,block-no-verify.sh,block-large-files.sh,context-ceiling.js,resume-after-compact.sh} ~/.claude/skills/{bearings,handoffprompt,reviewers-run-check-changes-and-find-errors,deep-qa-sweep,karpathy-guidelines,steal-this-code,grilling,domain-modeling,context-budget,graphify}/SKILL.md
   ```
9. **Tools and versions:**
   - `command -v jq python3 git node rtk ast-grep gh gitleaks`;
   - `claude --version`. This guide assumes **2.1.295 or later**. On an older version `onFailure: block` is ignored, so the push guard would fail open if it crashed. Report that; everything else still works.
   - the LSP plugin for the main language (`claude plugin list`).
10. **Docs tools:** whether a Context7 MCP server is connected (tools named `resolve-library-id` / `query-docs`, or `claude mcp list`).

**Stack research (after the survey, before building anything stack-specific).** Nothing in this guide assumes a language. Training data is stale for fast-moving frameworks, so for the detected runtime, framework and major tools:
1. Pull the **current** docs for the exact major version in the lockfile: through Context7 when it is connected (`resolve-library-id`, then `query-docs` for "project setup, lint/typecheck/format commands, testing, breaking changes"), otherwise `WebFetch` of the official docs and changelog. Never rely on memory for a command, flag or config key.
2. Record in `.planning/agent-setup.md` → `## Stack research`: versions; breaking changes since your training data; the formatter, linter, typechecker and test runner the ecosystem uses today; the module-boundary tool (§11.1); the LSP plugin name (§5); whether the global gates cover this language (§12); and the source URL or Context7 id of each fact.
3. Adapt every `<pm>`, `verify`, formatter, rule and gate line in this guide to those findings. A command you did not confirm in the docs or by running it does not go into CLAUDE.md.
4. If the framework writes its own agent-instructions block into CLAUDE.md (Next.js does), keep it.

**Write the mapping table to `.planning/agent-setup.md`.** Also add a one-line pointer to it under STATE.md → `## Current focus`, so it survives compaction. If STATE.md doesn't exist yet, create it from E.3 first. Below the table, keep the §16 progress and (later) the Phase B gap report. Example rows:

| Item | Found | Plan |
|---|---|---|
| CLAUDE.md | 250 lines, includes an architecture dump | Relocate the dump verbatim to `docs/ARCHITECTURE.md`; trim to 200 lines or fewer |
| `.claude/rules/` | none | db, auth, core-domain, docs-sync |
| CI | one workflow, no required check | Add `ci-success` (Phase B, §11.2) |

**Existing `.gitignore` ignores `.claude/` or `.planning/`:** add the new files with `git add -f`. Keep `.claude/settings.local.json` ignored.

**Empty repo (brand-new project):**
1. Run `git init -b main && git commit --allow-empty -m "chore: init"`. This is the one allowed commit on the default branch, and it is never pushed. The hooks and the stamp script need a `HEAD` to exist.
2. Then create the branch (§0.2).
3. **Don't scaffold the app** unless the owner already stated the product and stack. Build the agent layer with `TBD` placeholders, and put "stack / product?" in the question block.
4. Once the stack is known, §11 becomes the scaffolding checklist.

---

## 3. Target file tree

```
CLAUDE.md                     ≤200 lines, always loaded (§4)
RECONSTRUCTION.md             ≤100-line navigation map, stamped (§5)
CONTEXT.md                    domain glossary, 10–30 terms (§9)
.claude/
  settings.json               hooks + permissions (§7, Appendix D)
  bearings.json               codemap freshness config (§5, Appendix A)
  guard-push.json             protected branches for the push guard (§7, Appendix B)
  hooks/                      bearings-check.sh  bearings-stamp.sh  require-pr-on-blocker.sh
                              restore-state.sh   hooks.test.sh
  rules/<area>.md             path-scoped landmines (§6)
  skills/  agents/            project-specific ones only (§12)
.planning/                    tracked in git; never delete (§8)
  STATE.md                    current focus, blockers, decisions in flight
  DEFERRED-BY-BLOCKER.md      Section F: ledger of real-but-out-of-scope review findings
  agent-setup.md              mapping table + setup progress (§2)
  codebase/                   7 generated deep-dives, stamped (§5)
docs/
  ARCHITECTURE.md             5-minute mental model, layer rules, "where to add X"
  ADR/README.md  ADR/0001-*.md   immutable architecture decisions (§9)
  INCIDENTS.md                numbered incidents; each ends with a guard (§9)
  DECISIONS.md                short durable "why" log (optional)
  specs/                      one-page spec before each big feature (§11.5)
  (when needed) DEPLOYMENT.md ENV-FLAGS.md SECRETS.md CI.md API.md DATABASE.md runbooks/
scripts/verify  (or a runner task "verify")   ONE command: typecheck + lint + format check (§11.2)
.worktreeinclude              gitignored files to copy into new Claude worktrees (e.g. .env.local)
.env.example                  every env var name, no values
.gitignore                    add: .claude/settings.local.json  .claude/worktrees/  CLAUDE.local.md  .env*  !.env.example
```

**Any of these files already exists?** Keep it and add only the missing sections; never replace it. If a relocation target (for example `docs/ARCHITECTURE.md`) already exists, append the moved block under a new `##` heading.

---

## 4. CLAUDE.md

### Rules for the file

- **200 lines or fewer.** It loads every session and is re-read after `/compact`. Long files get rules ignored.
- **Each fact has one home:**
  - path-specific guidance goes in `.claude/rules/`;
  - geography goes in `RECONSTRUCTION.md`;
  - long references go in `docs/`, with a one-line pointer here.
- **Content test:** "would removing this line cause a mistake?" If not, cut it. Leave out anything Claude can read from the code itself: no file-by-file descriptions, no tutorials.
- **Never `@import` the map, the deep-dives or a big doc.** Imports load at launch and cost context every session.
- **Use HTML comments for maintainer notes.** `<!-- -->` blocks are stripped from context.
- **Don't repeat global rules.** `~/.claude/CLAUDE.md` already covers EnterWorktree, the subagent model/effort table, ast-grep, pen.dev design prompts and the owner's Working rules.
- **`<pm>` below** means this repo's task runner: `npm run`, `pnpm`, `uv run`, `make` and so on. If the repo has none, create `scripts/verify` as a small shell script and call it directly.

### An existing CLAUDE.md

If CLAUDE.md is over 200 lines or full of reference material:
1. Move whole blocks **verbatim** into `docs/<TOPIC>.md` (an architecture dump goes to `docs/ARCHITECTURE.md`; an env list goes to `docs/ENV-FLAGS.md`). Leave a one-line pointer behind.
2. Keep its commands and its 2–3 most expensive invariants.
3. Then add the missing sections below.

Relocated text doesn't need re-verifying. If a Working rules bullet contradicts a rule already in the repo, keep the repo's rule and report the conflict.

### Section order

1. Title, plus one paragraph on what this is.
2. Orientation: read this first.
3. Tech stack: only the versions that matter, and pinned dependencies with the reason.
4. Commands, including `verify`. Claude cannot guess these.
5. Environments and data: dev vs prod, the prod guard, and "migrations go to every environment".
6. Env deltas: only the production values that differ from code defaults, each with its rollback. The full list lives in `docs/ENV-FLAGS.md`.
7. Working rules (owner, binding). Only if the global CLAUDE.md lacks them; then use the block below.
8. Invariants: the 2–3 most expensive silent rules, each with its incident and guard test. These also live in `.claude/rules/`; repeating the costliest ones here is deliberate.
9. Branch, push and deploy discipline.
10. Conventions: short bullets.
11. Documentation index: one line per doc, plus the Consult / Record / Keep-in-sync rules.
12. Compaction line.

### Skeleton

````markdown
# <Product>

<One paragraph: what it does, for whom, where it runs.>

## Orientation — read this first
- **Navigation map:** `RECONSTRUCTION.md` (≤100 lines). Read it before editing unfamiliar areas.
- **Deep dives:** `.planning/codebase/{ARCHITECTURE,STRUCTURE,CONVENTIONS,TESTING,STACK,INTEGRATIONS,CONCERNS}.md`,
  each stamped `<!-- refreshed: -->` + `last_mapped_commit:`. <If not generated yet, write "not generated yet" here.>
- **Trust signal:** the session-start `Orientation:` / `Codebase maps:` lines say whether they are STALE. If STALE,
  trust the code over the docs and refresh (RECONSTRUCTION.md → "Deep dives").
- **Path-scoped rules** live in `.claude/rules/` and load when a matching file is read, written or edited. Don't copy them here.
- **Who calls X / where is Y defined:** use the LSP tool (`workspaceSymbol`, then `incomingCalls` / `goToDefinition`)
  before Grep. Grep is for strings, config keys and docs.

## Tech stack
- <framework + version>, <db>, <hosting>. <Pinned dep> stays at <x.y> because <reason / incident>.

## Commands
```bash
<pm> dev       # local dev server
<pm> verify    # typecheck + lint + format check — run before saying work is done
<pm> test      # full test suite (when asked, and in CI)
<pm> build     # only right before a push
```

## Environments & data
- Local and preview use the **dev** database, never production. `<path to prod guard>` makes tests and bulk scripts
  refuse the production endpoint.
- Every migration is applied to every environment; the PR says which were done.

## Env deltas (production values that differ from code defaults)
| Var | Prod | Effect | Rollback |
|---|---|---|---|
| <VAR> | <value> | <what it changes> | <how to undo> |
Full reference: `docs/ENV-FLAGS.md`.

## Working rules (owner, binding)
<paste the BULLETS of the Working rules block below — not its heading>

## Invariants (cheap to violate, expensive to discover)
- **<rule>.** <incident / why>. Guard: `<test path>`.

## Branch, push & deploy
- Default branch `<main>`; production deploys from `<prod branch or tag>`.
- Big or multi-file work → branch `feat|fix|chore/<slug>-<YYMMDD>` + PR. If another session may be active, always branch.
- A push to a protected branch is blocked while `.planning/STATE.md` → `### Blockers/Concerns` lists an unresolved push
  blocker (hook; format in that file). Clear it with a `RESOLVED:` line inside the bullet; never delete the entry.
- <Only if this repo's code is ts/tsx/js/jsx/mjs/cjs/sql/css:> `/code-review` must cover the changes before a push
  (global hook). Real but out-of-scope findings go to `.planning/DEFERRED-BY-BLOCKER.md` → Section F as `SWEEP-F<n>`.

## Conventions
- In code comments, cite code by **symbol**, not line number (`src/billing/charge.ts::chargeOnce`, never
  `charge.ts:120`): line citations drift. <Only for ts/js/sql/css code:> a global commit hook blocks self-falsifying ones.
- <naming, error handling, where constants live…>

## Documentation
- `docs/ARCHITECTURE.md` — mental model, layer rules, where to add X.
- `docs/ADR/` — immutable decisions. `docs/INCIDENTS.md` — numbered incidents + runbooks.
- <one line per other doc>
**Consult before changing:** <area> → <doc>. **Record decisions:** hard-to-reverse → new ADR; smaller → `docs/DECISIONS.md`,
with the reasoning. **Keep in sync:** see `.claude/rules/docs-sync.md` (change X → update doc Y in the same commit).

## Compaction
When compacting, always keep: the current task and its acceptance checks, the files modified, the commands used to
verify, and every owner decision made in this session.
````

### Working rules block

Paste the bullets and adapt the `<…>` parts. Drop a bullet marked *(if …)* when its condition doesn't hold. If `~/.claude/CLAUDE.md` already has a "Working rules" section, skip the whole block.

```markdown
## Working rules (owner, binding)
- **Finish approved work, THEN ask** every open decision in one batch, recommendation first. Never interleave
  questions with in-flight work. (Planning questions before starting are fine.)
- **Tell the owner what a PR ships before opening or merging it** — even docs-only.
- **Fix every review and QA finding in the branch, any size.** Hand big ones to a fix agent. Only findings that need
  a product or legal decision become questions; Section F is only for pre-existing or genuinely out-of-scope items.
- **No new tests unless asked** — except the guard test an incident requires (`docs/INCIDENTS.md`) and census tests.
  `<pm> verify` (typecheck/lint) is always fine; the owner verifies by running the app.
- **`<pm> build` only right before a push**, never while iterating.
- **Only do what the task asks.** Propose extras in the reply; don't build them.
- **Evidence over claims.** Never call work done without running the check; report failures in the first line.
- **A fresh worktree needs dependencies installed first.** Parallel agents get one folder each — never a shared scratch dir.
- *(if the repo has a UI)* **Less friction wins** in UX choices: pick the lowest-friction option, state the safety
  trade-off once. **UI QA must look at the screenshots**, not only at geometry/overflow probes.
- *(if the repo has a UI)* **Test/QA results go in a local HTML compare page** (verdict chips, metrics, before/after),
  opened for the owner.
```

This setup's census and test-integrity guards (§11.2) count as "asked".

---

## 5. Codemap: the orientation layer ("bearings")

**If a `bearings` skill is installed, don't run it end to end during this setup.** It covers the same ground in its
own order and would redo steps this guide has already placed. Follow this section instead. Install the hooks from
Appendices A and D.

**The idea:** routing plus freshness. Small always-on pointers, path rules that fire only when relevant, generated
deep-dives that are *stamped*, and a hook that says out loud when they are stale. Not a big map loaded every session.

Build the layers in this order:

1. **LSP plugin for the main language** (owner action; §0.1 forbids you installing it).
   - The owner installs the language-server binary, runs `claude plugin install <name>@claude-plugins-official`, then `/reload-plugins`.
   - TypeScript: `typescript-lsp`, binary `typescript-language-server`. Python: `pyright-lsp`, binary `pyright-langserver`. Go: `gopls-lsp`, binary `gopls`. Rust: `rust-analyzer-lsp`, binary `rust-analyzer`.
   - Put the exact commands in the report. It gives a live call graph that never goes stale.
2. **Rules** (§6).
3. **`RECONSTRUCTION.md`: 100 lines or fewer** (template E.2). Sections:
   - What this app is.
   - Directory map (dir, purpose, edit guidance).
   - Where the core business logic lives (10–15 real backticked paths).
   - Read these before editing.
   - Deep dives table.
   - Landmines you can't grep for: numbered geographic or historical facts.

   It is a map of pointers, never a copy of CLAUDE.md. Check that every path exists.
4. **`.planning/codebase/`: 7 deep-dives.** Use `gsd-map-codebase` only if it is installed, writes the stamp lines below, and does not commit. Otherwise spawn **4 parallel mapper agents** (Sonnet, effort xhigh), each writing straight into `.planning/codebase/` and returning only a confirmation:
   - **tech** → `STACK.md` (languages, runtime, frameworks, deps, config) + `INTEGRATIONS.md` (external APIs, DBs, auth providers, webhooks).
   - **arch** → `ARCHITECTURE.md` (pattern, layers, data flow, abstractions, entry points) + `STRUCTURE.md` (directory layout, key locations, naming).
   - **quality** → `CONVENTIONS.md` (style, naming, patterns, error handling; mark what lint/CI/tests *enforce* vs what is merely *customary*) + `TESTING.md` (framework, layout, mocking, how to run locally vs in CI).
   - **concerns** → `CONCERNS.md` (tech debt, bugs, security, performance, fragile areas, ranked by blast radius; only what is TRUE NOW).

   Every mapper prompt must include this paragraph:
   > "Every claim carries a backticked real file path. Start the file with `<!-- refreshed: <YYYY-MM-DD> -->`, then `last_mapped_commit: <git rev-parse --short HEAD>`, then the title, then `**Analysis Date:** <YYYY-MM-DD>`. Never open `.env*` files or print secret values; name variables only. Do not stage, commit, or revert anything."

   **Empty or tiny repo (under about 20 source files):** defer this layer. Point the map's Deep dives table and CLAUDE.md at "not generated yet".
5. **Freshness guard.**
   1. Install `bearings-check.sh` and `bearings-stamp.sh` into `.claude/hooks/` (Appendix A).
   2. Write `.claude/bearings.json`. Set `scopedPaths` to your real source dirs, and check that each one has tracked files: `git ls-files -- <dir> | head -1`.
   3. Wire SessionStart (Appendix D).
   4. After the map exists and is committed, run `bash .claude/hooks/bearings-stamp.sh`. It adds two stamp lines under the map's title; commit them: `git commit -m "chore: stamp the map" -- RECONSTRUCTION.md`.

   Every session then starts with up to 6 lines, for example:
   ```
   Orientation: RECONSTRUCTION.md — stamped 2026-10-19, 0 scoped commits behind — current
   Codebase maps: .planning/codebase/ (7 docs) — oldest refreshed 2026-10-19, 0 scoped commits behind — current
   Rules: 6 in .claude/rules/, 0 dead glob(s)
   ```
   - **STALE** means more than 30 days old, or more than 150 commits touching `scopedPaths` since the stamp.
   - The guard never blocks: it always exits 0.
   - It says `unreachable` when a stamp SHA vanished after a rebase. It never silently reports "0 behind".
   - The dead-glob count uses `git ls-files`, so stage new files before you trust it.

**Refresh when STALE:**
1. Re-map only the drifted directories (`gsd-map-codebase --paths <dirs>` or the scoped mapper prompt).
2. Fix the map's drifted rows and commit them.
3. Re-stamp, then commit the stamp.

That takes minutes, not hours.

**Don't:**
- `@import` the map into CLAUDE.md;
- regenerate every session;
- build a vector or knowledge-graph index instead. It is another static artifact on the same rot clock, and the LSP is already live.

---

## 6. Path-scoped rules: `.claude/rules/<area>.md`

**Format.** The only frontmatter key is `paths:` (template E.1). Example:

```markdown
---
paths:
  - "src/db/**"
  - "migrations/**"
---
# Database — landmines

- **Never edit an applied migration; add a new one.** `<the CI check that enforces it, if one exists>`.
- **<Invariant that is expensive or silent to violate.>** <Why, in one clause — the incident or the guard test.>
- Change here → update `docs/DATABASE.md` in the same commit.
```

**Rules for rule files:**
- **One file per risky area that exists.** The usual set: db, auth, money/billing, the core domain, build-config, ui, testing/CI and docs-sync.
- **Size:** 30 lines or fewer each, and 15 or fewer when the globs hit hundreds of files. Write 6–10 bullets, each shaped as: **invariant** → why (incident or guard) → doc to update.
- **When they load:** when Claude reads, writes or edits a matching file (since Claude Code 2.1.288; older versions load on Read only). A rule without `paths:` loads every session. Avoid that, because CLAUDE.md does that job.
- **Every glob must match at least one tracked file.** Check with `git ls-files | grep -E '<regex>'`. The session-start line counts dead globs.
- **Every fact must be verified** in code, CI or tests before you write it. Cite the proof in the bullet.

**Always create `docs-sync.md`** (keep only rows whose doc exists, and add rows as docs appear):

```markdown
---
paths:
  - "CLAUDE.md"
  - "RECONSTRUCTION.md"
  - "docs/**"
  - ".claude/**"
---
# Docs ownership — which doc owns which fact

| Changing… | Update in the same commit |
|---|---|
| an API endpoint / server action | `docs/API.md` |
| the DB schema | `docs/DATABASE.md` |
| env vars / deploy config | `docs/DEPLOYMENT.md` + `docs/ENV-FLAGS.md` |
| error handling / retries / outages | `docs/INCIDENTS.md` |
| CI workflows | `docs/CI.md` |
| a top-level dir or core module | `RECONSTRUCTION.md` |
| a hard-to-reverse architecture choice | new ADR in `docs/ADR/` |

- Rules live in CLAUDE.md; geography in RECONSTRUCTION.md; deep dives in `.planning/codebase/`. Don't copy across — point.
- CLAUDE.md stays ≤200 lines; RECONSTRUCTION.md ≤100.
```

---

## 7. Project hooks and settings

`.claude/settings.json` (full file in Appendix D; merge it into an existing file, never overwrite):

| Entry | What it does | Fails |
|---|---|---|
| SessionStart → `bearings-check.sh` | Prints the orientation and staleness lines | Open, always exits 0 (informational) |
| SessionStart, matcher `compact` → `restore-state.sh` | Re-injects `.planning/STATE.md` → `## Current focus` after compaction | Open, always exits 0 |
| PreToolUse `Bash` → `require-pr-on-blocker.sh`, with `"onFailure": "block"` | Blocks a push to a protected branch while STATE.md has an unresolved push blocker | **Closed** |
| `permissions.deny` on `.env`, `.env.local`, `.env.*.local`, `.env.development`, `.env.production`, `.env.test`, `secrets/**` | Claude's file tools can't read secrets. `.env.example` stays editable. Add any other real env-file names found in Discovery. | Holds in every mode, including bypassPermissions |
| `permissions.ask` on `git push` and `rtk git push` | Every push asks the human, as a second layer behind the hook | Still prompts in bypassPermissions |

Set **`.claude/guard-push.json` → `protectedBranches`** to the default branch and the production branch found in §2, for example `["main", "production"]`. Shell globs are allowed: `"release/*"`.

Facts that shape these choices (official docs):
- **What blocks a PreToolUse call:**
  - exit 2;
  - exit 0 with JSON `permissionDecision: "deny"`.
- **What doesn't:** any other exit, a crash or a timeout lets the action **proceed**, unless the entry has `"onFailure": "block"` (Claude Code 2.1.295+). So every guard gets `onFailure: block`.
- **A hook `deny` and deny rules hold even in bypassPermissions.** Ask rules prompt there too.
- **Rule order: deny, then ask, then allow; the first match wins.** So a project `ask` on `git push` holds even though the owner's global settings carry an old `allow` for `Bash(git push:*)`.
- **Deny rules don't cover subprocesses.** They cover Claude's file tools and recognised shell reads (`cat`, `sed`). A subprocess such as a Python script can still open the file; only the sandbox (§14) stops that. `.claudeignore` has **no effect**, so use deny rules.
- **The `rtk git push` ask entries exist because of the global RTK hook.** It rewrites commands to `rtk git …`, and permission rules are checked against the rewritten command. Don't test it yourself: you never push, and the prompt would stall the session. List "ask rule not exercised" under not proven, and give the owner `! git push --dry-run` to try.
- **SessionStart output is capped.** It becomes context, up to 10,000 characters. Write it as plain facts. Imperative "SYSTEM:" wording can trip the prompt-injection defences.

Rules for every hook you add:
- **Command form (guarded launcher):**
  ```
  bash -c 'f="$(git rev-parse --show-toplevel 2>/dev/null)/.claude/hooks/<x>.sh"; [ -f "$f" ] && exec bash "$f"; exit 0'
  ```
  - **Why the repo root:** it resolves to the checkout the session is in, including a worktree. `${CLAUDE_PROJECT_DIR}` stays at the main checkout after EnterWorktree, so it would run the main checkout's copy. Hooks run in the session's current directory.
  - **If a worktree session ever reports the main checkout's state,** resolve the root from the hook input's `cwd` field instead.
  - **Why the existence check:** with `onFailure: block`, a *missing* script (for example a worktree on a branch where the hooks aren't committed yet) would otherwise block **every** Bash call. The launcher is tested; see Appendix D.
- **What the push guard is and isn't:** a speed bump for honest mistakes, not a security boundary.
  - It ignores quoted text, so a commit message that mentions a push doesn't trigger it.
  - It does scan `bash -c` / `eval` payloads.
  - It fails closed on any internal error.
  - **It does not see:** git aliases, a push inside a script file, `gh pr merge`, or GitHub MCP tools.
  - **It can't stop** Claude from adding a `RESOLVED:` line itself. That is why the `ask` rule on push stays as the human check.
- **Hooks match the command text,** so tolerate these variants: `/usr/bin/git`, `git -C dir`, `git -c k=v`, `command git`, `rtk git`, `cd x && git …`.
- **Ship a test.** `.claude/hooks/hooks.test.sh` drives every hook with real JSON payloads in a throwaway repo (Appendix C). Re-run it after any hook change.
- **Keep personal overrides local.** They go in `.claude/settings.local.json`, which is gitignored. Never put tokens in any `env` block.
- **Format on edit (recommended).** Add a PostToolUse `Edit|Write` hook that runs the stack's formatter on the changed file only: prettier or biome for JS/TS. Python is already covered, because the global layer runs ruff on `.py`. Keep it fast, and it always exits 0.
- **Stop gate (optional).** A Stop hook can run `<pm> verify` before Claude ends its turn.
  - It must check `stop_hook_active` in its input to avoid loops. Claude Code overrides the hook after 8 consecutive blocks.
  - Add it only if typecheck takes under about 20 s. Otherwise rely on the CLAUDE.md line "run verify before saying done".

---

## 8. `.planning/`: state, blockers, ledger

Track the folder in git (`git add -f` if it is gitignored) and never delete it. Exclude it from deploy bundles (for example `.vercelignore` or `.dockerignore`).

**`STATE.md`** (template E.3). The hooks read two parts of it:

- **`## Current focus`.** 5–15 lines: branch, what is in flight, next 3 steps, owner decisions pending.
  - Rewrite it, don't append, at the end of every work session.
  - `restore-state.sh` re-injects this section (up to the next `## ` heading, at most 4,000 bytes) after `/compact`, and git carries it to every worktree and machine.
  - Keep `### Blockers/Concerns` under a *different* `##` section, as E.3 does.
- **`### Blockers/Concerns`.** This is the only section the push guard reads.
  - **What counts as a blocker:** a top-level bullet whose text contains `push blocker` or `do NOT git push` (case-insensitive).
  - **How it is cleared:** a continuation line *inside that bullet* that starts with `RESOLVED:`. Use the form `RESOLVED: YYYY-MM-DD — <how it was verified>`.
  - Never delete entries; they are the audit trail.
  - A concern that is not a gate must avoid those phrases. That includes a "no push blockers" bullet.

```markdown
### Blockers/Concerns
- **Migration 0007 push blocker (2026-10-19, from `feat/orders-261019`): do NOT git push origin main until 0007 is
  applied to dev and prod.** Why: the new code reads `orders.status_v2`.
  Progress 2026-10-20: applied to dev.
  RESOLVED: 2026-10-21 — applied to prod, verified with `<command>`.
```

**`DEFERRED-BY-BLOCKER.md`.** When the global review gate blocks a push, its message tells the session to register real-but-out-of-scope findings here, under Section F. The gate never reads the file; it only gives the instruction a home.

```markdown
# Deferred by blocker
Findings a review gate surfaced that were real but out of scope for the diff under review. Registered so the next
reviewer does not re-find them. Open entries first; closed ones collapse to one line under **Closed**.

## Section F — review-gate deferrals

### SWEEP-F1 — <one-line title>
**Raised:** YYYY-MM-DD, <branch / context>. **Status:** OPEN
<facts, risk, `file::symbol`>
**Clearing condition:** <what resolves it>

**Closed**
- SWEEP-F0 — <title> — closed YYYY-MM-DD (<commit>)
```

The owner's rule still applies: fix every finding the branch itself introduced. Section F holds only pre-existing or genuinely out-of-scope items.

---

## 9. Docs set

**Create now**
- **`docs/ARCHITECTURE.md`**, matklad style, one page or less (template E.13):
  - a bird's-eye view;
  - the coarse modules and how they talk;
  - layer rules;
  - invariants stated as absences ("the domain never imports the UI");
  - a "where to add X" checklist.

  Only stable things go here. It is reviewed twice a year, not synced every commit.
- **`CONTEXT.md`.** A glossary of 10–30 domain terms, each with a definition and its forbidden synonyms. Grow it with the global `domain-modeling` skill. It stops agents inventing synonyms that make schema, UI and code drift apart. In an empty repo, leave a `TBD` table.
- **`docs/ADR/README.md` plus `0001-<first decision>.md`** (template E.5).
  - **Format:** Nygard: Status / Context / Decision ("We will…") / Consequences.
  - **Immutable:** an accepted ADR is never edited. A reversal is a new ADR that supersedes it; only the old ADR's Status line changes.
  - **When:** write one only for hard-to-reverse choices: database, auth provider, hosting, payments, AI vendor, module boundaries.
  - **What ADR-0001 records:** the module boundaries, if you added a boundary tool (§11.1). Otherwise, the stack as found, with "Enforced by: none yet". Never claim enforcement that doesn't exist.
- **`docs/INCIDENTS.md`**, numbered (template E.6). Each entry ends with the guard that now makes the incident impossible (a test, lint rule or hook), plus a rule bullet that cites the incident number. Start it with the template and no entries.
- **`docs/DECISIONS.md`** (optional). A short durable "why" log for decisions too small for an ADR.
  - Each heading states the rule as a sentence, then: why, the incident behind it, and the rules that follow.
  - Keep it under about 500 lines, and promote stable entries into ADRs or rules. Lesson learned: one such log grew to several hundred KB and stopped being readable.

**Create when the trigger appears**

| Doc | Trigger |
|---|---|
| `DEPLOYMENT.md` | First deploy |
| `ENV-FLAGS.md` | First behaviour flag (record default, why, rollback) |
| `SECRETS.md` | First production secret (one rotation runbook per secret) |
| `CI.md` | First workflow beyond the basic one |
| `API.md` | First public endpoint or server action |
| `DATABASE.md` | First table (columns, indexes, and the migration that added each) |
| `runbooks/` | Deploy, rollback and restore, each when it first becomes real |
| `docs/checkups/` | Once live: the monthly product check-up playbook |

---
## 10. Memory and recall

**Where each kind of knowledge goes.** This table decides whether Claude recalls the right fact at the right time:

| Knowledge | Home | Why there |
|---|---|---|
| Commands, invariants, always/never rules, owner working rules | `CLAUDE.md` | Loaded every session, re-read after `/compact` |
| Path-specific landmines | `.claude/rules/*.md` | Loads exactly when matching files are touched |
| Where things are | `RECONSTRUCTION.md` + LSP | Small, stamped, live |
| Current task, next steps, blockers | `.planning/STATE.md` | Travels via git to every worktree and machine; re-injected after compaction |
| Big decisions and their reasons | `docs/ADR/`, `docs/DECISIONS.md` | Versioned and reviewable |
| Incidents | `docs/INCIDENTS.md` + a rule + a guard | Evidence in the repo |
| Owner preferences, corrections, pointers to external systems (dashboards, accounts) | Auto memory | Can't be derived from the repo |
| Handing work to a new session | `/handoffprompt` (global skill) + `STATE.md` | |
| Anything git, the code or CLAUDE.md already holds (what shipped, PR numbers, file lists, conventions) | Nowhere | Duplicates rot |

**Auto memory, in short.** It is one folder per git repo, shared by all its worktrees and machine-local. At session start only the first **200 lines or 25 KB** of `MEMORY.md` load. Topic files are read on demand. Subagents don't get it, so pass them what they need in the prompt.

**Where this repo's memory folder is:**
- Use the auto-memory directory named in your own system prompt.
- Failing that, take the main worktree's absolute path, replace every character that isn't a letter or digit with `-`, and use `~/.claude/projects/<that>/memory/`.
- Creating `MEMORY.md` and `reference_docs_index.md` there is the one `~/.claude` write §0.1 allows, and only if `MEMORY.md` doesn't exist yet.

**House rules for memory in this repo** (what makes recall reliable):
- **One fact per file** (template E.7). Use the frontmatter Claude's own memory writer uses on this machine:
  - `name`: a kebab-case slug (`reference-docs-index`). Links to it are written `[[reference-docs-index]]`.
  - `description`: one line of 120–200 characters stating the fact and when it applies. This line is what recall keys on.
  - `metadata.type`: `user`, `feedback`, `project` or `reference`.

  The body holds the fact, then `**Why:**`, then `**How to apply:**`, then `[[links]]` to related memories.
- **File names:** `<type>_<slug>.md`. Add a `_<YYMMDD>` suffix only for time-bound project notes. Never put status words (shipped, live) in a name.
- **`MEMORY.md` is an index only:**
  - one line per memory: `- [Title](file.md) — hook`;
  - **hard limit: 100 lines and 12 KB**, half of what loads;
  - **prune when it passes 80 lines**;
  - sections: Working rules · Active (at most 10, each saying what is owed) · Landmines · Reference · History.
- **Topic files: 3 KB or less.** Anything bigger belongs in `docs/`.
- **Use absolute dates only.** Before relying on a remembered path, flag or function, verify that it still exists.
- **Pruning:**
  - At the end of each work batch, move shipped or closed Active items to History (`index_history.md`, created when the first item closes) and rewrite STATE.md `## Current focus`.
  - Monthly, as part of the check-up:
    - merge overlapping memories;
    - delete what git or docs now hold;
    - close Active items older than 21 days.
- **Day 1:** create `MEMORY.md` with the five section headers plus one Reference line pointing to `reference_docs_index.md`, and create that file (both in E.7). **Do not copy memories from other projects.**

---

## 11. Foundation best practice (Phase B)

**How to use this section:**
1. For every item, mark it **present**, **partial** or **missing**. Write the list to `.planning/agent-setup.md` under `## Phase B gap report`; the §17 report shows only the top gaps.
2. Build the **DAY-1** items that §0.3 allows: new files, merges, and config that acts only after merge. The doc-only items (ARCHITECTURE, CONTEXT, ADR folder, spec template) always qualify.
3. List everything else in the §17 question block: new dependencies, tool installs, git hooks, GitHub settings, runtime changes, paid services and strict-mode flips.
4. In a brand-new repo with a known stack, the same split applies. Scaffold files freely, but installs and services wait for the owner.

Tags: **DAY-1** means set up now. **WHEN-NEEDED** means add it when the named trigger appears. **SKIP-FOR-SOLO** means overkill for one founder plus agents.

The principle behind the list: **agents don't remember prose, so make invariants mechanical.** Prefer a lint rule, test or hook over a sentence in a doc.

### 11.1 Architecture
- **DAY-1: monolith first.** One repo, one deployable, one database, with modules inside. Split only when a boundary is proven by an independent scaling or deploy need.
- **DAY-1: vertical slices plus a pure core.**
  - Organise by feature or module.
  - Put business rules in plain functions that import no framework code.
  - Put I/O (database, payments, AI providers, email) behind thin adapters, and only at real external boundaries.
- **DAY-1: boundaries enforced by tooling.**
  - **Rules:** 3–5, for example: UI → app → domain and never the reverse; modules import each other only through a public index; no cycles.
  - **Tools:** dependency-cruiser or eslint-plugin-boundaries (TS), import-linter (Python), ArchUnit (JVM). The tool is a new dependency, so ask first.
  - **Rollout:** existing violations start at `warn` so CI is green; new ones fail. Record the rules as ADR-0001.
- **DAY-1: small files.** A soft cap of about 300 lines via lint. The global layer already refuses to Write a file over 800 lines.
- **DAY-1: glossary** (`CONTEXT.md`, §9).
- **SKIP-FOR-SOLO:** microservices, full hexagonal ceremony, arc42. Borrow only arc42's "quality goals" and "risks".

### 11.2 Quality
- **DAY-1: one `verify` command.** It runs typecheck, lint, format check and boundary check. CI runs the same command plus tests. Name it in CLAUDE.md. **This is the #1 item: it lets the agent check its own work.**
  - Wiring existing tools into one script is allowed.
  - Adding new tools needs the owner.
- **DAY-1: strict typecheck, linter and formatter.** TS `strict`, or pyright/mypy strict. On an existing codebase, report how many errors strict mode would raise instead of flipping it.
- **DAY-1: testing trophy.**
  - Mostly integration tests at module boundaries, against a real local database.
  - Unit tests for non-trivial pure logic.
  - A handful of end-to-end tests on the money and onboarding journeys.
  - Zero tolerance for flaky tests.
  
  The owner's "no new tests unless asked" rule governs what agents add during tasks. The project still needs the harness, and CI still runs the suite.
- **DAY-1: test-integrity guard.** CI flags deleted, skipped or `.only` tests and a shrinking test count. Kent Beck lists disabling or deleting tests as one sign an AI collaborator is cheating, next to loops and unrequested features.
- **DAY-1: census tests (architectural fitness functions).** Start with 2–3 and add one per incident (template E.10). Each test walks the whole tree and asserts an invariant for every member. Examples: every route declares an auth policy; every env read goes through the config module; every analytics event is in the registry. Shape:
  - Walk the filesystem rather than `git grep`, so new uncommitted files count.
  - Assert non-vacuity (`files.length > N`) so an empty walk cannot pass.
  - Keep an allow-list, and fail on stale allow-list entries.
  - Make the failure message say how to fix it.
- **DAY-1: CI with a single required check, `ci-success`** (template E.9).
  - An aggregate job with `if: always()`.
  - A `changes` job that decides what runs. Never use workflow-level `paths-ignore`: a path-skipped workflow leaves a required check pending forever.
  - The ruleset stored as JSON in the repo. Applying it is a GitHub setting, so ask first.
- **DAY-1: trunk-based development with short-lived branches and PRs,** even solo. Use conventional commits: `feat:`, `fix:`, `chore:`, `docs:`.
- **DAY-1: pre-commit:** lint-staged (lint and format staged files) plus gitleaks (E.12). Enabling git hooks needs the owner, because a missing binary breaks every commit.
- **WHEN-NEEDED:**
  - property tests (fast-check, Hypothesis) as soon as pricing, quota or permission logic exists;
  - golden/snapshot tests and evals for prompts at the first LLM feature;
  - SemVer and a changelog at the first public API or SDK.
- **SKIP-FOR-SOLO:** Pact contract tests in a monolith; mandatory human reviewers (the agent review plus CI is the review).

### 11.3 Operations
- **DAY-1: config validated at boot** (zod, pydantic-settings).
  - One schema parses every env var and crashes with a clear message.
  - Nothing else reads env directly; enforce that with a census test.
  - Commit `.env.example`.
  - *In an existing app this changes runtime behaviour, so ask first.*
- **DAY-1: dev/prod isolation plus a production guard** (template E.11).
  - Separate databases and credentials.
  - Tests, bulk scripts and agent-run migrations refuse the production endpoint: a denylist of the prod host or id, checked in test setup and in scripts.
  - The escape hatch `ALLOW_PRODUCTION_DB=1` never lives in `.env`, a shell profile or CI.
  
  This is real: in July 2025 a Replit agent deleted a live production database during a code freeze, and Replit then added automatic dev/prod separation.
- **DAY-1: agents get dev keys only,** and every paid API key has a spend cap.
- **DAY-1: backups on,** plus a restore drill before real customer data, then quarterly. This is an owner action on the database host.
- **DAY-1: migrations discipline.**
  - Never edit an applied migration; a CI check enforces it.
  - Use expand → migrate → contract for breaking changes.
  - Apply every migration to every environment.
- **DAY-1: idempotency on every money or side-effect path.** Use idempotency keys and dedupe webhooks by event id.
- **DAY-1: kill switches** for expensive or risky features. An env flag is enough. Document each flag with its default and rollback.
- **DAY-1: observability.** Structured JSON logs carrying a request id, an error tracker, and a `/health` endpoint. *The error tracker is an external service, so ask first.*
- **DAY-1 on auth and AI endpoints: rate limiting.**
- **WHEN-NEEDED:**

  | Add | Trigger |
  |---|---|
  | SLOs | First paying customers |
  | Audit log | First payments or multi-user data |
  | Distributed tracing | A second service |
  | Feature-flag platform | First real rollout or A/B test |

### 11.4 Security
- **DAY-1: secrets.**
  - `.env*` is gitignored.
  - Real secrets live in the host's secret store.
  - Never put them in `settings.json` `env` or in committed files.
  - Claude `permissions.deny` covers the real env files (§7).
- **DAY-1: gitleaks** as a pre-commit hook (`rev: v8.30.1`, the latest tag on 2026-10-19; `pre-commit autoupdate` moves it forward). The hook id is `gitleaks`. For a manual scan, run `gitleaks git --staged`. Add GitHub secret scanning and push protection where available.
- **DAY-1: GitHub Actions hardening.**
  - The workflow-level `permissions:` is read-only by default, granted per job.
  - Third-party actions are pinned to full commit SHAs.
  - Dependabot runs with an explicit `cooldown` (E.12). github.com already waits 3 days by default, and cooldown never delays security updates.
- **DAY-1: a ruleset on the default branch:** PR required, `ci-success` required, no force-push, no deletion. Write the JSON; applying it is an owner action.
- **DAY-1, if the product calls an LLM: OWASP Top 10 for LLM apps.**
  - Treat model output as untrusted input.
  - Give tools least privilege, and cap spend.
  - Never give one agent all three of: private data, untrusted content and an outbound channel. This is the "lethal trifecta".
- **Before launch:** walk **OWASP ASVS Level 1** with an agent (auth, session, access control, input and output handling).
- **WHEN-NEEDED:** OpenSSF Scorecard once the repo is public; SBOM and a licence check for the first enterprise buyer.
- **SKIP-FOR-SOLO:** SLSA provenance, fuzzing.

### 11.5 Process
- **DAY-1: a one-page spec before any multi-file feature** (`docs/specs/<slug>.md`, template E.8).
  - Contents: goal, users, out of scope, interfaces, acceptance checks, and an end-to-end verification step.
  - Have Claude interview the owner with AskUserQuestion to fill it, then build it in a fresh session.
  - Skip heavy spec frameworks.
- **DAY-1: `STATE.md` current focus** (§8).
- **DAY-1: every incident gets a guard** (§9).
- **DAY-1: an analytics event registry,** if anything is tracked. Typed event names and properties, plus a test that rejects PII-looking properties.
- **DAY-1: cost caps** on every provider, and a monthly cost line in the check-up.
- **DAY-1: a monthly check-up** on the calendar. It covers:
  - dependency age;
  - doc staleness (the bearings line);
  - rule-vs-reality drift (`/doctor prompt-audit`);
  - dead code;
  - flag expiry;
  - cost trend;
  - memory pruning (§10).

### 11.6 Top 15 — the order to do things in
1. One `verify` command, mirrored by CI and required on the default branch.
2. Dev/prod isolation, plus production guards on tests, scripts and migrations.
3. Strict typecheck, lint and formatter inside `verify`.
4. A config schema validated at boot; agents on dev keys; spend caps.
5. Module-boundary rules enforced by tooling (3–5 rules), recorded as ADR-0001.
6. Backups plus a restore drill.
7. Migrations discipline, with a CI check.
8. A ruleset, short-lived branches and PRs, read-only workflow tokens, SHA-pinned actions.
9. gitleaks, `.env` hygiene and Claude deny rules.
10. `ARCHITECTURE.md`, `CONTEXT.md` and the ADR folder.
11. A pure domain core, integration tests and a test-deletion guard.
12. Structured logs with request ids, error tracking and a health endpoint.
13. A dependency policy: lockfile, version pin (`mise.toml` or `.nvmrc`), Dependabot or Renovate with cooldown.
14. Idempotency on money and side-effect paths.
15. A spec before every build, `STATE.md`, and numbered incidents that each end in a guard.

---

## 12. The global layer already active (don't copy it, supply what it needs)

**This table describes the author's machine.** On a machine without this layer, skip the table: nothing in the kit
depends on it, and §2 step 8 reports what is missing.

| Global piece | What it does in this repo | What the repo must provide |
|---|---|---|
| **`/code-review` push gate** (`code-review-gate/lib/mark/clear.sh`) | Denies `git push` until `/code-review` at effort high or above has covered every changed code file since the last landed push. Sensitive paths (migrations, middleware, auth/reauth, consent, rls, policy, grants, pin) also owe `/security-review`. A 4-hour fix window follows each review round. | **It only gates `ts tsx js jsx mjs cjs sql css`.** If this repo's code is something else, the gate never fires: drop that line from CLAUDE.md and report it. Don't edit the global hook. Its refusal message points at `.planning/DEFERRED-BY-BLOCKER.md` Section F, so that file must exist. |
| **Citation-drift gate** | Blocks a `git commit` that adds a `path:LINE` citation in a **code file** (same extensions; Markdown isn't scanned) when the same diff shifts the cited file at or above that line | Cite symbols, not lines, in code comments. A repo may ship its own `scripts/check-citation-drift.mjs`; the global gate then steps aside and the repo runs its copy from its own pre-commit. |
| **block-no-verify** | Refuses any command containing `--no-verify` | Pre-commit hooks that pass honestly |
| **block-large-files** | Refuses a Write of more than 800 lines | Small modules. Write long docs in parts. |
| **RTK proxy** | Rewrites shell commands (`git status` → `rtk git status`) to save tokens | Hooks and ask rules that tolerate an `rtk ` prefix. If plain `git` misbehaves inside a worktree, use `/usr/bin/git`. |
| **context-ceiling + resume-after-compact** | Warns near the context limit and tells Claude to resume after compaction | `## Current focus` in STATE.md, which `restore-state.sh` re-injects |
| **Status line** | Shows the model, a context bar, the folder and the **live** git branch | Use EnterWorktree, so the branch shown is the branch being edited |
| **Owner output style** | Short, result-first replies for a non-engineer | Nothing. Don't set `outputStyle` in project settings. |
| **Global skills** | bearings, handoffprompt, reviewers-run-check-changes-and-find-errors, deep-qa-sweep, karpathy-guidelines, steal-this-code, grilling, domain-modeling, context-budget, graphify | Mention one in CLAUDE.md only if a project workflow depends on it. Don't run `bearings` end to end (§5). |
| **Subagent table** (`~/.claude/CLAUDE.md`) | Haiku, Sonnet and Opus 5.5 with allowed efforts; never Fable | Project agents in `.claude/agents/` set `model` and `effort` within that table |

**Project-specific skills and agents.** Create them only for workflows this project repeats, such as a release checklist or a translation pipeline.
- **SKILL.md frontmatter:**
  - `name`;
  - `description`, including the phrases a user would say to trigger it;
  - optionally `when_to_use`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `model`, `effort`, `context: fork`, `paths`.
- **Skill design:** keep SKILL.md under 500 lines. Deterministic work goes in scripts the skill calls. Verification goes in a checker script. Agents supply the judgement; scripts and CI verify it.
- **Agent frontmatter:**
  - `name`;
  - `description` ("Use ONLY for…; never for…");
  - `model`, `effort`, `tools`;
  - optionally `isolation: worktree` and `memory: project`. Field names must match exactly, because unknown fields are silently ignored.

---

## 13. Git, branches, worktrees, push

- **Protect the default branch** with a ruleset (write the JSON; the owner applies it). Production deploys from a separate branch or tag where the host allows it, so a push to default means a preview and promotion is always explicit.
- **One branch per change:** `feat|fix|chore/<slug>-<YYMMDD>`. Whenever another session may be active, isolate on a branch regardless of size.
- **PR body:** what it ships, how it was verified, which environments got its migrations, and how to roll back.
- **Pre-push:** the build runs only as the last step before promoting to production.
- **Worktrees for parallel work:**
  - Use `EnterWorktree`, and add `.claude/worktrees/` to `.gitignore`.
  - A `.worktreeinclude` file (gitignore syntax, for example a `.env.local` line) makes worktrees that Claude creates copy those gitignored files. It doesn't apply to a manual `git worktree add`.
  - A fresh worktree needs its dependencies installed first.
- **The stash is shared** between all worktrees and sessions. Never use a bare `git stash` / `git stash pop`. Use a WIP commit, or `git stash push -u -m "<unique tag>"` and then `git stash apply <sha>`.
- **Never commit straight to production.** Never force-push a shared branch.

---

## 14. Optional extras (add when the trigger fits)

| Extra | Trigger | Note |
|---|---|---|
| **AGENTS.md** | Another tool is used (Codex, Cursor, Copilot, Gemini CLI) | Put tool-neutral content in `AGENTS.md`. Start `CLAUDE.md` with `@AGENTS.md`, then a `## Claude Code` section. Claude Code reads AGENTS.md on its own only when no CLAUDE.md exists. Prefer the import over a symlink. |
| **Stop-hook verify gate** | Typecheck runs under about 20 s | §7 |
| **CODEOWNERS** on `.claude/`, `CLAUDE.md`, `AGENTS.md`, `.mcp.json` | A second person gets write access | |
| **`.gitattributes`** `.planning/codebase/** linguist-generated=true` | Generated docs clutter PR diffs | |
| **Rule debugging** | A rule seems ignored | `/doctor prompt-audit` (2.1.283+) finds stale or conflicting instructions. A temporary `InstructionsLoaded` hook in `.claude/settings.local.json` that logs its stdin shows which rules loaded and why. |
| **Sandbox** (`sandbox.enabled`, `filesystem.denyRead`, `network.allowedDomains`) | Unattended runs | The only thing that stops a subprocess from reading secrets |
| **Headless or CI Claude** | Running `claude -p` on a repo you don't trust | `claude -p` runs a repo's hooks and `.mcp.json` **without a trust prompt**. `--bare` skips hooks, skills, plugins, MCP, memory and CLAUDE.md, and needs an API key. |
| **`.mcp.json`** | Project MCP servers | Reference secrets as `${VAR}`; never inline them |
| **Long autonomous runs** | Multi-hour agent runs | A `feature_list.json` in which agents may only flip `passes`, a progress log, and an `init.sh` that starts the app |

---

## 15. Do NOT copy from another project

This structure came out of one production repo, and the owner may have others set up the same way. When you can see
such a repo, copy its patterns and never its facts.

- **Product facts.** Do not copy any of these:
  - **Infrastructure:** database, hosting and error-tracker project ids or settings.
  - **AI models:** model ids and their pricing.
  - **Commerce:** the payment-provider setup, credit or billing ledgers.
  - **i18n:** the locales and any translation pipeline.
  - **Design and UI:** design-tool rules, component and redesign rules.
  - **Bot protection and analytics:** vendor config, the event taxonomy.
  - **Framework specifics:** dependency pins and framework invariants. If this repo uses the same framework, re-derive these from this repo instead.
- **Its history.** Memory files, incidents, ADR content, STATE.md history and DEFERRED ledger entries.
- **Its decision log.** Keep this repo's decision log short.
- **Its tooling layer.** Workflow tooling and workspace hook blocks that belong to that repo.

**Copy the patterns:** the section order, the stamps, the blocker format, the ledger format, the census-test shape, the single CI check, the prod-DB guard, and "every incident gets a guard".

---

## 16. Done when

Keep this checklist's progress in `.planning/agent-setup.md`.

- [ ] **Stack research** is recorded in `.planning/agent-setup.md` with a source (URL or Context7 id) per fact, and every stack-specific command in CLAUDE.md was confirmed there or by running it.
- [ ] **CLAUDE.md** is 200 lines or fewer and has: Orientation, Commands (including `verify`), Working rules (only if the global CLAUDE.md lacks them), Invariants, Branch/push, Docs index, Compaction line. Every new fact is verified; relocated text has pointers.
- [ ] **`RECONSTRUCTION.md`** is 100 lines or fewer, committed, stamped (`bash .claude/hooks/bearings-stamp.sh`), and the stamp is committed.
- [ ] **`.claude/bearings.json` `scopedPaths`** are right: `git ls-files -- <dir> | head -1` prints a file for each one.
- [ ] **`.planning/codebase/`** has 7 stamped docs, or is explicitly deferred for a tiny or empty repo.
- [ ] **The bearings check passes:** after staging the new files, `bash .claude/hooks/bearings-check.sh` prints `Orientation: … current` and `Rules: N …, 0 dead glob(s)`, and exits 0.
- [ ] **`.claude/rules/`** has one file per risky area plus `docs-sync.md`, and every glob matches a tracked file.
- [ ] **`.claude/settings.json`** parses (`python3 -m json.tool .claude/settings.json`), and `bash .claude/hooks/hooks.test.sh` ends with `== summary: <N> passed, 0 failed`.
- [ ] **`.claude/guard-push.json`** lists this repo's default and production branches.
- [ ] **The push guard is proven on the real file**, after STATE.md is committed:
  1. Add a temporary blocker bullet under `### Blockers/Concerns`.
  2. Run `echo '{"tool_name":"Bash","tool_input":{"command":"git push origin <default branch>"}}' | bash .claude/hooks/require-pr-on-blocker.sh; echo $?`. It must print 2.
  3. Add a `RESOLVED:` line inside the bullet and run the same command. It must print 0.
  4. Restore the file: `git checkout -- .planning/STATE.md`.
- [ ] **`.planning/STATE.md`** (with Current focus and Blockers/Concerns), **`.planning/DEFERRED-BY-BLOCKER.md`** (with Section F) and **`.planning/agent-setup.md`** exist and are tracked.
- [ ] **Docs:** `docs/ARCHITECTURE.md`, `CONTEXT.md`, `docs/ADR/README.md` + `0001`, and `docs/INCIDENTS.md` (template only).
- [ ] **The `verify` command** exists and passes, or its failures are listed honestly.
- [ ] **`MEMORY.md`** is created (headers plus the docs-index entry), or left alone if it already existed.
- [ ] **Owner actions are listed in the report:** the LSP plugin install, tool installs, git hooks, the ruleset, services.
- [ ] **Commits are grouped:**
  1. rules;
  2. hooks, settings and hook tests;
  3. CLAUDE.md, the map, `.planning/` and docs;
  4. the generated deep-dives;
  5. the Phase B items.

  Nothing pushed.

---

## 17. Report back to the owner

Plain words, result first, 25 lines or fewer:
1. **One line on what is now true.** Example: "This repo now has the full agent structure on branch `chore/agent-setup-261019`; nothing pushed."
2. **The stack found and where its facts came from** (Context7 or the docs URL), then **what loads every session** (CLAUDE.md line count, number of rules) and what loads on demand.
3. **The condensed mapping table:** created / adapted / skipped, and why. The full table is in `.planning/agent-setup.md`.
4. **Phase B:** what you built, and the top missing items ranked by the Top 15. The full gap report is in `.planning/agent-setup.md`.
5. **Proven vs not proven:** the hook-test summary line, the `verify` result, and anything you could not check.
6. **One batched question block.** Each decision with your recommendation first. Typical ones:
   - push or open a PR;
   - the owner actions (plugin, tools, git hooks, ruleset);
   - paid services;
   - runtime-behaviour changes;
   - stack/product, for an empty repo;
   - global-layer gaps, for example the review gate not covering this repo's language.

---

## Appendices A–D — the kit (hook scripts and settings)

The scripts are not printed in this document. They live in the kit folder next to it, `kit/claude/`, and are
**copied, never retyped** (§0.0 says where the kit is). Nothing below overwrites an existing file.

| Appendix | Kit file | Installed as | What it is |
|---|---|---|---|
| A | `hooks/bearings-check.sh` | `.claude/hooks/bearings-check.sh` | SessionStart freshness guard for the map, the deep-dives and dead rule globs (§5). Always exits 0. |
| A | `hooks/bearings-stamp.sh` | `.claude/hooks/bearings-stamp.sh` | Stamps the map with the current commit and date. Needs a `HEAD` commit. |
| A | `bearings.json` | `.claude/bearings.json` | Freshness config. Set `scopedPaths` to this repo's real source dirs (`["src"]`, `["app","lib"]`); each must contain tracked files. |
| B | `hooks/require-pr-on-blocker.sh` | `.claude/hooks/require-pr-on-blocker.sh` | The push guard (§7, §8). Its header comment documents the detection rules and the known gaps. |
| B | `guard-push.json` | `.claude/guard-push.json` | Push-guard config. Set `protectedBranches` to this repo's default and production branch names (shell globs allowed, for example `release/*`). Every key is optional. |
| C | `hooks/restore-state.sh` | `.claude/hooks/restore-state.sh` | Re-injects STATE.md → `## Current focus` after `/compact`. Always exits 0. |
| C | `hooks/hooks.test.sh` | `.claude/hooks/hooks.test.sh` | Self-contained tests for every hook: real JSON payloads against throwaway repos. |
| D | `settings.template.json` | `.claude/settings.json` | Deny rules for secret files, ask-before-push, and the three hook entries with the guarded launcher (§7). |

**Install:**

1. **Copy the scripts and the two config files.** Run this from the repo root:
   ```bash
   KIT=~/new-project-foundation/kit/claude      # §0.0: change this if the kit lives elsewhere
   mkdir -p .claude/hooks
   for f in bearings-check.sh bearings-stamp.sh require-pr-on-blocker.sh restore-state.sh hooks.test.sh; do
     if [ -e ".claude/hooks/$f" ]; then echo "SKIP (exists, compare by hand): $f"
     else cp "$KIT/hooks/$f" .claude/hooks/ && chmod 755 ".claude/hooks/$f" && echo "copied $f"; fi
   done
   for f in bearings.json guard-push.json; do
     if [ -e ".claude/$f" ]; then echo "SKIP (exists): $f"; else cp "$KIT/$f" .claude/ && echo "copied $f"; fi
   done
   ```
2. **Settings.** If `.claude/settings.json` doesn't exist, copy `$KIT/settings.template.json` to it. If it exists, MERGE the template's keys into it (keep its existing hooks and permissions); never overwrite. Add any other real env-file names from Discovery to `deny`.
   - Each launcher runs the repo-root copy of its script and exits 0 when the script is missing, so `onFailure: block` can never brick the session. Tested: active blocker → exit 2 (also from a subdirectory); non-push → 0; script missing → 0; not a git repo → 0.
3. **Test.** Run `bash .claude/hooks/hooks.test.sh`. On macOS also run `/bin/bash .claude/hooks/hooks.test.sh`: that is bash 3.2, the oldest supported. The last line must read `== summary: <N> passed, 0 failed`. Re-run after any hook change.

**Dependencies:** `bash`, `git`, `awk`, `grep`, `sed`, `python3`. `bearings-check` needs `python3`. The push guard uses `jq`, or falls back to `python3`, and fails closed if it has neither.

---

## Appendix E — Templates

### E.1 Rule — `.claude/rules/<area>.md`
```markdown
---
paths:
  - "<glob matching the files this rule protects>"
---
# <Area> — landmines

- **<Invariant that is expensive or silent to violate.>** <Why, in one clause; cite the incident / CI check / test that proves it.>
- <Second invariant…>
- Change here → update `<owning doc>` in the same commit.
- Deep dive: `<path to the owning doc>`.
```

### E.2 Navigation map — `RECONSTRUCTION.md` (≤100 lines)
```markdown
# RECONSTRUCTION.md — navigation map for Claude Code sessions

<!-- The two bearings-sha / bearings-date stamp lines are inserted under the title by .claude/hooks/bearings-stamp.sh. -->
Read this before navigating or editing unfamiliar parts of the codebase. It is a map with pointers, not a copy of
CLAUDE.md — rules live there (and in `.claude/rules/`), geography lives here.

## What this app is
<2–4 lines: what it does, stack in one breath, where it runs.>

## Directory map
| Dir | Purpose | Edit guidance |
|---|---|---|
| `src/…` | … | … |

## Where the core business logic lives
- `<path>` — <one line>   (10–15 entries, real backticked paths)

## Read these before editing
- <area> → `<doc or rule>`

## Deep dives (generated, stamped)
| Question | Read |
|---|---|
| Architecture / data flow | `.planning/codebase/ARCHITECTURE.md` |
| Directory layout | `.planning/codebase/STRUCTURE.md` |
| Conventions | `.planning/codebase/CONVENTIONS.md` |
| Testing | `.planning/codebase/TESTING.md` |
| Stack / deps | `.planning/codebase/STACK.md` |
| Integrations | `.planning/codebase/INTEGRATIONS.md` |
| Known debt / fragile areas | `.planning/codebase/CONCERNS.md` |

Check the `<!-- refreshed: -->` stamp; the SessionStart line says whether they are stale. Refresh: re-map the drifted
dirs only, fix drifted rows here, commit, then `bash .claude/hooks/bearings-stamp.sh` and commit the stamp.

## Landmines you can't grep your way to
1. <Geographic / historical facts that aren't visible in code.>
```

### E.3 State — `.planning/STATE.md`
```markdown
# Project State
_Last updated: YYYY-MM-DD (<branch>)_

## Current focus
- Branch: `<branch>` (<pushed? / not pushed>)
- In flight: <one line>
- Next: 1) … 2) … 3) …
- Waiting on owner: <decision>, or "nothing"

## Roadmap (next 3–5 items)
- …

## Decisions in flight
- YYYY-MM-DD — <decision> — <why> (→ ADR-000N when final)

### Blockers/Concerns
<!-- Push gates go here as top-level bullets (format: §8). Clear with a RESOLVED: line inside the bullet; never delete. -->
- (none)

## Session log (newest first, one line each, keep ≤20)
- YYYY-MM-DD — <what changed> — <commit>
```

### E.4 Ledger — `.planning/DEFERRED-BY-BLOCKER.md`
The ledger skeleton is shown in §8. Number the entries `SWEEP-F1`, `SWEEP-F2`, and so on. Never reuse a number.

### E.5 ADR — `docs/ADR/0001-<slug>.md` (+ `docs/ADR/README.md`)
```markdown
# ADR-0001 — <Title>

## Status
Accepted — YYYY-MM-DD. Enforced by `<lint rule / test path>` — or "none yet" (never claim a guard that doesn't exist).
<!-- If ever reversed: "Superseded by ADR-000N (YYYY-MM-DD)" — the only edit an accepted ADR ever gets. -->

## Context
<Forces and constraints; what goes wrong without this decision; cite incidents.>

## Decision
We will <…>.

## Consequences
<What gets easier, what gets harder, which guard enforces it, follow-ups.>
```
`docs/ADR/README.md` contains:
- the sentence "An accepted ADR is immutable. To change a decision, write a new ADR that supersedes it";
- a "write an ADR when…" list: hard-to-reverse choices only, never bug fixes or UI tweaks;
- an index table: `| # | Title | Status |`.

### E.6 Incident — `docs/INCIDENTS.md` entry
```markdown
## Incident N: <title> (<one-line cause>)
> **Read this first — the diagnostic tell.** <The symptom combination that identifies this incident.>

<Why the safety nets didn't catch it. First occurrence: YYYY-MM-DD, <commit/PR>.>

### Symptoms
### Diagnosis        <!-- numbered steps, the decisive check first -->
### Resolution
### Prevention       <!-- the guard that now makes it impossible: test / lint rule / hook path; + the rule bullet citing "Incident N" -->
```

### E.7 Memory: the day-1 files, plus an example
The folder is this repo's memory folder (§10). Create these two files on day 1, and only if `MEMORY.md` doesn't exist yet.

`reference_docs_index.md`:
```markdown
---
name: reference-docs-index
description: Where this repo's knowledge lives — CLAUDE.md (rules), RECONSTRUCTION.md (map), .planning/STATE.md (current focus), docs/ (ADRs, incidents); read before searching memory
metadata:
  type: reference
---
Repo knowledge lives in the repo, not in memory: CLAUDE.md → Documentation lists every doc and what it owns.
**Why:** memory is machine-local and starts empty per repo; the repo travels with git.
**How to apply:** for "where is X / what was decided", check CLAUDE.md, RECONSTRUCTION.md, `.planning/STATE.md` and `docs/ADR/` first.
```
`MEMORY.md` (index only; hard limit 100 lines / 12 KB, prune past 80):
```markdown
## Working rules (owner)

## Active (≤10, newest first — each says what is owed)

## Landmines

## Reference
- [Docs index](reference_docs_index.md) — where repo knowledge lives; read before searching memory.

## History
<!-- create index_history.md when the first Active item closes; one pointer line here -->
```
**An illustration only: don't create this file.** It shows the shape of a later memory, an owner correction that the repo can't hold, saved as `feedback_<slug>.md`:
```markdown
---
name: feedback-staging-before-prod
description: Owner wants every deploy to go to staging and be clicked through before production, even one-line fixes; applies to all deploy steps
metadata:
  type: feedback
---
Deploy to staging and check it in the browser before promoting to production — no exceptions for small fixes.
**Why:** the owner said so on 2026-10-19 after a one-line fix broke checkout.
**How to apply:** every deploy plan lists the staging check; never promote straight to production.
```

### E.8 Spec — `docs/specs/<slug>.md` (one page)
```markdown
# Spec — <feature> (YYYY-MM-DD)
**Goal:** <one sentence, user-visible outcome>
**Users / trigger:** <who, when>
**In scope:** <bullets>   **Out of scope:** <bullets>
**Interfaces touched:** <routes / actions / tables / events, by name>
**Acceptance checks:** 1) <observable: UI state, API response, DB row> 2) …
**End-to-end verification:** <exact steps / commands that prove it works>
**Risks / rollback:** <flag name, migration reversibility>
**Decisions needed from owner:** <list, or "none">
```

### E.9 CI — single required check (GitHub Actions excerpt)
```yaml
permissions:
  contents: read                    # read-only by default; grant more per job only where needed

jobs:
  changes:                          # decides what runs — never use workflow-level paths-ignore
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read           # paths-filter lists PR files through the API; without this it fails on private repos
    outputs:
      code: ${{ steps.filter.outputs.code }}
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - id: filter
        uses: dorny/paths-filter@<full-commit-sha>   # third-party: pin the SHA
        with:
          filters: |
            code: ['src/**', 'tests/**', '.github/**', '<lockfile>', '<package manifest>', '<tool configs>']
  verify:
    needs: changes
    if: needs.changes.outputs.code == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - run: <install command> && <pm> verify && <pm> test
  ci-success:                       # the ONLY required status check in the ruleset
    if: always()
    needs: [changes, verify]
    runs-on: ubuntu-latest
    steps:
      - uses: re-actors/alls-green@<full-commit-sha>
        with:
          jobs: ${{ toJSON(needs) }}
          allowed-skips: verify
```
**Resolving each SHA:** run `gh api repos/<owner>/<repo>/commits/<tag> --jq .sha`. If you can't resolve one, leave that workflow out and list it in the report. **Never write a SHA from memory.**

Pin every action to a full commit SHA. `pinact` can do it, and Dependabot's `github-actions` ecosystem keeps the pins fresh. Store the ruleset as JSON in `.github/rulesets/`.

### E.10 Census test (TypeScript + vitest; same shape in any language)
Use this exact example only when the repo already has one config module. Otherwise pick an invariant that holds **today** (for example "every route file exports an auth policy"), and build the allow-list from the current tree, so the test starts green.
```ts
// __tests__/env-read-census.test.ts — every env read goes through src/config.ts (validated at boot)
import { readdirSync, readFileSync, statSync, existsSync } from 'node:fs';
import { join } from 'node:path';
import { expect, test } from 'vitest';

const ROOT = 'src';
const MIN_FILES = 5; // raise as the codebase grows; keeps an empty or wrong-root walk from passing
const ALLOW = new Set(['src/config.ts', 'src/db/production-guard.ts']);
const FRAMEWORK_READS = /process\.env\.(NODE_ENV|NEXT_PUBLIC_[A-Z0-9_]+)\b/g; // framework-mandated reads are fine

function walk(dir: string): string[] {
  // Walk the filesystem, not `git ls-files`, so a new uncommitted file is counted too.
  return readdirSync(dir).flatMap((name) => {
    const p = join(dir, name);
    if (statSync(p).isDirectory()) return walk(p);
    return /\.(ts|tsx)$/.test(p) ? [p] : [];
  });
}

test('only the config module reads process.env', () => {
  const files = walk(ROOT);
  expect(files.length).toBeGreaterThan(MIN_FILES); // non-vacuity: an empty walk must not pass
  const offenders = files.filter(
    (f) => !ALLOW.has(f) && /process\.env\b/.test(readFileSync(f, 'utf8').replace(FRAMEWORK_READS, '')),
  );
  expect(offenders, 'Read env through src/config.ts (validated at boot), not process.env').toEqual([]);
});

test('the allow-list has no stale entries', () => {
  for (const f of ALLOW) expect(existsSync(f), `${f} is allow-listed but no longer exists`).toBe(true);
});
```

### E.11 Production-database guard
```ts
// src/db/production-guard.ts — imported by the test setup and by bulk/maintenance scripts, never by app runtime.
const PRODUCTION_MARKERS = ['<prod db host or project id>']; // a denylist of the production endpoint only

export function assertNotProductionDatabase(url: string = process.env.DATABASE_URL ?? ''): void {
  // An unfilled placeholder would make the guard silently inert — fail loudly instead.
  if (PRODUCTION_MARKERS.some((m) => m.includes('<'))) throw new Error('production-guard: set PRODUCTION_MARKERS');
  // Deliberate-ops escape hatch. Never set it in .env, a shell profile or CI.
  if (process.env.ALLOW_PRODUCTION_DB === '1') return;
  if (PRODUCTION_MARKERS.some((m) => url.includes(m))) {
    throw new Error('Refusing to run against the PRODUCTION database. Point DATABASE_URL at the dev database.');
  }
}
```
**When to wire it in:** call it at the top of the test setup file and of every script that writes in bulk. Do this only when the production marker is known from docs or config (never by connecting). Otherwise list it as an owner action, because an unfilled marker makes the test setup throw. Record it in CLAUDE.md under "Environments & data".

### E.12 Dependabot + secret scanning
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"            # or pip, gomod, cargo, …
    directory: "/"
    schedule:
      interval: "weekly"
    cooldown:
      default-days: 7                   # version updates only; security updates are never delayed
    groups:
      minor-and-patch:
        patterns: ["*"]
        update-types: ["minor", "patch"]
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    cooldown:
      default-days: 7
```
**gitleaks:**
- Install the binary (`brew install gitleaks`).
- With the pre-commit framework, add this to `.pre-commit-config.yaml`:
  ```yaml
  repos:
    - repo: https://github.com/gitleaks/gitleaks
      rev: v8.30.1        # latest tag on 2026-10-19; `pre-commit autoupdate` moves it forward
      hooks:
        - id: gitleaks
  ```
- With husky, add this line to `.husky/pre-commit`, after `npx lint-staged`:
  ```
  gitleaks git --pre-commit --redact --staged --verbose
  ```
- Allow-list false positives in `.gitleaks.toml` or with a `gitleaks:allow` comment.

### E.13 `docs/ARCHITECTURE.md` and `CONTEXT.md` skeletons
```markdown
# Architecture
## Bird's-eye view            <!-- what comes in, what goes out, in 5 lines -->
## Modules                    <!-- coarse codemap: name — responsibility — entry file -->
## Layer rules                <!-- enforced by <tool>, see ADR-0001 -->
## Invariants                 <!-- stated as absences: "domain/ never imports ui/" -->
## Cross-cutting              <!-- config, errors, logging, auth -->
## Where to add X             <!-- new endpoint / table / page / job: which files, in what order -->
```
```markdown
# Domain glossary
| Term | Meaning | Not to be confused with / forbidden synonyms |
|---|---|---|
| <Project> | <definition> | never "job", never "task" |
```

---

## Sources (checked 2026-10-19)

**Claude Code docs:**
- Memory, rules and AGENTS.md: https://code.claude.com/docs/en/memory
- Hooks, exit codes and `onFailure`: https://code.claude.com/docs/en/hooks
- Hooks guide: https://code.claude.com/docs/en/hooks-guide
- Permissions and modes: https://code.claude.com/docs/en/permissions and https://code.claude.com/docs/en/permission-modes
- Settings: https://code.claude.com/docs/en/settings
- Skills and subagents: https://code.claude.com/docs/en/skills and https://code.claude.com/docs/en/sub-agents
- Worktrees: https://code.claude.com/docs/en/worktrees
- Headless and `--bare`: https://code.claude.com/docs/en/headless
- Best practices: https://code.claude.com/docs/en/best-practices

**Anthropic engineering posts:**
- https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

**Architecture:**
- Fowler, MonolithFirst: https://martinfowler.com/bliki/MonolithFirst.html
- Fowler, Feature Toggles: https://martinfowler.com/articles/feature-toggles.html
- Fowler, ParallelChange: https://martinfowler.com/bliki/ParallelChange.html
- matklad, ARCHITECTURE.md: https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html
- matklad, basic things: https://matklad.github.io/2024/03/22/basic-things.html
- Nygard ADRs: https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- Thoughtworks, fitness functions: https://www.thoughtworks.com/radar/techniques/architectural-fitness-function

**Engineering practice:**
- Twelve-factor config: https://12factor.net/config
- Google SRE, postmortems: https://sre.google/sre-book/postmortem-culture/
- DORA capabilities: https://dora.dev/capabilities/
- Kent Beck, augmented coding: https://newsletter.kentbeck.com/p/augmented-coding-beyond-the-vibes
- Hamel Husain, evals: https://hamel.dev/blog/posts/evals/

**Security and supply chain:**
- OWASP LLM Top 10: https://genai.owasp.org/llm-top-10/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- Simon Willison, lethal trifecta: https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- gitleaks: https://github.com/gitleaks/gitleaks
- Dependabot options: https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference
- GitHub Actions hardening: https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
