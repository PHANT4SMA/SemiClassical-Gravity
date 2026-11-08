# PROJECT CONTEXT

> Ground-truth technical facts for the app this chat is working on. The filesystem spans several Android Studio projects under `AndroidStudioProjects/`, so this file is organized per app: confirm which app this chat concerns before writing code, then read only that section. If a chat's output contradicts this file, the code is the bug, unless the decision deliberately changed (then log it in STATUS.md).Only change it when a decision or constraint actually changes.

> EDIT RULES: `edit\_file` only, one section at a time. Never `write\_file`, never rewrite the file whole. Max 80 lines. Each decision max 2 lines (decision and why). No code, no per-file descriptions, no status.

## App: \[ \]

- One-line description: \[ \]

- Repo path: `AndroidStudioProjects/\\\[ \\\]/`

- Package: \[ \]

- Min SDK: \[ \]

- Target / Compile SDK: \[ \]

- Java target: \[ \]

- Kotlin: \[ \]

- AGP: \[ \]

- Compose BOM: \[ \]

- DI: \[ \]

- Key libraries: \[ \]

- Architecture: \[ \]

- Testing: \[ \]

*Add one `\\\#\\\# App:` section per active app — copy the block above. Remove a section once an app is* *finished or abandoned, don't let it linger as stale ground truth for a chat that opens later.* *This file only answers "what environment/tools is this specific app built against." Per-chat* *progress, decisions, and blockers belong in STATUS.md — tag STATUS entries with which app they* *concern.*

