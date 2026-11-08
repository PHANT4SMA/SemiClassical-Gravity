# SKILL: /prompt (v2)

Trigger: the user types `/prompt` (first prompt for a new app) or `/prompt <change>` (a change or addition to an app that already exists).
Output: ONE kickoff prompt for Code Claude, in a single code block so the user can copy it straight into a new chat.
Code Claude starts every chat cold. It has the repo, git, STATUS and CONTEXT. It does not have your plan. The prompt must therefore be self-contained: after reading it, Code Claude should need to ask nothing about WHAT to build or WHY. Code Claude decides HOW, and it splits the work into checkpoints itself.

## Gate: check before writing
Do not write the prompt until all of these are true. If any is false, list what is missing and ask the user ONE question about the first missing item.
- The user approved the scope (what the app will do and not do).
- Intent, target user and non-goals exist.
- Every decision has a "why".
- Every must-have has a "done when" that the user can observe by hand on a device.
- The biggest risks are identified.
- No TBD remains. Anything unconfirmed is listed under VERIFY.

## Modes
- NEW APP (`/prompt`): the whole first version.
- CHANGE TO EXISTING APP (`/prompt <change>`): STATUS and CONTEXT already exist. WHAT TO BUILD covers only the change. Repeat the intent, non-goals and relevant decisions, because the new chat has not seen them.

## Keep out (Code Claude owns all of this. Never include it)
- Checkpoints, build order, and STATUS rows. Code Claude plans these.
- SDK, language, build tool or library VERSIONS, and the CONTEXT fields (Code Claude fills CONTEXT from the repo)
- File, class or package names, project structure, and code
- Coding conventions, testing approach, definition of done, escalation rules
- Git commits and tags
- Device quirks and UI hazards. Code Claude finds those itself
Research findings are allowed as reference material (see below). Versions inside research are not instructions.

## Template
Fill every section. Write "None" where a section is genuinely empty. Never delete a section.
```
# KICKOFF: <App name> | <short title>
App: <name>. Confirm this is the app you are working on and read only its section in CONTEXT.
Mode: NEW APP | CHANGE TO EXISTING APP
One-line description: <...>

## INTENT
<2-4 sentences: what the app does and why it exists>

## TARGET USER
<who, and the situation they use it in>

## NON-GOALS (do not build these)
- <thing> (not now | not ever)

## DECISIONS (decision, why. Max 2 lines each)
- ...

## WHAT TO BUILD
Must-have capabilities, in priority order. You split these into checkpoints yourself.
1. <what the user can do> | done when: <what the user observes on a device>
2. ...

## RISKS TO CHECK EARLY (what could invalidate the plan. Reduce these first)
- <risk> | why it matters

## RESEARCH RELEVANT HERE (reference only. Verify against the repo before adopting)
- <finding> | <URL> | evidence: READ / SNIPPET-ONLY / SINGLE-SOURCE | license if code

## VERIFY (unconfirmed. Check these against the real repo and APIs first)
- ...
```

## Self-check before sending
- Could Code Claude start without asking the user anything about what to build or why?
- Is anything from the keep-out list in the prompt? Remove it.
- Does every must-have have a "done when" the user can observe?
- Is every decision at most 2 lines?
- Are all uncertain items marked VERIFY instead of stated as fact?
- Is the priority order a priority, not a build order?
