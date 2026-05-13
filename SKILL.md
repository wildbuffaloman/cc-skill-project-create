---
name: project-create
version: "0.3.0"
description: Create a Project or Program brief through research, an overlap-check routing gate (may route the work into an existing brief instead of creating a new one), interactive Q&A, template application, vault linking, and INBOX delivery for review. Projects use a two-phase flow — Phase 1 (Scoping) drafts the brief, Phase 2 (Review & Commit) iterates, then creates a dedicated folder and assigns final status.
user-invocable: true
argument-hint: "note path, note title in INBOX, or topic description"
---

Create a Project Brief or Program Brief through a structured interactive process — research the input, run a guided Q&A to scope the initiative, decide project vs. program, apply the correct template, link to relevant vault notes and external references, and deliver for review.

**Projects follow a two-phase commit:**
- **Phase 1 — Scoping**: research + Q&A + draft brief saved to INBOX. The Continuation Prompt contains the Phase 2 activation text so the flow is resumable across sessions.
- **Phase 2 — Review & Commit**: present the draft, let the user edit or brainstorm further, then on confirmation create a dedicated folder under `01 PROJECTS/<Brief Name>/`, move the brief into it, and set the final status (active or incubating).

**Programs keep a single-phase flow** — draft, save to their area folder, cross-link.

## Philosophy

Every idea deserves a fair shot at becoming a project — but only after it has been properly scoped. This skill applies the vault's project scaffold requirements: Outcome, Plan with 3-7 milestones, Actions with next 10, Log with proof of motion. If it cannot fit that structure, it belongs in incubation, not active projects.

The skill is conversational — it gathers context through research and Q&A before writing anything. The user drives scope decisions; the AI provides structure, research, and vault connections.

Pipeline: `Input -> Research -> Route -> Q&A -> Classify -> Template -> Link -> Deliver -> Cross-Link`

The `Route` step (Step 1.5) is the overlap-check gate: a similarly scoped existing project may already cover this work, in which case the skill either spawns the new initiative as a **sub-project** of an existing parent, folds it **inside** an existing brief (as Next Action / Waiting For / milestone / Working Notes subsection / Sub-Project Index row), or proceeds with a genuinely new brief. The gate is mandatory.

## Vault Exception

This skill may:
- Create new files in `00 HUB/00 INBOX/` — the draft project/program brief and any companion files
- **For Projects only**, during Phase 2 finalization: create a new folder under `01 PROJECTS/<Brief Name>/` and move the confirmed brief from INBOX into that folder
- Read files anywhere in the vault for research and linking
- **Modify existing vault files only for cross-linking** — inserting wikilinks to the new brief into related notes (Step 7). All proposed edits must be listed and confirmed by the user before execution. No other modifications to existing files are permitted.
- **Edit the draft brief in place during Phase 2** — iterative revisions before the user confirms the final version are allowed on the draft file itself.
- **Modify an existing brief during Step 1.5 fold-in** — when the routing gate (Step 1.5d) routes the work into an existing brief instead of creating a new one, surgical insertion into that brief's Next Actions / Waiting For / Plan / Working Notes / Log is permitted with user approval (same contract as Step 7 cross-link scan). No new brief is created on this path.

## Inputs

The user provides one of:
- **Note path** — a full path to an existing note to convert into a project brief
- **Note title** — a filename in `00 HUB/00 INBOX/` to convert
- **Topic description** — a freeform description of the idea or initiative

If the input is a note, read it first. Its contents become the seed material for research and Q&A. If the note is empty or title-only, use the title as the seed topic.

## Steps — Interactive Conversation

Work through these steps. Pause after each step to get user input before moving on.

### Step 1: Research Phase

**1a — Read the input**
If a note was provided, read its full contents. Extract: topic, any stated goals, people mentioned, areas referenced, existing links.

**1b — Vault search**
Search the Obsidian vault for related material:
- Grep and Glob across `01 PROJECTS/`, `02 AREAS/`, `03 REFERENCE/` for notes matching the topic and related keywords
- Check for existing projects or programs that this initiative might relate to, be a sub-project of, or conflict with
- Find relevant Area MOCs, Programs, and Reference materials
- Look for OKRs, strategic plans, or other planning documents that provide context

**1b.5 — Find related prior reading** *(opt-in, auto-suggested)*

After the structural vault search in 1b, run [[find-related]] to surface scored candidates across `00 HUB/00 READ_REVIEW/` + `03 REFERENCE/` + wikis. The basic grep/Glob in 1b finds matches by filename and keyword presence; `/find-related` adds frontmatter overlap + wikilink graph + AI semantic scoring, and brings in READ_REVIEW (which the grep step typically skips because it's intake, not kept content).

Invoke:

```
/find-related "<one-line topic from 1a>" --area "[[<area inferred so far>]]" --target "00 HUB/00 INBOX/<Brief Name>.md"
```

If the project's area is not yet known, omit `--area` — the skill will scan broader. The `--target` flag pre-declares the artifact Phase 2 will modify (the draft brief that Step 5c will save).

The skill writes a checkbox manifest to `00 HUB/00 INBOX/YYYY-MM-DD-find-related-<slug>.md`. Embed its High and Medium confidence candidate tables inline in Step 1d's research findings summary so the user reviews them alongside vault search results and web findings.

**Skip when:** the topic is genuinely novel (no prior reading expected), the user explicitly says "skip prior reading scan", or the project is a structural admin task with no topical surface.

**Phase 2 apply runs during Step 6d** — once the brief is committed, run `/find-related apply <manifest-path>` to materialize user-marked items into the brief's `### Related Reading` subsection before the cross-link scan in Step 7. See Step 6d step 6.

**1c — Web research** — only if the topic benefits from external knowledge
Use WebSearch to find:
- Best practices, frameworks, or methodologies relevant to the initiative
- Industry benchmarks or reference points
- Common pitfalls and critical success factors

**1d — Present research findings**
Share a concise summary of what was found — vault connections, relevant context, and external insights. This grounds the Q&A in real context.

### Step 1.5: Overlap Check & Routing Decision (mandatory)

This is a **forcing-function gate**. Every `/project-create` run must execute it. Its job is to answer one question before any Q&A starts: **"Given what Step 1 found, is this really a new project, or does the work actually belong inside (or under) an existing one?"**

The user's standing directive is verbatim: *"should always check if there is a similarly scoped project existing to suggest to the user to have the new project either be a spawned as a sub-project (if scope merits it) or simply as a new feature/slice/milestone/task/Next Action/research/design/etc. within an existing project."*

**1.5a — Score candidates from Step 1b / 1b.5 results**

Promote a vault-search hit to *candidate* status if it satisfies any of:

- **Sub-project signal** — same `parent:` program as the proposed area's likely program
- **Sibling signal** — same area + topic-keyword overlap ≥ 2 keywords
- **Semantic signal** — `/find-related` High or Medium confidence hit with `status: active | incubating | delegated`
- **Fold-in signal** — has an open Next Action or Milestone row whose phrasing matches the proposed initiative
- **Duplicate signal** — same H1 / frontmatter `description:` first sentence semantically overlaps with the proposed Outcome

Build a candidate list (max 5, ranked by signal strength). **If the list is empty:** write *"No strong overlap found — proceeding as new project."* in the conversation and continue to Step 2. The explicit statement is the audit trail; do not skip the gate silently.

**1.5b — Present candidates and route**

For each candidate, surface in one compact block:
- Path + name (as a `[[wikilink]]`)
- `status:` / `parent:` / first-sentence Outcome
- **Recommended route** — one of: `sub-project` · `fold-in` · `sibling-new` · `considered-but-distinct`
- **Why** — one sentence: which signal fired

Then use AskUserQuestion with four route choices:

| Route | Meaning | Flow consequence |
|-------|---------|------------------|
| `new` | Genuinely standalone — none of the candidates fit | Continue to Step 2; note candidates in Working Notes per Step 1.5 fall-through |
| `sub-project of [[X]]` | Spawn under existing parent | Go to **1.5c**; lock `parent: "[[X]]"`; continue to Step 2 |
| `fold into [[X]]` | This is a feature/NA/milestone/research/design *inside* an existing brief | Go to **1.5d** — early exit, no new brief |
| `defer / not now` | Don't create anything; capture as someday-maybe or drop | Go to **1.5e** — early exit |

**1.5c — Sub-project path** (route = `sub-project`)

- **If [[X]] is a program** (`category: program`): lock `parent: "[[X]]"` for Step 4 frontmatter. Surface the **SUB-PROJECT-INDEX markers check** (the same check Step 6d.5 runs) right now so the user knows a marker-insertion proposal may come at commit time. Continue to Step 2 with Q&A framed as *"scoping a sub-project of [[X]]"* — skip the parent-program question in Q&A.

- **If [[X]] is a project** (`category: project`): sub-projects-of-projects is unusual. Ask: *"Is this really a sub-project, or is it a slice/NA of [[X]]?"*
  - Slice/NA → re-route to **1.5d** (fold-in).
  - Genuinely a sub-project → first surface that [[X]] should probably be promoted to a program (separate decision — out of scope for `/project-create`). Capture the promotion question as a Working Notes entry on the new brief, continue as standalone, and link [[X]] as `depends_on`.

**1.5d — Fold-in path** (route = `fold into [[X]]`)

This is the early-exit branch. `/project-create` does **not** produce a new brief. Execute:

1. Read [[X]] in full to confirm structure (which sections it has, which it doesn't).
2. Ask the user to pick the **shape** of the fold-in via AskUserQuestion:
   - `Next Action` — single concrete action goes into [[X]]'s `## Next Actions`
   - `Waiting For` — item from another person goes into [[X]]'s `## Waiting For`
   - `Plan milestone` — new milestone row (with target) goes into [[X]]'s `## Plan with Milestones`
   - `Working Notes subsection` — research/design context goes into a new H3 under [[X]]'s `## Working Notes`
   - `Sub-Project Index row` — only valid if [[X]] is a program and the fold-in is actually project-shaped → re-route to **1.5c** (sub-project).
3. Build a one-edit manifest specifying: target file path, target section, exact text to insert.
4. Present the manifest. Apply only on user approval, via the Edit tool, with surgical insertion that preserves existing structure (matching list/table formatting, frontmatter intact).
5. Append a Log row to [[X]]: `| <today> | Task | Folded in via /project-create routing gate (originally proposed as standalone) |`.
6. **Dispose of the input note** (only if the input was an INBOX note): ask the user via AskUserQuestion — default option `link from [[X]] Key Resources + leave in INBOX for /inbox-clear`, alternatives `archive to 04 ARCHIVES/`, `delete`. **Never silently delete.**
7. **Terminate.** Skip Steps 2-7 entirely. Print a one-line summary: *"Folded `<topic>` into [[X]] as `<shape>`. No new brief created."* That's the end of the run.

**1.5e — Defer path** (route = `defer / not now`)

Ask via AskUserQuestion: *"Capture as a someday note in `00 HUB/00 INBOX/someday-<slug>.md`, or drop entirely?"* Apply choice. Terminate — skip Steps 2-7.

**1.5f — Fall-through (route = `new`)**

Continue to Step 2 normally. Before doing so, capture a one-line entry that will land in the new brief's Working Notes under a `### Considered Alternatives` H3 subsection (see Step 4 content-quality standards): *"Considered [[Candidate A]], [[Candidate B]] during Step 1.5 — proceeded as standalone because `<one-sentence rationale>`."* This is the audit trail that the gate ran.

**1.5 Red flags — STOP and run the gate**

These are the rationalizations that would skip the check. If any of these thoughts surface, the gate has **not** been run yet:

| Excuse | Reality |
|--------|---------|
| "The user clearly wants a new project, no need to ask" | The user invoked the skill *to be helped scope*. Skipping the gate denies the help. |
| "I already searched in 1b — this is redundant" | 1b returns candidates; 1.5 *routes* them. Different jobs. The gate is not a second search. |
| "There are no obvious matches, the gate is a waste of words" | State *"No strong overlap found — proceeding as new project"* explicitly. The audit-trail statement *is* the gate. |
| "The candidates are clearly weak, no need to surface them" | Surface them anyway as `considered-but-distinct`. The user decides relevance — not the skill. |
| "This is obviously a fold-in, just apply the edit" | Even on obvious fold-ins, present the manifest and get user approval before editing [[X]]. |
| "The input note was so specific the user already decided" | The input is a *topic seed*, not a routing decision. The gate stands. |

**Violating the letter of Step 1.5 is violating the spirit of Step 1.5.** No exceptions.

### Step 2: Phase 1 — Scoping Q&A

> **Projects:** this is the first of two phases. Phase 1 scopes the brief; Phase 2 (Step 6) reviews and commits it.
> **Programs:** single phase — the answers collected here flow directly into Steps 4 and 5.

Run an adaptive Q&A to fill in all sections of the brief. Use AskUserQuestion with well-informed options based on Step 1 research.

> **If Step 1.5 routed to `sub-project of [[X]]`:** the `parent:` frontmatter is already locked to `[[X]]`. Skip the parent-program question in Q&A. Frame the entire Q&A as *"scoping a sub-project of [[X]]"* — Outcome must be scoped narrower than [[X]]'s Outcome, milestones must be sub-project-shaped, and the Working Notes should reference [[X]]'s Working Notes for shared context rather than re-deriving it.

**Core questions to cover** — adapt phrasing and options based on the specific topic:

1. **Trigger and motivation** — What triggered this? Pain points, strategic alignment, or opportunity?
2. **Outcome** — What does "done" look like? One sentence. If the user struggles, propose 2-3 options based on research.
3. **Duration and lifecycle** — Is this something that will be actively managed for more than 12 months, or does it have a concrete completion point?
4. **Scope** — How big is this? Focused project or phased rollout?
5. **Owner and champion** — Who drives this on the ground?
6. **Timeline** — What is the target timeline for the first milestone?
7. **Dependencies** — What must be true or in place before this can start?
8. **Budget and resources** — Is there budget? What resources are needed?
9. **Prior work** — Has anything been done before? What exists already?
10. **Risks** — What could derail this? What are the top 2-3 risks?
11. **Success metrics** — How will you measure success? What KPIs matter?
12. **AI-specific instructions** *(optional — skip if none)* — Are there project-unique AI rules not already covered by the area CLAUDE.md? Examples: terminology/glossary, specific tools or MCPs to always use, process conventions (e.g., "always run `/deep-research` before proposing"). If yes, capture them; they will land under `### AI Context` inside Working Notes. If "no" or "covered at area level", skip — do not create an empty section.

Not all questions apply to every project. Skip questions that are irrelevant or already answered by the input note. Add topic-specific questions based on the research phase findings.

**Batch questions where possible** — use AskUserQuestion with up to 4 questions per call to keep the conversation efficient. Group related questions together.

### Step 3: Project vs. Program Decision

Based on the scoping answers, recommend whether this should be a **Project** or a **Program**.

**Primary test — duration-based classification:**

> "Will this be actively managed >12 months from now?"
> - **YES → PROGRAM** (lives in `02 AREAS/`, `category: program`)
> - **NO → PROJECT** (lives in `01 PROJECTS/`, `category: project`)

**Supporting criteria** — use these to reinforce or challenge the duration test:
- Does it have 3+ distinct workstreams that could each be a project? → leans Program
- Does it need a sub-project index? → leans Program
- Can one person own and execute it? → leans Project
- Is it a strategic initiative spanning multiple areas? → leans Program

Present the recommendation with reasoning. Let the user decide.

### Step 4: Build the Draft Brief

Apply the correct template based on the decision. For **Projects**, the Continuation Prompt section is populated with the Phase 2 activation text (see 4a below) instead of the standard empty handoff template.

**For Projects** — use the Project Brief Template structure:
```
Frontmatter: description, AREA, SUB-AREA, category: project, status (active|delegated|incubating|blocked|someday-maybe), tags, owner: "[[Name]]", parent: "[[Program]]" (required — must point to a program), depends_on (array of wikilinks), feeds (array of wikilinks), learns_from (array of wikilinks)
H2 Outcome — one sentence blockquote
H2 Why This Matters — success stakes, failure stakes
H2 Continuation Prompt — empty template for session handoffs
H2 Next Actions — concrete next steps from Q&A
H2 Waiting For — items from other people
H2 Dependencies — human-readable list of dependencies with wikilinks and descriptions
H2 AI Ecosystem — Feeds Into (table), Learns From (table)
H2 Key Resources — Evergreen Notes, Reference Materials, People/Collaborators
H2 Working Notes — research findings, frameworks, context from Q&A
H2 Plan with Milestones table — 3-7 milestones with targets
H2 Log — creation entry
```

**For Programs** — use the Program Template structure:
```
Frontmatter: description, AREA, SUB-AREA, category: program, status, review_cadence, tags, owner: "[[Name]]", parent: "[[Parent Program]]" (optional — only for sub-programs), depends_on (array of wikilinks), feeds (array of wikilinks), learns_from (array of wikilinks)
H1 Title
H2 Outcome — one paragraph blockquote
H2 Why This Matters — success stakes, failure stakes
H2 Sub-Project Index — table: #, Sub-Project (wikilink embedded in name), Status, Description
H2 Continuation Prompt — empty template
H2 Next Actions
H2 Dependencies — human-readable list of dependencies with wikilinks and descriptions
H2 AI Ecosystem — Feeds Into (table), Learns From (table)
H2 Waiting For
H2 Key Resources — Evergreen Notes, Reference Materials, People/Collaborators
H2 Working Notes
H2 Plan with Milestones table
H2 Log
```

**Content quality standards:**
- Outcome must be one sentence for projects, one paragraph for programs — concrete and measurable
- Why This Matters must connect to real stakes — strategic plans, OKRs, business impact
- Next Actions must be concrete and actionable — no vague "research X" without specifying what to research and where
- Working Notes contain research findings, frameworks, and context gathered during the process — the institutional memory. When a project has AI-specific instructions that aren't already covered by area-level CLAUDE.md (project-unique terminology, tool preferences, process conventions), add them under an `### AI Context` H3 subsection. Do not duplicate area-level rules here — those auto-load via the CLAUDE.md chain.
- Key Resources must include actual vault wikilinks found during research, not empty placeholders
- Milestones should be 3-7, each with a target date or timeframe
- Tags should include the area, topic keywords, and any relevant programs
- Key Resources subsections are: **Evergreen Notes** (distilled insights), **Reference Materials** (source docs, links), **People / Collaborators**. Never use "Atomic Notes" — the vault term is Evergreen Notes.
- **Table wikilink convention:** Embed `[[wikilinks]]` directly in the name/title column of tables — never use a separate "Link" column. Applies to Sub-Project Index, Milestones, AI Ecosystem tables, and any other table with named entities.
- **Considered Alternatives subsection (when Step 1.5 surfaced candidates that were rejected):** under `## Working Notes`, add an `### Considered Alternatives` H3 subsection with one line per rejected candidate — wikilink to the candidate, route that was recommended, and the one-sentence rationale for proceeding as standalone instead. Skip if Step 1.5 found no candidates.

**4a — Phase 2 Activation block (Projects only)**

For projects, populate the `## Continuation Prompt` section with this activation text (substitute `<Brief Name>` with the actual filename stem). Do **not** use the standard empty handoff template — that is only restored after Phase 2 finalizes in Step 6.

```markdown
## Continuation Prompt

> **Status: DRAFT — Phase 2 pending**
> **Created by:** `/project-create`
> **Phase 1 complete:** research, scoping Q&A, and draft brief written.
>
> **To activate Phase 2, paste the following to Claude Code:**
>
> ```
> Resume /project-create Phase 2 for [[<Brief Name>]].
>
> 1. Read the current draft brief.
> 2. Present a concise summary of each H2 section and ask me:
>    "Is there anything else about this project brief you'd like to edit or brainstorm before committing?"
> 3. If I want to iterate — help me revise sections in place (surgical edits on the draft file).
>    Loop the question until I explicitly confirm the final version.
> 4. When I confirm:
>    a. Ask me for the final status via AskUserQuestion — choices: `active` | `incubating`.
>    b. Create the folder `01 PROJECTS/<Brief Name>/` (exact filename stem, no extension).
>    c. Move the brief from `00 HUB/00 INBOX/` into that folder.
>    d. Update the `status:` frontmatter field to my choice.
>    e. Replace this Continuation Prompt block with the standard empty handoff template.
>    f. Run the Cross-Link Scan (Step 7 of the `/project-create` skill).
> 5. Report final location, status, and cross-links applied.
> ```
```

While in DRAFT state, set `status: incubating` in frontmatter as a safe default — it will be overwritten in Step 6.

### Step 5: Deliver Draft

**5a — Insert vault links**
For every vault note identified as relevant in Step 1, insert proper `[[wikilinks]]` in the appropriate section — Key Resources for reference materials, Working Notes for context, Dependencies for blockers.

**5b — Insert external references**
For any external resources found in web research, add as markdown links in Key Resources or Working Notes.

**5c — Save the draft**

- **Projects:** save to `00 HUB/00 INBOX/<Brief Name>.md` with `status: incubating` and the Phase 2 activation Continuation Prompt from Step 4a. Do **not** create the `01 PROJECTS/<Brief Name>/` folder yet — that happens in Step 6 after the user confirms the final version.
- **Programs:** save to `00 HUB/00 INBOX/<Brief Name>.md` for review (single-phase flow). The user will move it to `02 AREAS/AREA_FOLDER/` after approving.

**5d — Present summary**
Tell the user:
- Where the draft was saved
- What vault notes were linked
- For projects: that the brief is in DRAFT state and Phase 2 (review & commit) is next — you will proceed immediately to Step 6 in the current session, but if the session ends, the Continuation Prompt can resume Phase 2 in a new session.
- For programs: the recommended destination folder under `02 AREAS/` — remind the user to move the file there after review.
- Any open questions or items flagged for their attention

### Step 6: Phase 2 — Review & Commit (Projects only)

> **Programs skip this step** and proceed directly to Step 7 (Cross-Link Scan). The user moves a program brief manually to its area folder.

This is the commit phase. The draft brief is already on disk in `00 HUB/00 INBOX/`. Now walk the user through final review, iterate as needed, then physically commit the project by creating its folder, moving the brief into it, and setting the final status.

**6a — Present the draft for review**

Summarize the draft concisely:
- Title and one-line outcome
- Area / sub-area / parent program
- Planned milestones (count + first target)
- Key dependencies and waiting-for items
- Notable gaps or sections flagged during Step 1-5 as "needs user input"

Then ask explicitly:

> **"Is there anything else about this project brief you'd like to edit or brainstorm before committing?"**

**6b — Iterate until confirmed**

If the user wants changes:
- Make surgical edits to the draft file in place (section-level, not full rewrites)
- For bigger brainstorms (e.g., scope expansion, new milestones, risk reassessment), run mini-Q&A via AskUserQuestion and then apply the results
- After each edit cycle, re-summarize what changed and repeat the question in 6a

Loop until the user gives an explicit confirmation — wording like "commit it", "that's final", "good to go", "ship it", or equivalent. **Do not assume confirmation from silence or an ambiguous response** — ask again.

**6b-bis — Pre-Commit Lock-Ins** *(applies inside the iteration loop)*

During Phase 2 review, the user may surface decisions that should be baked into the brief **before** commit, not deferred to NA1 of the spawned project. These decisions typically arrive as:
- Constraint pre-locks ("X must equal Y") that change Outcome/NA framing
- Override of recommended defaults that the brief's Working Notes should record as explicit decisions
- Scope narrowings/widenings that affect MVP definition or milestone shape
- Coordination rules with sibling sub-projects (e.g., "this NA8 must be tested in the same session as that sub-project's NA4")

**Capture them in the brief BEFORE the Phase 2c status flip**, not in NA1 working notes after commit. Reason: they materially shape the brief's structure (Outcome scope, NA list, milestone cadence, AI Context rules). Treating them as post-commit work loses the pre-commit context where the user's reasoning was fresh.

**How to apply:** When you detect a pre-commit lock-in during 6b's iteration loop:
1. Surface explicitly: "Should I lock this decision into the brief before commit, or capture as NA1 working notes?"
2. If lock-in: edit the relevant section (Outcome / Working Notes "Decisiones Lockeadas / Pre-NA1 Lock-Ins" subsection / NA description / AI Context) before re-running 6a's confirmation question.
3. The Log entry written in 6d step 5 should reference these lock-ins explicitly so future readers can trace why the brief committed at the shape it did.

**Surfacing case (2026-05-12):** During spawn of [[Herramientas de Desarrollo Equipo IA]], user supplied 2 pre-commit decisions (D6 roster = 6 seats full AIT; D5 override = Manuel migrates Max to Team seat). Both materially changed the brief's Outcome, Working Notes (new "Decisiones Lockeadas" subsection), Plan with Milestones (added milestone #6 for Manuel personal Max cancellation), and frontmatter description. Capturing as NA1 working notes would have buried the rationale. Pattern repeated 3 more times the same session (NA1B for Expansión MCP scoping, MVP narrowing for Catálogo, scope split for Repositorio Central + Skills Library) — 4 pre-commit lock-ins across 4 spawns confirmed the recurrence.

**6c — Ask for final status**

Use AskUserQuestion with exactly two choices:
- `active` — starting work now; counts toward the 7-project WIP cap
- `incubating` — committed to the project, but not starting work this week

Do not offer `delegated`, `blocked`, or `someday-maybe` here — those are edge cases that should be set manually later if relevant.

**6d — Create folder and move the brief**

Execute in order:
1. Create the folder `01 PROJECTS/<Brief Name>/` (use the exact filename stem, no `.md` extension).
2. Move the brief file from `00 HUB/00 INBOX/<Brief Name>.md` to `01 PROJECTS/<Brief Name>/<Brief Name>.md` (prefer `mv` via Bash to preserve timestamps).
3. Update the `status:` frontmatter field in the moved file to the user's choice (`active` or `incubating`).
4. Replace the Phase 2 activation Continuation Prompt block with the standard empty handoff template:
   ```markdown
   ## Continuation Prompt

   > **Date:**
   > **Mode:**
   > **Context:**
   >
   > **Resume from:**
   > **Key decisions this session:**
   > **Blockers/open questions:**
   > **Files touched:**
   ```
5. Add a Log entry for commit: `| <today> | Milestone | Project created and committed via /project-create (status: <active|incubating>) |`
6. **Apply `/find-related` manifest** if one was created in Step 1b.5. Look for `00 HUB/00 INBOX/YYYY-MM-DD-find-related-<slug>.md` matching this brief. If present and any items are checked, invoke `/find-related apply <manifest-path>` — this materializes checked items as wikilinks in the brief's `### Related Reading` subsection (idempotent on re-run; the subsection is created under `## Key Resources` if missing). If no manifest exists or no items were checked, skip silently. Log the apply outcome in the brief's `## Log` as a Task row.

**6d.5 — Verify parent program has SUB-PROJECT-INDEX markers**

If the new project's `parent:` frontmatter points to a program brief (i.e., a file with `category: program` in `02 AREAS/`), grep that program brief for `<!-- SUB-PROJECT-INDEX START -->` markers. Two outcomes:

- **Markers present:** `sync_vault_indexes.sh --apply` will auto-include the new project on next run. No action needed beyond the standard Phase 2c sync that `/session-close` performs.
- **Markers absent:** Propose to add them in a sensible location — typically a new `## Sub-Projects` H2 section placed immediately before `## Waiting For` or `## Plan` (preserve existing section order; insert as a peer):

  ```markdown
  ## Sub-Projects

  <!-- SUB-PROJECT-INDEX START -->
  <!-- SUB-PROJECT-INDEX END -->
  ```

  Show the user the proposed insertion point and the parent brief's current section order. Apply only on approval. Without these markers, `sync_vault_indexes.sh` cannot auto-list child projects in the parent — the index drifts silently and downstream consumers (`00 HUB/Program Hierarchy.md`, manual readers of the parent brief) miss the new sub-project.

  Skip this proposal if the parent program brief does not exist (orphan parent — already an error condition surfaced by Step 7 cross-link scan), or if the brief intentionally maintains its own hand-curated sub-project table (rare; check for an existing manually-formatted "Sub-Projects" section before proposing markers — markers and hand-curation conflict).

**6e — Confirm commit**

Report to the user:
- Final path: `01 PROJECTS/<Brief Name>/<Brief Name>.md`
- Final status
- Folder created: yes
- Ready to run Step 7 (Cross-Link Scan)

### Step 7: Cross-Link Scan

**This step is mandatory.** After the brief is committed (projects) or saved to INBOX (programs), run a comprehensive cross-link scan to strengthen the knowledge graph bidirectionally.

**7a — Comprehensive vault scan**
Search across all active vault areas for notes related to the new brief's topic, area, and keywords:
- `01 PROJECTS/` — sibling projects, sub-projects of the same parent program, projects with overlapping dependencies or feeds
- `02 AREAS/` — Area MOCs, parent program brief, sibling programs, related programs in adjacent areas
- `03 REFERENCE/` — reference notes, reading notes, templates, and guides related to the topic
- `00 HUB/` — Hub MOCs, other INBOX items that relate

For each candidate note, check both:
- **Frontmatter relationships** — `depends_on`, `feeds`, `learns_from`, `parent` fields that should reference the new brief
- **Body content** — sections like Key Resources, Working Notes, Sub-Project Index, or Related Notes where a wikilink to the new brief would add value

**7b — Build cross-link manifest**
Compile a list of proposed edits, organized by note. For each proposed edit, specify:
- The **target file** path and name
- The **section** where the link should be inserted
- The **link type** — is this a frontmatter relationship (e.g., adding to `depends_on` array) or a body wikilink?
- **Why** — a brief justification for why this link adds value (semantic relationship: supports, depends on, feeds into, sibling of, etc.)

Also identify any **frontmatter relationship updates** needed on the new brief itself (e.g., if the scan reveals a `depends_on` or `feeds` relationship that wasn't captured during Steps 1-5).

**7c — Present manifest for user approval**
Display the full cross-link manifest to the user. Group edits by target file. Include a count of total proposed edits and affected files. The user may:
- Approve all
- Approve selectively (remove specific edits)
- Skip the cross-linking entirely

**7d — Execute approved edits**
For each approved edit:
- Read the target file
- Insert the wikilink in the specified section, matching existing formatting
- For frontmatter array fields, append the new wikilink to the existing array
- For body sections, add the wikilink in the most natural position (e.g., as a new list item, table row, or inline reference)
- Preserve all existing content — surgical insertion only

If the scan also identified updates needed on the new brief itself, apply those as well (updating the brief at its current location — `01 PROJECTS/<Brief Name>/<Brief Name>.md` for committed projects, or `00 HUB/00 INBOX/<Brief Name>.md` for programs awaiting manual placement).

**7e — Report results**
Summarize what was linked:
- Number of files updated
- Total wikilinks inserted
- Any edits that failed (e.g., file structure was unexpected) with explanation
- The new brief's updated graph connections

## Rules

- Pause after each major step to get user input — this is a conversation, not a monologue
- **Step 1.5 (Overlap Check & Routing Decision) is mandatory.** Every `/project-create` run must execute the routing gate after Step 1d and before Step 2. Skipping the gate silently is forbidden. The audit-trail statement must appear in the conversation — either *"No strong overlap found — proceeding as new project"* (no candidates found) or the chosen route from the AskUserQuestion (candidates surfaced and routed). If the route is `fold into [[X]]` or `defer / not now`, the skill early-exits — no new brief, no folder, no Steps 2-7. If the route is `sub-project of [[X]]`, the `parent:` frontmatter is locked before Step 2 starts. If the route is `new`, the rejected candidates are captured in the brief's `### Considered Alternatives` Working Notes subsection.
- Use AskUserQuestion for structured choices — batch up to 4 questions per call
- Only include vault notes that are clearly relevant — when in doubt, leave it out
- Use exact note names without .md extension inside wikilinks
- Do not modify existing vault files except during Step 6 (Phase 2 commit — moving the draft from INBOX to `01 PROJECTS/<Brief Name>/`) and Step 7 (Cross-Link Scan — approved wikilink insertions). Draft-brief edits during Phase 2 iteration target only the draft file itself.
- Avoid empty placeholder wikilinks — omit sections with no content rather than leaving blanks
- Web research is optional — skip it for purely internal/organizational projects where external knowledge adds nothing
- If the input note already contains substantial content, pre-fill Q&A answers from it and confirm with the user rather than re-asking
- The vault working directory is resolved at runtime from the CLAUDE.md chain
- **Projects created by this skill go into their own dedicated folder:** `01 PROJECTS/<Brief Name>/<Brief Name>.md`. The folder is created in Step 6 after the user confirms the final version in Phase 2. Status is tracked via the `status:` frontmatter field.
- **Sibling skill / consumer coordination:** When modifying this skill's output convention (folder structure, filename pattern, frontmatter fields, required/optional sections, Continuation Prompt format) OR the skill's interaction contract with downstream consumers, update those consumers in the same change. **Current downstream consumers that must be kept in sync:**
  - **`/clean-projects`** — audits `01 PROJECTS/` layout and template compliance. Before finalizing convention changes here, grep its SKILL.md and confirm it handles both old and new behavior during any transition.
  - **`sync_vault_indexes.sh`** (at `05 AI/CLAUDE CODE/SATORI VAULT AI/scripts/`) — auto-populates `<!-- SUB-PROJECT-INDEX -->` markers in parent program briefs. Step 6d.5 above is the coupling: every project this skill creates under a `category: program` parent must end up listed in that program's index. The script is the consumer; this skill is responsible for ensuring the parent has the markers.

  Evidence (2026-04-22): folder-wrapped convention landed in `/project-create` weeks before `/clean-projects` was updated; user had to spot the drift manually.

  Evidence (2026-05-02): created [[AI Acceleration Phase0 Sessions 1st Class]] under [[AI Acceleration Club]] (a program brief). Parent had no SUB-PROJECT-INDEX markers because the brief predated the marker convention; `sync_vault_indexes.sh` returned "Programs missing markers (1)" and required a manual marker-insert + re-run before the new project could be auto-indexed. Step 6d.5 added to surface this proactively.
- Phase 2 restricts status choice to `active` or `incubating`. Other values (`delegated`, `blocked`, `someday-maybe`) can be set manually later if the project's state changes.
- Programs go to `02 AREAS/` under the relevant area folder — saved to INBOX, then moved manually by the user (no two-phase flow, no folder creation).
- Project Briefs always include: Outcome, Why This Matters, Continuation Prompt, Next Actions, Waiting For, Dependencies, Key Resources, Working Notes, Plan with Milestones, Log
- Program Briefs always include: Outcome, Why This Matters, Sub-Project Index, Continuation Prompt, Next Actions, Waiting For, Key Resources, Working Notes, Plan with Milestones, Log
- Never place Learning Programs or area-level concerns as projects — those have their own skill or template
- Log format: one item per row with Date, Type, and Update columns. Type is Task or Milestone.
- Waiting For: only items from other people or agents. Self-tasks go in Next Actions.
- All typed relationships go in YAML frontmatter as wikilinks per vault root CLAUDE.md Graph-Ready Conventions. Use `owner: "[[Name]]"`, `parent: "[[Program]]"`, and arrays for `depends_on`, `feeds`, `learns_from`. No inline Dataview fields in the body. Dependencies and AI Ecosystem sections contain human-readable context only (lists, tables, descriptions).
- **AREA is required** and must be a wikilink to a known area. The vault has area categories (folder groupings) and areas (the actual entities). Never put a category in the AREA field.
  - Valid AREA values: `[[01 HEALTH]]`, `[[01 THE DUHAUS]]`, `[[The Unschool]]`, `[[ADM - AED Mastermind]]`, `[[BUFFALO 4]]`, `[[DUVOG]]`, `[[FORO YPO]]`, `[[MENTORING]]`, `[[BUFALINDA]]`, `[[DAG]]`, `[[SATORI LLC]]`, `[[WEALTH MANAGEMENT]]`, `[[04 AI & SOFTWARE]]`, `[[MUSIC WORKS]]`, `[[VINILOVERSUS]]`, `[[WILD BUFFALO]]`, `[[WRITING]]`, `[[06 PERSONAL]]`
  - **Invalid** (these are categories, not areas): `[[02 COMMUNITY]]`, `[[03 BUSINESS]]`, `[[05 CREATIVE]]`
  - To determine the correct AREA, find where the parent program lives in `02 AREAS/` — the area-level folder/MOC is the AREA.
- **SUB-AREA** is optional. If present, it must be a wikilink. Only use when there's a meaningful subdivision within the AREA.
- **priority** must be one of: `critical`, `high`, `standard`, `low`. Default to `standard` if not specified.

## Dependencies

### Required

- Nothing beyond Claude Code's built-in tools (Read, Write, Edit, Glob, Grep, Bash for `mv`, WebSearch, WebFetch, AskUserQuestion). This skill does research, runs Q&A, writes briefs, and updates cross-links — all via built-ins.

### Optional

- **WebSearch / WebFetch** — used in Step 1c for external research on industry/domain topics. Fallback if the user declines web research (purely internal projects): skip Step 1c and proceed with vault-only context. Both are Claude Code built-ins; no external API keys required.
- **[[find-related]]** — used in Step 1b.5 to surface scored related items from `00 HUB/00 READ_REVIEW/` + `03 REFERENCE/` + wikis. The skill writes a checkbox manifest to INBOX; Step 6d step 6 applies user-marked items into the brief's `### Related Reading` subsection. Skip when the topic is genuinely novel or the user opts out — vault search in 1b still happens.

### Vault Conventions

- Applies [[Inbox Routing]] — drafts land in `00 HUB/00 INBOX/` before the Phase 2 commit moves Projects to `01 PROJECTS/<Brief Name>/`.
- Applies [[Brief Template Compliance]] — Projects and Programs must include required sections (Outcome, Why This Matters, Next Actions, Waiting For, Log, etc.).
- Applies [[Linking Conventions]] (Hard-Coded Reference Rule) — all note references in the brief and all cross-link edits use `[[wikilinks]]`.
- Applies [[Graph-Ready Conventions]] — frontmatter relationships (`depends_on`, `feeds`, `learns_from`, `parent`, `owner`) are wikilink arrays, never inline Dataview.
- Applies [[Dual-Hierarchy Model]] — Step 3 project-vs-program classification uses the >12-month duration test.
- Assumes PARA structure for vault search in Steps 1b and 7a (`01 PROJECTS/`, `02 AREAS/`, `03 REFERENCE/`, `00 HUB/`).
- Sibling-skill rule: updates to the output convention (folder layout, frontmatter fields, Continuation Prompt format) must be mirrored in the `/clean-projects` auditor — see Rules section.

### Does NOT Require

- No MCPs (no Granola, Slack, Google Workspace, SQL Server, etc.).
- No CLIs (no gws, gh, accli).
- No desktop apps.
- No external services or API keys.
- No Python or Node packages.
- No plugins.

## Merge Mode (invoke when consolidating two existing project briefs)

When two existing project briefs are recognized as duplicates or as sibling sub-projects that should consolidate into one, follow this 6-step canonical pattern. The skill itself doesn't currently expose a `--merge` flag — invoke this pattern manually when the user directs ("merge these two") and ALWAYS surface contradictions first per the user's guard.

### Step M1 — Surface contradictions BEFORE merge

Read both briefs. Build a comparison matrix across these fields: **owner / co_owner**, **frontmatter convention** (current vs legacy fields), **scope** (narrow vs broad), **Next Actions**, **Waiting For**, **Continuation Prompt freshness**, **Plan structure**, **status + priority**, **frontmatter description**, **tags**, **depends_on / feeds / learns_from arrays**. Classify each row as ⚠ REAL contradiction (must resolve) vs ✓ NON-contradictory (narrower/broader scope of same idea).

Present the matrix to the user. Surface the structural decision (which folder/name is the merge target; whether to nest siblings; etc.) as a separate question. Do NOT execute moves until the contradictions are explicitly resolved.

### Step M2 — Decide merge direction + ask the structural questions

Use AskUserQuestion to lock:
- **Merge target** — which folder name + path is the final home? Default: the cleaner/shorter name OR the one with the freshest Continuation Prompt + broader scope.
- **Owner / co_owner resolution** — restore co-owner if dropped; reconcile mismatched ownership.
- **Filesystem nesting** — should sub-project folders nest inside the merge target's folder, or stay flat at `01 PROJECTS/`? Trade-off: nesting clarifies topology but `/clean-projects` glob patterns must tolerate nested briefs (most globs are `**/*.md` so this is OK).
- **Related-projects consolidation** — defer to a separate scan brief, OR execute inline if scope is small.

### Step M3 — Filesystem operation (archive + move + nest)

Execute via Bash `mv`:
1. Archive the non-target brief: `mv "01 PROJECTS/<non-target>/<non-target>.md" "04 ARCHIVES/CLOSED PROJECTS/<non-target> (pre-merge YYYY-MM-DD).md"`
2. **Stamp pre-archive metadata FIRST** per [[Archive Protection]] edit-then-move ordering: set `status: archived-merged`, `merged_into: "[[<target>]]"`, `archived: YYYY-MM-DD` in the file BEFORE the `mv` to archives (the `h1-vault-protection.sh` hook blocks post-move edits inside `04 ARCHIVES/`).
3. If the merge target has a longer/different folder name, move its contents into the chosen target folder.
4. Move research/ + nested sub-project folders + other artifacts into the merged folder.
5. Remove the now-empty old folder via `rmdir`.

### Step M4 — Content absorption in the merged brief

Add a `### MERGE: [[<non-target>]] absorbed YYYY-MM-DD` subsection in the merged brief's `## Working Notes` capturing:
- **Scope inherited** — what the non-target brief owned (sections, locked decisions, NAs, milestones).
- **Locked decisions preserved as input** — verbatim or summarized.
- **Open gaps deferred** — items the non-target brief hadn't resolved; mark as input to next planning pass (e.g., a `/deep-planning` rebuild) rather than continuation work.
- **Reasoning for merge** — why separation was creating a failure mode (the user's "design fear" framing if applicable).

### Step M5 — Frontmatter cascade

Update the merged brief's frontmatter:
- Drop self-reference from `depends_on` (if the non-target brief was previously a sub-project reference).
- Migrate non-target's `learns_from` / `feeds` entries that aren't already present.
- Update `description:` to reflect the consolidated scope.
- Standardize on current frontmatter convention (drop legacy `parent_program`, `area`, `sub_area`, `related_projects`, `relates_to` if mixed in).
- Update H1 if the title changed.

### Step M6 — Cross-link redirect + Sub-Project Index update

Find all active vault references to `[[<non-target>]]` excluding `04 ARCHIVES/`, daily notes, lessons.md, reports, Log entries (preserve historical truth). Replace with `[[<target>]]`. Strategy:
- Use `replace_all=true` on files with ALL-active references (no historical Log entries to preserve).
- Use surgical edits on files with mixed active + historical references.
- In the relevant SUB-PROJECT-INDEX tables (parent program, Program Hierarchy, Area MOC), mark the merged row with strikethrough notation: `~~[[<non-target>]]~~ → [[<target>]] (merged YYYY-MM-DD)`.
- Append a Log entry on the merged brief AND the parent program brief documenting the merge.

### Rules for Merge Mode

- **Surface contradictions BEFORE any move** — per the user's explicit guard. No merge proceeds without surfacing.
- **One merge at a time** — batch merges lose nuance. Sequential per the 6-step pattern.
- **Preserve historical Log entries** — never rewrite retroactively. Per [[Archive Protection]] + temporal-truth convention.
- **Stamp metadata before move-to-archive** — per [[Archive Protection]] edit-then-move ordering. The hook blocks post-move edits silently.
- **Cascade redirect preserves graph integrity** — replace_all on uniform-active files, surgical on mixed. Both Edit and replace_all need a fresh Read after Bash `mv` invalidates the path cache.

### Evidence (2026-05-12)

This pattern was applied twice in one session:
1. **[[Bufalinda Vault]] → [[Repositorio Central del Estado de Bufalinda]]** — sibling sub-project absorbed; design fear ("unmaintainable central repository") shared between both; merge eliminated "two systems both claiming single source of truth" failure mode. Hook blocked post-move stamp; intent captured upstream in merge target's Working Notes "MERGE:" section.
2. **[[Bufalinda AI Brain V0 Implementation]] (umbrella) → [[Bufalinda AI Brain V0]] (runtime)** — duplicate umbrella + runtime briefs; restored Manuel as `co_owner:`; absorbed runtime brief's U1-U6 phases + 12-row decisions table reference + project-specific AI Context into merged brief's Working Notes "Runtime Track" subsection; nested 4 newly-spawned sub-projects into merged folder.

Both merges followed the 6-step pattern exactly. The 2026-05-12 session also produced a Scan Brief at `00 HUB/00 INBOX/2026-05-12 - Bufalinda AI Consolidation Scan Brief.md` that codifies a Phase 3 spec mirroring this Merge Mode for the next focused consolidation session.

## Related Skills
- [[decision-brief]] — when the project has open architectural decisions that need structured analysis before implementation
- [[clean-projects]] — downstream auditor that must stay in sync with this skill's output convention (see Rules)
- [[find-related]] — Step 1b.5 callee; surfaces related prior reading from READ_REVIEW + REFERENCE + wikis as a checkbox manifest, applied in Step 6d step 6 to the brief's `### Related Reading` subsection
- [[close-day]] — its `creation-routing.md` Task category enforces the same "fold into existing brief" pattern at a different entry point (daily-signal routing). Step 1.5d of this skill is the `/project-create`-entry analogue: same discipline, same edit shape (Direct file edit, no skill invocation), different signal source.
