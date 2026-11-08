# PROJECT PROTOCOL

> The detailed *how* behind the workflow rules. Pulled in when a chat is actually at a checkpoint or relay moment — not something to re-read every turn (the always-on rules live in Project Custom Instructions instead, see bottom of this file for the condensed version).


## STATUS.md and CONTEXT.md writing rules

Use `edit\_file` only, on the specific lines that changed. A `write\_file` approval prompt for either file means a rule was broken: reject it. Respect the line limits in each file's header. Archive old entries to docs/archive/ instead of growing the file.

## 1. Checkpoints (every chat, every logical unit of work)

Do not implement continuously toward a fixed feature count. Stop at a **logically complete unit** — you (the agent) may propose where that is, but the user approves before you continue.

**"Feature complete" requires all of the following before a checkpoint is offered — code existing is not the same as done:**

```
Requirements understood  
        ↓  
Implementation complete  
        ↓  
Build succeeds  
        ↓  
Relevant tests pass  
        ↓  
No obvious errors  
        ↓  
Agent explains the implementation  
        ↓  
USER manually tests it  
        ↓  
USER approves  
        ↓  
   "Continue."
```

**Checkpoint report format** (agent produces this, every checkpoint). Use exactly this template:

```
CHECKPOINT \<id\>: \<feature name\>  
  
Changed: \<files changed / added / removed — same format as before, unchanged\>  
  
How to test:  
1. \<concrete step on the device\>  
2. \<...\>  
Expected: \<what you should see\>  
  
Known issues: \<specific item, or "none"\>  
  
Next: \<next checkpoint id + one line on what it covers\>  
  
\[Optional — include only if relevant, otherwise omit the line entirely\]  
Decisions: \<architecture decision the user should know about\>  
Inspect: \<file worth reading before approving\>
```

Rules for the report:

- Use this format exactly. No extra prose before or after it.

- Omit optional lines when empty. Never print empty or "N/A" fields.

- Do not print whole files back; describe the change and name the file.

- The "Next" line matters — the user should be able to say "I've tested this and understand it, continue," not just "continue" blind.

**If problems are found:** fix before proceeding. Don't accumulate unresolved issues across checkpoints.


## 2. Change Preservation (any chat, before touching existing code)

**Before modifying an existing file**, understand: what it currently does, why it exists, who calls it, what behavior must survive the change.

**Before deleting anything**, state:

```
File/function being removed:  
Reason:  
Replacement:  
Existing callers affected:  
Tests confirming the replacement works:
```

**Before a substantial rewrite**, state:

```
Current implementation:  
Why it needs changing:  
New implementation:  
Behavior preserved:  
Potential regression:
```

Never accept or act on a bare instruction to "clean up" or "improve" existing code without this — that phrasing is the easiest way for working code to quietly break.


## 3. Git Discipline

Every approved checkpoint = a commit. Every approved checkpoint also gets a light handoff (§4a). Every full relay (§4b) = a tagged checkpoint.

```
main  
 ├── checkpoint-01-\<feature\>  
 ├── checkpoint-02-\<feature\>  
 ├── ...  
 └── checkpoint-0N-handoff-to-chat-\<N+1\>
```

No agent inherits unverified work — every handoff begins from a known-good, committed checkpoint, not "whatever files happen to be sitting there."


## 4. Handoffs (two tiers)

Token cost grows with chat length, so handoffs are split into a cheap one done at every checkpoint and a full one reserved for real relays. Default rule: **one checkpoint = one chat** where practical, so full relays are rare.

### 4a. Light handoff (every approved checkpoint)

```
1. Commit and tag the checkpoint.  
2. Append to STATUS.md (max ~15 lines):  
   - Checkpoint id, date  
   - Done: features completed  
   - Files: added / modified / removed  
   - Decisions: any that affect later work  
   - Known issues  
   - Next step  
3. Update REPO\_MAP.md if files were added, removed, or changed role.
```

No repo inspection, no full build review, no full diff. This keeps the next chat able to continue without a full audit.

### 4b. Full handoff (only when relaying to a new/different chat)

Triggered when: context is degrading (repeated thrashing on the same bug, forgetting earlier constraints), the chat is nearing its practical length limit, or a chat's scope of work is genuinely complete and passing to another role. Agent will be notified.

**Outgoing chat's final handoff checklist:**

```
1. Stop implementing new functionality.  
2. Inspect the entire repository.  
3. Run the complete build.  
4. Run all relevant tests.  
5. Check for compilation warnings/errors.  
6. Review the full Git diff since the last handoff.  
7. Look specifically for: TODOs, placeholder implementations, dead code,  
   duplicated logic, security problems, incomplete features,  
   inconsistent architecture, temporary hacks.  
8. Verify existing functionality still works.  
9. Update STATUS.md.  
10. Write HANDOFF.md (template below).  
11. Do not modify functionality after this audit unless fixing a  
    verified problem found during it.
```

**HANDOFF.md contents:**

```
Project purpose  
Current architecture  
Every implemented feature  
Important files  
Important classes/functions  
Data flow  
Navigation flow  
Dependencies  
Design decisions and why  
Known issues  
Technical debt  
Unimplemented requirements  
Testing performed  
Recommended next steps
```

**Optional, worth including for anything non-trivial — a developer walkthrough per subsystem**, more useful than prose:

```
What problem does this solve?  
Where does execution begin?  
What classes are involved?  
How does data flow?  
What happens when something fails?  
What would I modify to change X?  
What files should I inspect first?
```

e.g. `LoginScreen.kt → AuthViewModel → AuthRepository → FirebaseAuth → Result → uiState → LoginScreen`


## 5a. Testing Conventions

- **Naming:** `subjectUnderTest\_scenarioOrExpectedResult` — e.g. `loadTasks\_error`, `clearCompletedTasks\_clearsTasks`. No "test" prefix, no vague names.

- **ViewModel/logic tests:** unit tests (`src/test/`), never touch a real DB or network. Inject a **fake** implementation of the repository interface (e.g. `FakeTaskRepository`) via the same constructor/DI path production code uses — this is why the contract-first approach in CONTEXT.md matters: a clean interface is what makes faking possible at all.

- **DAO/Room tests:** instrumented tests (`src/androidTest/`), since Room needs a real (in-memory) database — don't try to unit-test these.

- **Coroutines:** use `runTest` + a test coroutine rule so async code runs synchronously in tests, not real delays.

- A checkpoint's "relevant tests pass" (§1) means: unit tests for anything touched in `src/test/`, instrumented tests for anything touched in `src/androidTest/` — not just "the app still builds."

## 5. Handoff Audit (incoming chat, before writing any code)

The receiving chat does NOT start building immediately, even with STATUS.md/HANDOFF.md in hand. It audits first, but the audit is **scoped**, not a whole-repo scan:

> Before modifying anything: read STATUS.md and REPO\_MAP.md, run `git log` and the diff since the last tag, and run the build and tests. Verify the summary matches the diff. Open individual files only to resolve a specific discrepancy. Identify architectural inconsistencies, incomplete implementations, suspicious changes, deleted functionality, and potential regressions. Do not scan the whole repository unless this follows a §4b full handoff or the diff and STATUS.md disagree. Do not begin new feature development until the existing implementation is understood.

**Audit report format:**

```
HANDOFF AUDIT  
  
Last tag: ...  
STATUS.md matches diff: YES / NO (details)  
Build / tests: PASS / FAIL  
Problems or unexpected changes: ...  
Missing pieces: ...  
Recommended next checkpoint: ...
```

User approves the audit before the incoming chat writes anything. Then resume the normal checkpoint loop from §1.


## 6. Repo Map and Read Discipline

Exploring the repo costs tokens twice: once when a file is read, and again on every later turn because it stays in context. To keep this small:

**Maintain REPO\_MAP.md** at the repo root. Update it at checkpoints (§4a) whenever structure changes.

```
\# REPO MAP (updated at checkpoint \<id\>)  
  
Entry points: MainActivity.kt → NavHost → \<screens\>  
  
Data flow: \<Screen\> → \<ViewModel\> → \<Repository\> → \<Dao / Network\>  
  
Files:  
- path/File.kt — one-line role  
- ...
```

**Read rules (every chat):**

- Read STATUS.md and REPO\_MAP.md first.

- Open only files named in the task or on the map's path for that feature.

- Prefer search or line ranges over whole-file reads.

- Never read `build/`, `.gradle/`, or generated output.

- Show only the tail or the failures of build and test output.

- Use targeted edits instead of rewriting files.

- For well-specified tasks, implement directly; don't list alternatives unless asked.


## Condensed version for Project Custom Instructions

Always-on rules. Merge with existing Custom Instructions rather than replacing them.

```
Follow PROTOCOL.md.  
- Stop at each logical unit; wait for my approval before continuing.  
- Checkpoint reports use the fixed template (Changed / How to test /  
  Known issues / Next; optional Decisions, Inspect). No extra prose.  
- After each approved checkpoint: commit, tag, append to STATUS.md  
  (light handoff), update REPO\_MAP.md if structure changed.  
- New chat: read STATUS.md + REPO\_MAP.md, check the diff since the  
  last tag, run build/tests, report, wait for approval. Don't scan  
  the whole repo.  
- Read only files named in the task or on the map's path. Use  
  search/line ranges. Never read build/, .gradle/, generated dirs.  
- Targeted edits, not rewrites. Build/test output: failures or tail only.  
- Before deleting or substantially rewriting code, state what it does,  
  why it changes, and what must be preserved.  
- For well-specified tasks, implement directly; don't list  
  alternatives unless asked.
```

