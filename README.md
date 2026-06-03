KIRO PROMPT — Integrate the meridian-frontend-template BMAD improvements into ng-portals
========================================================================================

# CONTEXT

I'm working inside the bank's closed environment on two repositories on this workstation:

1. `poalimad` — a BMAD template repository. This folder contains the bank-customized BMAD template plus the 2026-06 standards restructure we authored. Treat this as the **reference source** of the improvements I want to bring into the bank's real repo.

2. `ng-portals` — the bank's actual frontend monorepo cloned from Bitbucket. It already contains BMAD installed inside it, and the bank engineers have likely customized it over time. Treat this as the **target** where the improvements must land.

I need you to merge the improvements from `poalimad` into `ng-portals` **without overwriting any bank customization that's already there**. The merge philosophy is:

> **Preserve everything ng-portals already has. Insert only the items we contributed. Stop and surface to me on any ambiguity.**

---

# WHAT YOU'LL FIND INSIDE THE poalimad FOLDER

The `poalimad` folder contains a bank-customized thin-native BMAD template. The branch I cloned includes a 2026-06 restructure with seven conceptual contributions. Familiarize yourself with these before doing anything else:

1. **Four-document owner-aligned standards split** under `docs/standards/` —
   - `regulation.md` (REG-* rules, owned by Legal / Compliance)
   - `architecture-security.md` (ARCH-SEC-* rules, owned by Security Architecture)
   - `code-security.md` (CODE-SEC-* rules, owned by Security Engineering)
   - `code-quality.md` (CODE-QUAL-* rules, owned by Engineering Excellence)

Each agent loads only the documents its role requires.

2. **A new top-level Custom Skill `bmad-security-design`** at `_bmad/custom/skills/bmad-security-design/` (three files: `SKILL.md`, `customize.toml`, `threat-modeling-examples.md`). The skill produces `threat-model.md` as a sibling artifact to `architecture.md`, using a STRIDE-FE shape (Spoofing, Tampering, Information Disclosure, Elevation; Repudiation and DoS deliberately dropped as backend-layer concerns). The `SKILL.md` includes a full "Threat-model lifecycle" section.

3. **`threat-model.md` as a HARD-REQUIRED artifact** at the implementation-readiness gate. The bank workflow `_bmad/custom/bmad-check-implementation-readiness.toml` CHECK 2B FAILs the gate when `threat-model.md` is missing at the active scope and surfaces a CONCERN when it is stale (architecture.md modified after threat-model.md).

4. **Per-role document loading**, wired into the six bank-persona agent overrides under `_bmad/custom/bmad-agent-*.toml`:
   - Galit (PM) loads `regulation.md`
   - Yossi (Architect) loads `architecture-security.md`
   - Shraga (Dev / Builder) loads `code-security.md` + `code-quality.md`
   - Lenny (UX) loads `regulation.md` + `code-security.md`
   - Dvora (Analyst) loads `regulation.md`
   - Paige (Tech Writer) loads `regulation.md` + `code-security.md`
   All six also load `docs/bank-wide-standards.md` as the universal index.

5. **Option X — mid-workflow dispatch from `bmad-create-architecture`** into `bmad-security-design`. The workflow's `activation_steps_append` runs the skill ONCE between architecture draft and finalization. The dead-loop guard `BMAD_CASCADE_FROM=security-design` is intentionally absent. The agent itself does not dispatch the skill — the workflow does.

6. **Audit-trail commit policy** at `docs/audit-trail-conventions.md`, defining which `_bmad-output/` paths are committed in consumer repos (PRDs, architecture, threat-model, UX specs, epics, per-story `## Review` verdicts) so code-review outcomes are no longer ephemeral.

7. **PRD `## Regulation` section discipline** — `docs/bank-prd-conventions.md` lists Regulation as section #7 of a 10-section PRD layout, `docs/bank-prd-template.md` includes the section template, and the PRD workflows (`bmad-create-prd.toml`, `bmad-edit-prd.toml`, `bmad-validate-prd.toml`) are re-wired to load `regulation.md`. `bmad-validate-prd` raises a CONCERN if the section is missing.

Bank-wide conventions (Angular, NX, `@poalim/*` aliases, Hebrew RTL, Teudat Zehut PII handling, the six persona names) are preserved as project-agnostic content. Family-Banking-specific content was deliberately removed — do not propagate any `CHILD-*` rule IDs or `family/parent` examples even if you find them inside the poalimad folder by accident.

---

# THINGS TO PAY ATTENTION TO

- The four standards documents contain `[BANK-PLACEHOLDER]`, `[REG-PLACEHOLDER]`, `[ARCH-SEC-PLACEHOLDER]`, `[CODE-SEC-PLACEHOLDER]`, and `[CODE-QUAL-PLACEHOLDER]` tokens. These are intentional. **Do not substitute them** — the bank security/legal/engineering committees fill them post-merge.
- The improvements are **frontend-only** by design. Do not introduce backend, infra, or CI rules.
- The `_bmad/core/`, `_bmad/bmm/`, `_bmad/bmb/`, and `_bmad/scripts/` trees are upstream-managed BMAD. **Never edit them on either repo.** Customization lives only in `_bmad/custom/` and `docs/`.
- ng-portals is a large monorepo. The BMAD installation inside it may be at the repository root or scoped to a subdirectory. You must locate it before doing anything else.

---

# WHAT I NEED YOU TO DO

Execute the steps below **in order**. Do not write any file or run any merge until I explicitly approve the plan you produce in Step 4.

## Step 1 — Inventory the poalimad source

Walk the `poalimad` folder and produce an inventory of the customization surface:

- List every file under `_bmad/custom/`, including the skill folder `_bmad/custom/skills/bmad-security-design/`.
- List every file under `docs/standards/`.
- List `docs/audit-trail-conventions.md` and the convention prose files: `docs/bank-wide-standards.md`, `docs/bank-prd-conventions.md`, `docs/bank-prd-template.md`, `docs/bank-architecture-conventions.md`, `docs/per-scope-artifact-conventions.md`.
- List the root-level `BANK-CHANGELOG.md` and `BANK-CUSTOMIZATIONS.md`.

For each file, note: file size, last modified date, whether it appears to be one of the 2026-06 additions, and a one-line description of what it does. Use the seven conceptual contributions above as your reference for "what we built".

## Step 2 — Locate the BMAD installation inside ng-portals

Search ng-portals for the BMAD root. Likely signals: an `_bmad/` directory, a `_bmad/core/` tree, a `_bmad/custom/` tree, a `bmad-` prefix in skill folders, or any of `BANK-CHANGELOG.md` / `BANK-CUSTOMIZATIONS.md` at the repo root.

Report back: the path to the BMAD root inside ng-portals, the BMAD version (look for a version file, `package.json`, or upstream sync notes), and a short tree showing what already exists under `_bmad/custom/` and `docs/` (standards docs, conventions, skills).

## Step 3 — Compute the semantic delta

For each of the 28 candidate files from Step 1, classify against ng-portals' state from Step 2 using these four classes:

- **A — Already present, byte-identical** → no action needed.
- **B — Already present, semantically equivalent** (same intent, possibly different wording or order) → no action needed, but flag for me so I can decide whether to harmonize wording.
- **C — Absent or substantively different** — this is where the merge happens. For "new file" candidates (`docs/standards/*.md`, `_bmad/custom/bmad-check-implementation-readiness.toml`, `_bmad/custom/bmad-code-review.toml`, the `bmad-security-design` skill, `docs/audit-trail-conventions.md`, `BANK-CHANGELOG.md`, `BANK-CUSTOMIZATIONS.md`), the action is ADD if ng-portals has nothing at that path. For "modified file" candidates (the six agent overrides, the five workflow overrides, the five convention docs), the action is MERGE — apply only the discrete semantic additions from the poalimad version, preserving every line ng-portals already has.
- **D — Conflict** (ng-portals has its own version of an ADD target, or ng-portals' modified file already contains a similar-but-different addition for the same concept) → STOP and report. Do not merge, do not overwrite.

For the MERGE class (Class C on modified files), break the change into atomic items where possible. For each item, give me: the exact text to insert, the location in ng-portals' file where it should go (the surrounding anchor — e.g., "inside `persistent_facts = [ ... ]` array, alongside the existing `file:` entries"), a grep-string I can use to detect whether the item is already there, and the rationale tied to one of the seven conceptual contributions.

## Step 4 — Produce a merge plan, DO NOT APPLY YET

Output a single structured plan with three sections:

1. **NEW FILES to ADD** — table of path + source location + size + a one-line rationale.
2. **MODIFIED FILES to MERGE** — for each, the list of discrete items (each with insertion location, exact text, detection grep, and rationale).
3. **CONFLICTS** — files where ng-portals has its own conflicting content that needs my decision.

Show me this plan and stop. Do not write any file, do not stage anything, do not modify ng-portals. Wait for my approval.

## Step 5 — Apply the plan ONLY after I approve

Once I say "go", create a new feature branch off the current ng-portals working branch. Apply the plan: write new files for ADDs, insert atomic items at the specified locations for MERGEs, leave conflicts untouched. After applying, run a verification sweep: every TOML must parse, every `[*-PLACEHOLDER]` token must remain literal, no file outside the merge plan may be modified, and the BMAD-managed trees must be untouched.

Report the resulting `git diff --stat`, the verification results, and any anomalies. Do not push or merge. Wait for human review.

---

# OUTPUT FORMAT FOR STEP 4 (the plan I'll review)

Use clear markdown headings: `# Step 1 inventory`, `# Step 2 ng-portals BMAD state`, `# Step 3 classification`, `# Step 4 plan`. For Step 4, use tables for ADDs and detailed item-by-item blocks for MERGEs. For each conflict, give me one paragraph explaining what's different and what your suggested resolution would be — but do not act on the suggestion until I confirm.

# IMPORTANT RULES

- Preserve every customization the bank already has in ng-portals. Never overwrite.
- Never touch `_bmad/core/`, `_bmad/bmm/`, `_bmad/bmb/`, `_bmad/scripts/`.
- Never substitute `[*-PLACEHOLDER]` tokens — they're owned by bank committees.
- Never run `npx bmad-method install`, `npm install`, or any network-dependent command. The environment is offline and this is template-level work only.
- Never push the feature branch automatically. Wait for human review and approval.
- On any ambiguity, stop and ask. The merge philosophy is preserve-first, insert-only, conflict-halts.

When you're ready, begin with Step 1.
