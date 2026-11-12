# Claude Code project foundation

A guide and a small kit that set up any repo, new or existing, so Claude Code can work in it for years without
re-deriving the architecture every session, breaking hard-won invariants, or losing the thread across sessions,
compactions and worktrees.

You paste one prompt into Claude Code. The agent reads the guide, detects your stack, builds the structure on a
branch, and stops with one batch of questions. It never pushes.

## What a repo gets

- **A short, always-loaded `CLAUDE.md`** (200 lines or fewer) and a 100-line navigation map, stamped so a
  session-start line says when it has gone stale.
- **Path-scoped rules** in `.claude/rules/` that load only when matching files are touched.
- **Three hooks, with tests:** a push guard that blocks a push to a protected branch while an unresolved blocker
  is on file, a restore-after-compaction hook, and the freshness check for the map.
- **Ask-before-push and deny rules for secret files** in `.claude/settings.json`.
- **A planning state file** (`.planning/STATE.md`) that travels with git and is re-injected after `/compact`.
- **A docs set** (architecture page, glossary, ADRs, incidents) and a memory layout that recalls well.
- **A gap report** against a foundation checklist: one `verify` command, dev/prod isolation, CI with a single
  required check, secret scanning and more.

## Requirements

- Claude Code 2.1.295 or later. On an older version a crashed guard hook lets the action through.
- `bash` (3.2 or later, so stock macOS works), `git`, `awk`, `grep`, `sed`, `python3`. `jq` is optional.

## Quick start (about 5 minutes of your time)

1. Clone this repo to the path the guide and the prompt expect:
   ```bash
   git clone https://github.com/OfficialCodeVoyage/claude-code-project-foundation ~/new-project-foundation
   ```
   If you put it somewhere else, replace `~/new-project-foundation` in the prompt below with your path.
2. Open a terminal **in the project you want to set up** and start `claude`.
3. Paste the prompt below. Change the "Facts about this repo" line if it is not an existing solo repo
   (see "Variants").
4. Let it work. It stops once, at the end, with one batch of questions (recommendation first). Answer them.
5. Read its report, look at the branch `chore/agent-setup-<date>`, then push or open the PR yourself.

### The prompt: copy from the first line to the last

```
Read ~/new-project-foundation/new_structure.md in full (§0–§17, the kit list under "Appendices A–D", and Appendix E). Then set this repo up exactly as the guide says, copying the scripts from ~/new-project-foundation/kit/.

Facts about this repo: existing code, I am the only developer.

Rules for this job:
- Work on a new branch chore/agent-setup-<YYMMDD>. Commit there. Do not push, open a PR, or merge; I will.
- Never edit anything under ~/.claude.
- Never copy content, names or memory from another project. Only the structure.
- Do not assume the language or framework. Detect the stack from the repo (§2 Discovery), then research the exact installed versions with Context7 (resolve-library-id → query-docs) or the official docs, and record versions, breaking changes, formatter/lint/typecheck commands and gate coverage in .planning/agent-setup.md → "Stack research" before writing any stack-specific line.
- Build the Phase A structure, then run the Phase B gap report. Build only the zero-risk Phase B items.
- Finish everything first, then ask me every open decision in ONE batch, recommendation first.
- Report back in the §17 format, and run the §16 done-when checklist with evidence before you say it is done.
```

### Variants (edit the "Facts" line)

- **Empty repo, nothing written yet:**
  `Facts about this repo: empty repo created at its final path; I am the only developer.`
  The guide then scaffolds from its checklist instead of writing a gap report.
- **Repo shared with other developers:**
  `Facts about this repo: existing code, shared with other developers (their names: …).`
  The agent keeps every change on its own branch and flags shared files it would touch.

## What is in this repo

| Path | What it is |
|---|---|
| `README.md` | This file: the steps and the prompt to paste. |
| `new_structure.md` | The guide the agent follows: §0–§17, the kit list, and the templates in Appendix E. |
| `kit/claude/hooks/` | The hook scripts and their tests: `bearings-check.sh`, `bearings-stamp.sh`, `require-pr-on-blocker.sh`, `restore-state.sh`, `hooks.test.sh`. |
| `kit/claude/` | `settings.template.json`, `bearings.json`, `guard-push.json`. |

Check the kit at any time:

```bash
bash ~/new-project-foundation/kit/claude/hooks/hooks.test.sh        # the bash on your PATH
/bin/bash ~/new-project-foundation/kit/claude/hooks/hooks.test.sh   # macOS: bash 3.2, the oldest supported
```

Both must end with `== summary: <N> passed, 0 failed`. The tests build throwaway git repos in a temp folder and
touch nothing else.

## If you are not the author

The guide was written for one person's machine and says so in places. Its opening note explains how to read it:
"the owner" is you, "the global layer" is the author's private `~/.claude/` setup that you neither have nor need,
and the Working rules block in §4 holds the author's preferences, to be replaced with yours.

## What the push guard is, and is not

A speed bump for honest mistakes, not a security boundary. It parses the command text; it does not run a shell.
It does not see git aliases, a push inside a script file, `gh pr merge` or GitHub tools that push for you. The
header of `kit/claude/hooks/require-pr-on-blocker.sh` lists the detection rules and the known gaps. Keep the
ask-before-push rule from the settings template as the human check.

## Changing the kit

This repo is the home of the scripts. After any change to one:

1. Run the tests on both bash versions (above) and `shellcheck kit/claude/hooks/*.sh`.
2. In a project that installed an older copy, replace `.claude/hooks/<name>` with the new file and run that
   project's `.claude/hooks/hooks.test.sh`.

## License

MIT. See `LICENSE`.
