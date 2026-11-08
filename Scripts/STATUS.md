# PROJECT STATUS

> Read this first. At each checkpoint Claude edits the checkpoint row and any [DECISION]/[BLOCKER] line with `edit_file`; the user gives the git tag and approves CONFIRMED. Two zones below: CHECKPOINTS (one line each, edited in place, always reflects "now") and LOG (append-only, never edit or delete past entries).

> EDIT RULES: edit this file only with `edit_file`, changing just the lines that changed. Never use `write_file` on it, never rewrite it whole. Limits: file max 80 lines. Each Log entry is one line, max 25 words. Each checkpoint Note max 1 line. No code, no file lists, no test scripts (git and the chat hold those). If a limit would be exceeded, move the oldest Log entries to docs/archive/ first.

*(One line per checkpoint, planned ones included. Edit the line in place when its status changes. No file lists and no test scripts: git has the files, the chat has the tests. Keep each Note to a single short line.)*

Status key: **PLANNED** · **IN PROGRESS** · **WAITING** (code done, not yet tested by user) · **CONFIRMED** (tested and approved by user) · **BLOCKED**

| ID | Checkpoint | Status | Git tag | Note |
| - | - | - | - | - |
| CP-01 | \[title\] | PLANNED | n/a |  |


## Open Questions / Blockers

*(Anything currently stalling work because it's waiting on a decision or on you. Remove a line once it's resolved.)*

- [ ] 

## Log

*(Append-only. New entries go at the top, newest first. Never edit or delete a past entry — if something changes, write a new entry that says so.)*

*(Tags: \[DECISION\] for a choice made and why · \[BLOCKER\] for something that stopped work. No other tags: the table is the status.)*

- DD-MM-YYYY [DECISION] or [BLOCKER]: one line, max 25 words.
