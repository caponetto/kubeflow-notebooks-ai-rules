# AI Rules for Kubeflow Notebooks

Guidelines and patterns for AI coding assistants working on the [Kubeflow Notebooks](https://github.com/kubeflow/notebooks) repository.

## 📖 Overview

This repository provides AI coding assistant configurations for Kubeflow Notebooks development.

**Why use this?** AI coding assistants (like Claude and Cursor) work better when they understand your project's conventions, patterns, and boundaries. These configuration files give AI tools the context they need to generate code that follows project standards.

Currently, it includes:

- `AGENTS.md` files: guidelines and patterns that AI tools read to understand how to write code for each module
- `CLAUDE.md`: symlink to the root `AGENTS.md` for Claude Code compatibility
- `skills/`: focused, short `SKILL.md` workflows that complement `AGENTS.md` policies

> [!NOTE]
>
> - This repository can be extended with other AI rules (e.g., Cursor skills, prompt templates) as needed.
> - Links between files are designed to work when symlinked into Kubeflow Notebooks, not within this repository.
> - **Future goal:** Move these AI configurations directly into Kubeflow Notebooks (requires community agreement).

## 🚀 Quick Start

### Installation

> [!IMPORTANT]
> This repository is compatible with the [`notebooks-v2`](https://github.com/kubeflow/notebooks/tree/notebooks-v2) branch of the Kubeflow Notebooks repository.

1. Clone this repository alongside Kubeflow Notebooks:

```bash
cd ~/dev
git clone git@github.com:caponetto/kubeflow-notebooks-ai-rules.git
git clone -b notebooks-v2 git@github.com:(kubeflow|your-fork)/notebooks.git kubeflow-notebooks
```

2. Run the install script to create symlinks in the Kubeflow Notebooks repo:

```bash
cd kubeflow-notebooks-ai-rules
./scripts/install.sh <path-to-kubeflow-notebooks>
```

This places `AGENTS.md` files in each module folder (e.g., `workspaces/frontend/AGENTS.md`) and symlinks all skills to `.agents/skills/` at the repo root following the [agentskills.io](https://agentskills.io/specification) open standard. AI tools auto-discover these files based on your working directory, so the agent automatically gets the relevant guidelines when you're working in that module.

> [!WARNING]
> Be careful not to commit the symlinked `AGENTS.md` files when pushing changes to the Kubeflow Notebooks repository.

### ✅ Verification

```bash
./scripts/check.sh <path-to-kubeflow-notebooks>
```

### 🗑️ Uninstall

```bash
./scripts/uninstall.sh <path-to-kubeflow-notebooks>
```

---

## 📁 Repository Structure

```
kubeflow-notebooks-ai-rules/
├── README.md
├── .gitignore
├── skills/                          # Focused tool-agnostic skills (agentskills.io standard)
│   ├── README.md                    # Skill index and usage model
│   └── <skill-name>/SKILL.md        # One focused workflow per skill (flat layout, name = dir)
├── agents/                          # AGENTS.md files
│   ├── mappings.conf                # Agent file mappings (used by scripts)
│   ├── AGENTS.md                    # Global policies
│   ├── CLAUDE.md -> AGENTS.md       # Claude Code compatibility
│   ├── backend/
│   │   └── AGENTS.md                # Backend guidelines + skill playbooks
│   ├── controller/
│   │   └── AGENTS.md                # Controller guidelines + skill playbooks
│   └── frontend/
│       ├── AGENTS.md                # Frontend guidelines + skill playbooks
│       └── cypress/
│           └── AGENTS.md            # Cypress guidelines + skill playbooks
└── scripts/
    ├── install.sh                   # Create symlinks
    ├── uninstall.sh                 # Remove symlinks
    └── check.sh                     # Verify symlinks
```

---

## 🤖 Agents

`AGENTS.md` files provide guidelines for AI coding assistants, including links to task-focused skill playbooks.

## 🧠 Skills

`skills/` contains short, focused `SKILL.md` files intended to be combined with `AGENTS.md`:

- `AGENTS.md`: stable policy, boundaries, and safety constraints
- `SKILL.md`: task-level workflows (for example, reconcile changes, API integration, flaky test triage)

This hybrid model keeps always-on constraints centralized while reducing prompt size for implementation-specific guidance.

| File                                | Purpose                                        |
| ----------------------------------- | ---------------------------------------------- |
| `agents/AGENTS.md`                  | 🌐 Global policies, safety rules, PR checklist |
| `agents/backend/AGENTS.md`          | ⚙️ Go backend server guidelines                |
| `agents/controller/AGENTS.md`       | ☸️ Kubernetes controller guidelines            |
| `agents/frontend/AGENTS.md`         | ⚛️ React/TypeScript guidelines                 |
| `agents/frontend/cypress/AGENTS.md` | 🧪 E2E testing guidelines                      |

### Referencing Guidelines

Reference the `AGENTS.md` files depending on the AI tool you use:

**🟣 Claude Code** - Automatically reads `CLAUDE.md` and `AGENTS.md` from the repo root.

**🔵 Cursor** - Supports `@` mentions to reference specific files:

```
@AGENTS.md @workspaces/frontend/AGENTS.md Create a new React component for...
```

### Writing Agent-Friendly Issues

Write issue descriptions that can be directly copied to an AI agent. Include context, expected behavior, and acceptance criteria.

**Feature example:**

```
Add a "Duplicate" button for Workspaces.

Context:
- Add button to: Workspace details page (next to "Delete") and Workspaces table row dropdown
- Create a new route that opens the create/edit wizard pre-filled with the source workspace data
- Name defaults to "{original}-copy" suffix, all fields are editable
- Validate that the new name doesn't already exist

Acceptance criteria:
- [ ] Duplicate button appears in both locations
- [ ] Wizard pre-fills correctly with "-copy" name suffix
- [ ] Validation error shown for duplicate names
- [ ] New workspace created successfully
- [ ] Unit and Cypress tests added
```

**Bug fix example:**

```
Fix: Workspace list does not refresh after deletion.

Steps to reproduce:
1. Open the Workspaces list page
2. Delete a workspace
3. The deleted workspace still appears in the list

Expected: List refreshes automatically after deletion.

Acceptance criteria:
- [ ] List refreshes after workspace deletion
- [ ] No full page reload required
- [ ] Regression test added
```

### Reviewing PRs with Agents

Use AI agents to perform a first-pass review of pull requests. Check out the branch locally and use the prompt below, replacing `<PR-URL>` with the actual pull request URL.

> [!NOTE]
> The prompt assumes your remote is named `upstream` and you're reviewing commits in `HEAD`. Adapt the git commands to your setup if needed.

> [!WARNING]
> Agent reviews are a starting point, not a replacement for human review. Always evaluate suggestions critically and make the final decision yourself.

**Prompt:**

````
Review all commits in this branch against our coding standards.

## Step 0: Gather context from the PR

Read the pull request at <PR-URL>.
Extract the PR description, review comments, and any linked issue URLs.
Open and read each linked issue to understand the full context and acceptance criteria.

## Step 1: Get the branch diff

Run these commands:
```
git fetch upstream
git log upstream/notebooks-v2..HEAD --oneline
git diff upstream/notebooks-v2...HEAD
```
Review only committed code. Ignore uncommitted changes.

## Step 2: Read the relevant guidelines

Always read @AGENTS.md first. Then, based on changed files:
- Controller changes: @workspaces/controller/AGENTS.md plus linked skills
- Backend changes: @workspaces/backend/AGENTS.md plus linked skills
- Frontend changes: @workspaces/frontend/AGENTS.md plus linked skills
- Cypress changes: @workspaces/frontend/src/__tests__/cypress/AGENTS.md plus linked skills

## Step 3: Review each changed file

Check for:
- MUST/MUST NOT violations (blocking)
- SHOULD/SHOULD NOT violations (recommendation)
- Anti-patterns from linked skill playbooks
- Common pitfalls from the module's AGENTS.md
- Code structure (single responsibility, separation of concerns)
- DRY violations (duplicated code)
- Alignment with the PR description and linked issue acceptance criteria

## Step 4: Report issues

For each issue found, output:

**File:** `path/to/file.ts` (line X)
**Severity:** Blocking | Recommendation
**Rule:** [Quote the specific rule violated]
**GitHub Comment:**
```
[2-3 sentence comment stating the issue and suggesting a fix. Do not reference AGENTS.md files. Ready to paste into GitHub PR review.]
```
````

---

## 🔧 Extensibility

When adding agent guidelines for a new module, use the existing `AGENTS.md` files as reference. If module guidance becomes too large, extract focused `SKILL.md` files following the [agentskills.io](https://agentskills.io/specification) standard:

```
.agents/skills/
├── kubeflow-notebooks-frontend-component-authoring/SKILL.md
├── kubeflow-notebooks-frontend-jest-rtl-testing/SKILL.md
└── kubeflow-notebooks-frontend-api-integration/SKILL.md
```

> [!IMPORTANT]
> Update `agents/mappings.conf` when adding new files.

---

## 🤝 Contributing

Found an issue or have a suggestion? Feel free to open a PR.
