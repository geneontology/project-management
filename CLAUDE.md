# GO Project Management

This repository is the Gene Ontology Consortium's project management hub. It contains no code. Each GitHub issue represents a project charter/proposal, and each is linked to a corresponding GitHub Organization Project board for task tracking.

This document is intended for use by both human users and Claude (via Claude Code) to support project management operations using the `gh` CLI.

## Important: This Is Not a Normal Code Repository

Issues and PRs in this repo have special meaning. **Do not** use standard development workflows here:
- **Issues are project charters**, not bug reports or feature requests. Only create issues to propose new projects, using the issue template.
- **PRs should only modify repo infrastructure** (templates, this file, README, etc.), not deliver project work. Project work happens in other geneontology repos.
- **Do not close issues casually** — closing an issue means the project is complete or withdrawn.

## Repository Structure

- `.github/ISSUE_TEMPLATE/proposed-project-2.md` — The issue template for new project proposals
- `CLAUDE.md` — This file; operational guide for Claude-mediated project management
- Issues in this repo — Project charters with scope, SWOT, personnel, approvals
- `geneontology` org project boards — Task-level tracking (issues from repos like `noctua`, `minerva`, `gocam-py`, etc.)

## Key Concepts

### The Three Layers

1. **Project-management issue** (this repo): The "charter" — description, scope, motivations, SWOT, personnel, approval status. Created from the issue template.
2. **Org project board** (`https://github.com/orgs/geneontology/projects/<N>`): The task tracker — contains work items (issues/PRs) from various geneontology repos.
3. **Meta-project 108** (`https://github.com/orgs/geneontology/projects/108`): The portfolio dashboard — contains all project-management issues as items with lifecycle status, priority, PI/PO/TL assignments, and dates.

### Cross-Linking

Each project has two-way manual links:
- The **issue body** contains a "Project link" URL pointing to the org project board
- The **project board's Description and Readme** contain a URL back to the project-management issue
- The issue is also an **item in project 108** for portfolio-level tracking

### Approval Workflow (Labels)

New issues are auto-assigned 6 red labels representing required approvals:
- `Needs PI` — Principal Investigator
- `Needs PO` — Project Owner
- `Needs TL` — Technical Lead
- `Needs tech doc` — Technical documentation/specs
- `Needs PM approval` — Project Manager approval
- `Needs LA approval` — Lead Architect approval

Labels are removed as approvals are obtained. The green `Ready` label is added when all approvals are complete.

### Project 108 Status Values (Lifecycle)

Items in project 108 have a Status field tracking lifecycle stage:
- **Creation (initial requirements document)** — Brand new, defining requirements
- **Requirement cycle (Planning/Technical)** — In planning phase
- **Priority (project triage)** — Being evaluated for prioritization
- **Resourcing (waiting for capacity)** — Approved but blocked on staffing
- **Hopper** — Approved and resourced, waiting to start
- **Active** — Currently being worked on
- **Ongoing** — Continuous/maintenance work (no end date)
- **Collaborations** — External collaboration projects
- **Deprioritized or addressed by other projects** — Shelved
- **Complete - 2023 and earlier**, **Complete - 2024**, **Complete 2025**, **Complete 2026** — Done

### Project 108 Custom Fields

- **PI**, **PO**, **TL** — Principal Investigator, Project Owner, Technical Lead
- **Priority** — Priority level
- **Team members**, **Members** — Personnel
- **Next review date** — When project is next reviewed
- **Estimated delivery date** — Target completion
- **Date started** — When work began

## Common Operations

### Getting a Status Report

Use `gh` CLI to query project 108 for portfolio-level status:
```bash
# All items with status
gh project item-list 108 --owner geneontology --limit 200 --format json

# View a specific project-management issue
gh issue view <NUMBER> --repo geneontology/project-management
```

### Examining a Specific Project

1. Read the project-management issue for the charter: `gh issue view <N> --repo geneontology/project-management`
2. Find the project board number from the "Project link" in the issue body
3. View the project board: `gh project view <N> --owner geneontology`
4. List its work items: `gh project item-list <N> --owner geneontology --limit 50`

### Creating a New Project

The full lifecycle for creating a new project:

1. **Create the org project board:**
   ```bash
   gh project create --owner geneontology --title "<Project Title>" --format json
   ```
   Note the project number and URL from the output.

   **Important:** The board should be a Kanban board with the default columns (Todo, In Progress, Done), unless otherwise specified. The `gh` CLI creates boards in table layout by default and the GitHub API does not support changing the view layout. **The user must manually switch the layout to "Board"** in the GitHub UI after creation: click the gear icon ("View options") next to the search bar, then select "Board" under the Layout section. Remind the user to do this.

2. **Create the issue in this repo** using the template, with the project link filled in:
   ```bash
   gh issue create --repo geneontology/project-management \
     --title "<Project Title>" \
     --body "$(cat <<'EOF'
   #### Project link
   https://github.com/orgs/geneontology/projects/<N>

   #### Project description
   ...
   EOF
   )"
   ```
   Note: the template's default labels (Needs PI, Needs PO, etc.) are applied automatically when using the template via the GitHub UI, but must be added manually via `--label` flags when using the CLI.

3. **Set the project board's description and readme** to link back to the issue:
   ```bash
   gh project edit <N> --owner geneontology \
     --description "https://github.com/geneontology/project-management/issues/<ISSUE>" \
     --readme "https://github.com/geneontology/project-management/issues/<ISSUE>"
   ```

4. **Add the issue to project 108** and set its initial Status:
   ```bash
   # Add the issue
   gh project item-add 108 --owner geneontology \
     --url https://github.com/geneontology/project-management/issues/<ISSUE> --format json
   # Note the item ID from the output, then set Status
   gh project item-edit --project-id PVT_kwDOAHZEs84AHENv \
     --id <ITEM_ID> \
     --field-id PVTSSF_lADOAHZEs84AHENvzgEEnEw \
     --single-select-option-id af8edfb0  # "Creation (initial requirements document)"
   ```

### Querying by Status

```bash
# Get all active projects
gh project item-list 108 --owner geneontology --limit 200 --format json | \
  python3 -c "import json,sys; [print(i['title']) for i in json.load(sys.stdin)['items'] if i.get('status')=='Active']"
```

### Closing/Completing a Project

1. Update the item's Status in project 108 to the appropriate "Complete" value
2. Close the project-management issue
3. Close the org project board if all work items are done

## GitHub CLI Setup and Permissions

The `gh` CLI is the primary tool for both users and Claude to interact with this system. All operations described in this document have been tested and verified to work.

### Required Scopes

| Scope | Needed for | How to add |
|-------|-----------|------------|
| `repo` | Creating/editing/closing issues, managing labels | Usually present by default |
| `read:org` | Listing org members and teams | Usually present by default |
| `project` | All project board operations (create, edit, delete, add/edit/delete items, set fields). Also covers read access. | `gh auth refresh -s project` |

If you only need read access to projects (e.g., generating reports), `read:project` is sufficient. For full lifecycle management, the `project` scope is required.

### Adding Missing Scopes

If you encounter a scope error, the user must run the auth refresh interactively (Claude cannot do this):
```bash
gh auth refresh -s project
```
This opens a browser-based authorization flow. The scope is persistent once granted.

### Verified Operations

All of the following have been tested with the `repo` + `project` scopes:
- **Read**: `gh project list/view/item-list/field-list`, `gh issue list/view`, `gh api graphql` (project queries)
- **Write**: `gh project create/edit/delete`, `gh project item-add/item-edit/item-delete`, `gh issue create/edit/close`

### Project 108 Field IDs and Option IDs

Setting fields on project 108 items requires internal IDs. These are stable unless fields are recreated.

**Project 108 ID**: `PVT_kwDOAHZEs84AHENv`

**Status field** (ID: `PVTSSF_lADOAHZEs84AHENvzgEEnEw`):

| Status | Option ID |
|--------|-----------|
| Creation (initial requirements document) | `af8edfb0` |
| Requirement cycle (Planning/Technical) | `98236657` |
| Priority (project triage) | `a884febf` |
| Resourcing (waiting for capacity) | `47fc9ee4` |
| Hopper | `29171843` |
| Active | `18138791` |
| Ongoing | `e357eb6f` |
| Collaborations | `dd366792` |
| Deprioritized or addressed by other projects | `d2ad0f3a` |
| Complete 2026 | `7298b845` |
| Complete 2025 | `24400c1e` |
| Complete - 2024 | `5c1c89dc` |
| Complete - 2023 and earlier | `ee2fa35a` |

**Priority field** (ID: `PVTSSF_lADOAHZEs84AHENvzg2K5ms`):

| Priority | Option ID |
|----------|-----------|
| 1 | `49a10fed` |
| 2 | `bf00517d` |
| 3 | `227c160a` |
| 99 back burner project | `fcf1ff3b` |

**Team members field** (ID: `PVTSSF_lADOAHZEs84AHENvzg2K6Bg`):

| Member | Option ID |
|--------|-----------|
| Dustin | `57924f8b` |
| Tremayne | `5015b519` |
| Anushya | `c8a0c711` |
| Sierra | `56c78ae0` |
| Patrick | `325fcafd` |
| Jim | `156a30d0` |

**Example: setting Status on a project 108 item:**
```bash
gh project item-edit --project-id PVT_kwDOAHZEs84AHENv \
  --id <ITEM_ID> \
  --field-id PVTSSF_lADOAHZEs84AHENvzgEEnEw \
  --single-select-option-id 18138791  # Active
```

To get an item's ID, query project 108 items as JSON and match by title or issue number.

If these IDs ever become stale, re-query them:
```bash
gh api graphql -f query='{
  organization(login: "geneontology") {
    projectV2(number: 108) {
      field(name: "Status") {
        ... on ProjectV2SingleSelectField { id options { id name } }
      }
    }
  }
}'
```

## Key People

- **pgaudet** (Pascale Gaudet) — Default assignee for all project proposals, project manager
- **kltm** (Seth Carbon) — Lead architect, frequent project author

## Cleanliness Criteria

These are the ideals we strive for. Not every project or item will meet all of them at all times, but deviations should be noted in reports and audits — quietly, as observations, not alarms.

### Linkage Integrity

- **Every project-management issue should have a corresponding org project board**, and vice versa. The issue body should contain the project board URL, and the project board's description/readme should link back to the issue.
- **Every project-management issue should be an item in project 108** with a valid lifecycle Status set.
- **Issue numbers and project board numbers are not 1:1** (the org has many project boards beyond those tracked here). This is expected, but every project-management issue should have exactly one matching board.

### Ticket Discipline

- **Every PR in any geneontology repo should be tied to a ticket (issue)**. Orphan PRs — those not linked to any issue — represent untracked work.
- **Every ticket (issue) that represents project work should belong to a project board**. Issues floating outside of any project board are invisible to project tracking.
- **Every project board item should have a Status set**. Items without status are ambiguous and hard to report on.

### Lifecycle Hygiene

- **Every item in project 108 should have a current, accurate Status**. Stale statuses (e.g., "Active" on a project with no recent activity) obscure the true state of the portfolio.
- **Completed projects should be closed**: the project-management issue should be closed, the project 108 status should reflect the completion year, and ideally the org project board should be closed if all work items are done.
- **Approval labels should reflect reality**: a project-management issue should not carry "Needs..." labels for approvals that have already been obtained, nor should it have the "Ready" label if approvals are still pending.

### Reporting Guidance

When generating status reports or audits, check for and quietly note any deviations from the above criteria. For example:
- Issues missing a project board link or not present in project 108
- Project boards whose description does not link back to a project-management issue
- PRs not associated with any issue
- Issues not assigned to any project board
- Items in project 108 with no Status or stale Status
- Mismatches between approval labels and actual project state

These observations help maintain awareness without requiring immediate action on every deviation.

## Notes

- The link between issues and project boards is manual (URLs in text), not a native GitHub metadata association
- When generating reports, always query project 108 for the authoritative lifecycle status
- Issue labels track approval state; project 108 Status tracks lifecycle state — these are complementary

## Alliance / AGR Yearly Progress Reporting

This repository also hosts working drafts for the GO Consortium's yearly contribution to the **Alliance of Genome Resources** NIH progress report. The final artifact lives in Google Drive (owned by the Alliance); sweeping, drafting, and iteration happen here as `draft-go-section-<year>.md` files.

### Context

- The Alliance is a separate project from the GOC but pays for some GO software time; many PIs are shared.
- GO contributions appear as one sub-section inside the report's "Specialists Team" section — alongside Textpresso, BLAST, JBrowse, AllianceMine, PAVI, DIOPT, DevOps, User Support, and others.
- The GO sub-section is typically one paragraph (~200-300 words). Some working groups in prior years have run as short as two sentences.
- Reporting window is roughly March→March each year.
- Paul Sternberg's four content categories (asked each year): GO Pipelines, Annotation tools, GO-CAM visualizations, and plans for GO term pages (specifically the round-trip linkage to Alliance gene pages).
- The NIH progress report is distinct from the Alliance's *Genetics* journal article (broader scope, different cadence) — do not conflate the two deliverables.

### Drive references

- 2025 report folder (completed): https://drive.google.com/drive/folders/14U9SYv3SCGHqp8agRVzcKTlPaSpvM7cr
- 2026 working doc: https://docs.google.com/document/d/1u0-fylXiFINT5avst3_EaLqSzFzXi9Y-
- Prior-year exemplars for idiom calibration: `Alliance Progress Report 2021.docx` / `2022.docx` (in Drive).

Before drafting, read the prior year's actual final report to calibrate length and tone. Working drafts and specialist-group outline docs are not a substitute.

### Audience and tone

- **Zero-knowledge reader.** Assume NIH program officers and Alliance admins have no GO vocabulary. Every paragraph should land without relying on insider terms like Noctua, Minerva, AmiGO, or GO-CAM as givens.
- **Narrative throughline.** Structure paragraphs to tell the story of where the year went, not to tick through Sternberg's four categories in order. The categories are *content anchors*, not a section layout.
- **No version numbers, no PR numbers, no casual repo/tool names.** But DO name first-class sub-projects that have their own publications or product identity — e.g., `dragon-ai-agent` (own publication at PMC11484368), GO-CAM Browser. Naming these credits the sub-project and signals scope.
- **Lean.** 3–4 short stories, not an exhaustive list.

### Sweep approach

- **Window**: roughly March 1 (prior year) through March 31 (current year).
- **Primary sweep**: `geneontology` org PRs/issues in window, focused on software deliverables (`go-api`, `go-fastapi`, `go-site`, `amigo`, `noctua`, `noctua-py`, `minerva`, `gocam-py`, `pathways2GO`, `wc-gocam-viz`, `web-components`, `go-cam-browser`). **Skip routine ontology/data churn** in `go-ontology` and `noctua-models`; only flag infra or tooling changes there.
- **Secondary sweep**: `alliance-genome` org — GO team members (kltm, dustine32, tmushayahama, sierra-moxon, cmungall, pgaudet, etc.) may have authored PRs into Alliance repos that constitute direct GO→Alliance contribution. A `geneontology`-only sweep will miss these.
- **AI-authored ontology edits**: search `geneontology/go-ontology` PRs authored by the `dragon-ai-agent` bot account. External analysis lives under `monarch-initiative/dragon-ai-results`.
- **Keep the detailed sweep inventory at the bottom of the draft** even when the polished paragraphs are short — it is reusable for the separate *Genetics* article and for internal reporting.

### Working convention

- Drafts live here as `draft-go-section-<year>.md` and are **gitignored** — they are transient local artifacts, not committed to the repo. The canonical version is in Drive.
- Iterate in the local draft; a human mirrors stable changes into the Drive working doc.
- **Writes to the Alliance Drive docs must be done manually by a human.** Claude and other automation tools are not authorized to edit, create, or otherwise modify the Alliance-owned Drive documents (yet) — even if the Drive MCP exposes write tools in the session. This applies to the working doc, the final GO sub-section doc, and anything else in the Alliance report folders. Read access is fine.
- At the end of each round, cross-check the local draft against the live Drive version — Alliance editors may make in-place changes that do not flow back (e.g., the 2026 Drive doc gained a Cloudflare point that was edited in directly).
