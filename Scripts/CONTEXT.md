# PROJECT CONTEXT

> Ground-truth technical facts for the app this chat is working on. This file may cover several apps, so it has one section per app: confirm which app this chat concerns before writing code, then read only that section. If a chat's output contradicts this file, the code is the bug, unless the decision deliberately changed (then log it in STATUS.md). Only change it when a decision or constraint actually changes.

> EDIT RULES: `edit_file` only, one section at a time. Never `write_file`, never rewrite the file whole. Max 100 lines. Each decision max 2 lines (decision and why). No code, no per-file descriptions, no status.

## App: [name]

- One-line description: [ ]
- Repo path: `AndroidStudioProjects/[ ]/`
- Package: [ ]
- Min SDK: [ ]
- Target / Compile SDK: [ ]
- Java target: [ ]
- Kotlin: [ ]
- AGP: [ ]
- Compose BOM: [ ]
- DI: [ ]
- Key libraries: [ ]
- Architecture: [ ]
- Testing: [ ]
- Key decisions: [decision, why. Max 2 lines each]

*Add one `## App:` section per active app by copying the block above. Remove a section once an app is finished or abandoned, so it doesn't linger as stale ground truth for a chat that opens later. This file only answers "what environment and tools is this app built against". Progress, decisions in flight and blockers belong in STATUS.md; name the app in the entry.*
