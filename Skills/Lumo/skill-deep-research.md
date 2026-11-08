# SKILL: /Deep Research (v2)

Trigger: the user types `/Deep Research [topic]`, or asks you to research or search for something. With no topic, research the open UNKNOWN and VERIFY items in the current plan.
Output: a research report in the format at the bottom. It feeds the plan and, later, the /prompt skill.

## Honesty rule (read this first)
A label you cannot back up is a defect. If you are unsure which label applies, use the LOWER one. A short honest report beats a long confident one.

## Step 0: Preconditions and scope
- Web search must be on. If it is off or unavailable, say so and stop.
- Your own knowledge may be out of date. For libraries, versions, APIs, permissions and policies, always search. Never answer from memory.
- The topic must be a specific question ("how do apps detect floor level with the barometer, and what breaks?"). If it is vague ("sensors for a unique feature"):
  - Either ask the user to narrow it with ONE question, or
  - Run in LANDSCAPE mode: a lighter report with only sections 1, 2, 3, 6 and 7 below. No proposed decisions.

## Step 1: Frame the question
Restate the topic in one line. Break it into 3-6 sub-questions drawn from: feasibility, existing solutions and libraries, implementation patterns, known pitfalls, platform and policy constraints, alternatives. Research each sub-question with its own separate searches. Do not combine them into one query.

## Step 2: Search in this order of trust
1. Official docs (for Android: developer.android.com, official library docs)
2. Maintainers' sources: repo README, releases, changelogs, migration guides
3. GitHub: repos, issues, discussions, pull requests. These give real examples and known bugs
4. Community threads: Stack Overflow, Reddit (r/androiddev), library discussion boards. These give pitfalls and workarounds
5. Blogs and articles, as the last resort

Useful query shapes (adapt them):
- `<topic> sample app kotlin site:github.com`
- `<library> issue <problem> site:github.com`
- `<topic> site:stackoverflow.com`
- `<topic> site:reddit.com/r/androiddev`
- `<library> migration guide` / `<library> changelog`

Aim for 8-20 searches on a substantial topic and fewer on a narrow one. Vary the wording when a query misses.

## Step 3: Evidence rules
Every finding gets exactly one label:
- **READ**: you opened the page and can name one specific detail you saw on it (a date, a version, a stated limit, a section title). Write that detail after "seen:". No "seen:" detail means you may NOT use READ.
- **SNIPPET-ONLY**: you only have the search snippet, or you cannot open the page. This is the DEFAULT label. Add the URL to "Needs a human look".
- **SINGLE-SOURCE**: add it to READ when only one independent source supports the claim.

Rules by claim type:
- Claims about APIs, permissions, version behavior and store policy must come from OFFICIAL docs. If the only source is an article, forum or blog, still report it, but add "unofficial, VERIFY".
- Version attribution: state the Android version or API level where a behavior starts only if the cited source says so. Otherwise write "version not confirmed". Never merge two different changes into one version.
- Prefer sources from the last 12 months for fast-moving libraries. Flag older ones.
- When sources conflict, report the conflict. Do not pick silently.
- Never invent URLs, repo names, versions or code.

## Step 4: Dig into threads and repos
For each promising thread or repo, open it. Do not rely on the search snippet.
- Threads: record the problem, the accepted fix or workaround, the date, whether it is resolved, and which version it applies to. Prefer threads with a confirmed fix and recent activity.
- Repos: record last commit date, license, whether it is maintained, and the minimum platform version it supports. If you could not confirm a field, do NOT write "unknown" in the repo line. Move that repo to "Needs a human look" instead.
- Examples: summarize the pattern in your own words. Quote code only when essential and short. Note the license, because code may be adopted.

## Step 5: Stop rule
Stop when every sub-question is either answered from sources or listed as an unknown. Do not pad.

## Report format
```
DEEP RESEARCH: <topic>
Mode: FULL | LANDSCAPE
Sub-questions: ...

1. SUMMARY (max 5 lines)

2. FINDINGS
- <finding> | READ (seen: <detail>) / SNIPPET-ONLY / + SINGLE-SOURCE | <URL> | <date> | official / unofficial

3. OPTIONS AND TRADEOFFS
(2-3 options, then a recommendation and why)

4. PITFALLS FROM THREADS
- <pitfall> | fix or workaround | <URL> | resolved: yes/no

5. REFERENCE IMPLEMENTATIONS (FULL mode only; only repos you opened)
- <repo URL> | what is relevant | license | last commit | min platform

6. NEEDS A HUMAN LOOK
- <URL> | why it matters

7. VERIFY (max 5. Only items that would change a decision. For Code Claude to check against the real repo)
- ...

8. PROPOSED DECISIONS (FULL mode only, and only when a specific app or problem is already chosen)
- Product and architecture-level choices only, each "decision, why", max 2 lines.
- No API calls, libraries, permissions or SDK choices here. Those are Code Claude's call. Put them in section 7 as hints instead.
```
