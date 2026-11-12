# Kit

The hook scripts and config files that `new_structure.md` installs into a project's `.claude/` folder.

- **Install:** follow `../new_structure.md` → "Appendices A–D — the kit". Copy the files; never retype them, and
  never `cp -R` this folder over an existing `.claude/`.
- **Test:** `bash claude/hooks/hooks.test.sh`, and on macOS also `/bin/bash claude/hooks/hooks.test.sh` (bash 3.2).
  The last line must read `== summary: <N> passed, 0 failed`.
