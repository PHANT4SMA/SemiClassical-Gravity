# PROJECT PROTOCOL

> The detailed *how* behind the workflow rules. Pulled in when a chat is actually at a checkpoint or relay moment — not something to re-read every turn (the always-on rules live in Project Custom Instructions instead).

---

## 0. Setup and limits

Before CheckPoint-01, a git repo exists with a first commit. No repo, no checkpoints.
Claude can't run git or Gradle. The user runs them and reports. Claude states build/test results only if the user reported them.

---

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

**Checkpoint report format** (agent produces this, every checkpoint):

```
What I changed
Files created
Files modified
Files deleted
Architecture decisions made
How the feature works
How to test it
What you should inspect
Known issues
What I recommend doing next
```

The "recommend next" line matters — the user should be able to say "I've tested this and understand it, continue," not just "continue" blind.

**If problems are found:** fix before proceeding. Don't accumulate unresolved issues across checkpoints.

---

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

---

## 3. Git Discipline

Every approved checkpoint = a commit. Every relay (see §4) = a tagged checkpoint.

```
main
 ├── checkpoint-01-<feature>
 ├── checkpoint-02-<feature>
 ├── ...
 └── checkpoint-0N-handoff-to-chat-<N+1>
```

No agent inherits unverified work — every handoff begins from a known-good, committed checkpoint, not "whatever files happen to be sitting there."

---

## 3a. STATUS at a checkpoint

- **Offered:** Claude sets the CheckPoint row to WAITING and adds any [DECISION] or [BLOCKER] line, using edit_file within the limits in STATUS.md's header.
- **Approved:** the user commits and tags `checkpoint-NN-<feature>` and tells Claude the tag. Claude sets CONFIRMED and fills the Git tag column.
- Claude never sets CONFIRMED or records results on its own.

---

## 4. Handoff (relaying to a new chat)

One command: `/handoff` (the handoff skill). The user decides when to run it; Claude doesn't start it.

**Switch chats when:** context is degrading (looping on the same bug, forgetting earlier constraints), the chat is nearing its practical length limit, or the chat's scope of work is complete. Default is one checkpoint per chat, so relays should be rare.

**What HANDOFF.md holds:** only what exists nowhere except the chat. That means the resume point, open decisions, rejected ideas, gotchas, decisions missing from STATUS.md, and the user's preferences. Anything STATUS.md, CONTEXT.md, git or the code already holds is left out and pointed to instead. The skill carries the template and the size cap. HANDOFF.md is replaced on every handoff.

**Outgoing sequence:**
```
1. Stop implementing new functionality.
2. User runs build and tests and reports the results. Claude states
   results only if the user reported them.
3. Run /handoff. Claude reads STATUS.md only and writes HANDOFF.md.
4. Claude lists any STATUS gaps (decisions, blockers, rows missing from
   STATUS.md). User approves before anything is edited.
5. User tags the last known-good commit per §3
   (checkpoint-0N-handoff-to-chat-<N+1>), then commits HANDOFF.md and any
   in-flight code on top as a WIP commit with no tag.
6. Start the new chat with the starter prompt from the skill and paste
   `git status -sb` and `git diff --stat <last-tag>`.
7. Do not modify functionality after the handoff unless fixing a verified
   problem found during it.
```

Prefer handing off right after an approved checkpoint. If a relay is forced mid-unit, the WIP commit in step 5 is what the incoming audit (§5) treats as unverified.

**Developer walkthroughs (HANDOFF.md, "Walkthroughs" section).** Include one per subsystem when this chat built or changed a non-trivial subsystem whose flow is not already in CONTEXT.md. Otherwise omit the section. These are the one exception to "no repo state in HANDOFF.md": they are more useful than prose and save the next chat from rediscovering the flow. One line per question, and skip any question the code makes obvious.

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

---

## 5. Handoff Audit (incoming chat, before writing any code)

The receiving chat does NOT start building immediately, even with STATUS.md/HANDOFF.md in hand. It audits first:

> Before modifying anything: read HANDOFF.md and STATUS.md, and use the git output the user pasted (`git status -sb`, `git diff --stat <last-tag>`). Verify the summary matches the diff. Open individual files only to resolve a specific discrepancy or to check files in the WIP commit. Do not scan the whole repository unless STATUS.md and the diff disagree, HANDOFF.md flags a problem, or the user asks. Do not begin new feature development until the existing implementation is understood.

**Audit report format:**

```
HANDOFF AUDIT

Last tag: ...
HANDOFF and STATUS consistent with the diff: YES / NO (details)
Build / tests: as reported by the user, or "not reported"
Problems or unexpected changes: ...
Recommended next step: ...
```

User approves the audit before the incoming chat writes anything. Then resume the normal checkpoint loop from §1.

---

## 5a. Testing Conventions

- **Naming:** `subjectUnderTest_scenarioOrExpectedResult`, e.g. `loadTasks_error`. No "test" prefix.
- **ViewModel/logic:** unit tests in `src/test/` with a fake repository injected through the production DI path. No real DB or network. Coroutines via `runTest`.
- **DAO/Room:** instrumented tests in `src/androidTest/` with an in-memory DB.
- **"Relevant tests pass" (§1)** means unit tests for what was touched in `src/test/` and instrumented tests for what was touched in `src/androidTest/`, not just "it builds."
