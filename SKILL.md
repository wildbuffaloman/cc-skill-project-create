---
name: project-create
version: "0.0.1"description: Create a Project or Program brief through research, interactive Q&A, template application, vault linking, and INBOX delivery for review. Converts inbox notes, ideas, or raw topics into fully scoped briefs.
user-invocable: true
argument-hint: "note path, note title in INBOX, or topic description"
---

Create a Project Brief or Program Brief through a structured interactive process — research the input, run a guided Q&A to scope the initiative, decide project vs. program, apply the correct template, link to relevant vault notes and external references, and deliver to INBOX for review.

## Philosophy

Every idea deserves a fair shot at becoming a project — but only after it has been properly scoped. This skill applies the vault's project scaffold requirements: Outcome, Plan with 3-7 milestones, Actions with next 10, Log with proof of motion. If it cannot fit that structure, it belongs in incubation, not active projects.

The skill is conversational — it gathers context through research and Q&A before writing anything. The user drives scope decisions; the AI provides structure, research, and vault connections.

Pipeline: `Input -> Research -> Q&A -> Classify -> Template -> Link -> Deliver -> Cross-Link`

## Vault Exception

This skill may:
- Create new files in `00 HUB/00 INBOX/` — the project/program brief and any companion files
- Read files anywhere in the vault for research and linking
- **Modify existing vault files only for cross-linking** — inserting wikilinks to the new brief into related notes (Step 6). All proposed edits must be listed and confirmed by the user before execution. No other modifications to existing files are permitted.

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

**1c — Web research** — only if the topic benefits from external knowledge
Use WebSearch to find:
- Best practices, frameworks, or methodologies relevant to the initiative
- Industry benchmarks or reference points
- Common pitfalls and critical success factors

**1d — Present research findings**
Share a concise summary of what was found — vault connections, relevant context, and external insights. This grounds the Q&A in real context.

### Step 2: Interactive Q&A — Scoping

Run an adaptive Q&A to fill in all sections of the project brief. Use AskUserQuestion with well-informed options based on Step 1 research.

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

### Step 4: Build the Brief

Apply the correct template based on the decision:

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
- Working Notes should contain the research findings, frameworks, and context gathered during the process — this is the institutional memory
- Key Resources must include actual vault wikilinks found during research, not empty placeholders
- Milestones should be 3-7, each with a target date or timeframe
- Tags should include the area, topic keywords, and any relevant programs
- Key Resources subsections are: **Evergreen Notes** (distilled insights), **Reference Materials** (source docs, links), **People / Collaborators**. Never use "Atomic Notes" — the vault term is Evergreen Notes.
- **Table wikilink convention:** Embed `[[wikilinks]]` directly in the name/title column of tables — never use a separate "Link" column. Applies to Sub-Project Index, Milestones, AI Ecosystem tables, and any other table with named entities.

### Step 5: Link and Deliver

**5a — Insert vault links**
For every vault note identified as relevant in Step 1, insert proper `[[wikilinks]]` in the appropriate section — Key Resources for reference materials, Working Notes for context, Dependencies for blockers.

**5b — Insert external references**
For any external resources found in web research, add as markdown links in Key Resources or Working Notes.

**5c — Determine destination and status**

For **Projects**, ask the user for the `status:` value (this goes in frontmatter):
- `active` — time is scheduled for this project in the coming week (counts toward 7-project WIP cap) → destination: `01 PROJECTS/`
- `delegated` — someone else leads, Alberto only tracks progress (separate track, no hard cap) → destination: `01 PROJECTS/`
- `incubating` — committed project, not currently being worked on → destination: `01 PROJECTS/`
- `blocked` — waiting on a specific person or dependency to unblock → destination: `01 PROJECTS/`
- `someday-maybe` — not committed, a possibility → destination: `01 PROJECTS/04 Someday-Maybe/`

For **Programs**: destination is `02 AREAS/AREA_FOLDER/` under the relevant area (unchanged).

**5d — Save to INBOX**
Save the completed brief to `00 HUB/00 INBOX/` with the note title as filename.

**5e — Present summary**
Tell the user:
- Where the brief was saved
- What vault notes were linked
- The recommended destination folder and status
- Any open questions or items flagged for their attention
- Remind them to review and move the brief to the approved destination folder

### Step 6: Cross-Link Scan

**This step is mandatory.** After the brief is saved, run a comprehensive cross-link scan to strengthen the knowledge graph bidirectionally.

**6a — Comprehensive vault scan**
Search across all active vault areas for notes related to the new brief's topic, area, and keywords:
- `01 PROJECTS/` — sibling projects, sub-projects of the same parent program, projects with overlapping dependencies or feeds
- `02 AREAS/` — Area MOCs, parent program brief, sibling programs, related programs in adjacent areas
- `03 REFERENCE/` — reference notes, reading notes, templates, and guides related to the topic
- `00 HUB/` — Hub MOCs, other INBOX items that relate

For each candidate note, check both:
- **Frontmatter relationships** — `depends_on`, `feeds`, `learns_from`, `parent` fields that should reference the new brief
- **Body content** — sections like Key Resources, Working Notes, Sub-Project Index, or Related Notes where a wikilink to the new brief would add value

**6b — Build cross-link manifest**
Compile a list of proposed edits, organized by note. For each proposed edit, specify:
- The **target file** path and name
- The **section** where the link should be inserted
- The **link type** — is this a frontmatter relationship (e.g., adding to `depends_on` array) or a body wikilink?
- **Why** — a brief justification for why this link adds value (semantic relationship: supports, depends on, feeds into, sibling of, etc.)

Also identify any **frontmatter relationship updates** needed on the new brief itself (e.g., if the scan reveals a `depends_on` or `feeds` relationship that wasn't captured during Steps 1-5).

**6c — Present manifest for user approval**
Display the full cross-link manifest to the user. Group edits by target file. Include a count of total proposed edits and affected files. The user may:
- Approve all
- Approve selectively (remove specific edits)
- Skip the cross-linking entirely

**6d — Execute approved edits**
For each approved edit:
- Read the target file
- Insert the wikilink in the specified section, matching existing formatting
- For frontmatter array fields, append the new wikilink to the existing array
- For body sections, add the wikilink in the most natural position (e.g., as a new list item, table row, or inline reference)
- Preserve all existing content — surgical insertion only

If the scan also identified updates needed on the new brief itself, apply those as well (updating the file saved in Step 5d).

**6e — Report results**
Summarize what was linked:
- Number of files updated
- Total wikilinks inserted
- Any edits that failed (e.g., file structure was unexpected) with explanation
- The new brief's updated graph connections

## Rules

- Pause after each major step to get user input — this is a conversation, not a monologue
- Use AskUserQuestion for structured choices — batch up to 4 questions per call
- Only include vault notes that are clearly relevant — when in doubt, leave it out
- Use exact note names without .md extension inside wikilinks
- Do not modify existing vault files except during Step 6 (Cross-Link Scan), where approved wikilink insertions are permitted
- Avoid empty placeholder wikilinks — omit sections with no content rather than leaving blanks
- Web research is optional — skip it for purely internal/organizational projects where external knowledge adds nothing
- If the input note already contains substantial content, pre-fill Q&A answers from it and confirm with the user rather than re-asking
- The vault working directory is resolved at runtime from the CLAUDE.md chain
- Projects go to `01 PROJECTS/` (flat) or `01 PROJECTS/04 Someday-Maybe/` for someday-maybe status. Status is tracked via the `status:` frontmatter field, not folder location.
- Programs go to `02 AREAS/` under the relevant area folder
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

## Related Skills
- [[decision-brief]] — when the project has open architectural decisions that need structured analysis before implementation
