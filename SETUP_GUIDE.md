# Project Board Setup Guide

This document explains how the LoVeT ClinicAI ticketing system was built, so a future developer or AI agent can recreate it for another project.

## Overview

We use **GitHub Projects v2** as a Kanban board for ticket management. A dedicated **tracker repo** (`lovet-tickets`) holds all issues, separate from the code repos. GitHub Actions automate adding issues to the board and tracking rejections.

## Architecture

```
lovet-tickets (issue tracker repo)
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml        # Guided bug report form
│   │   ├── feature_request.yml   # Feature request form
│   │   ├── improvement.yml       # Improvement form
│   │   └── config.yml            # Disables blank issues, adds board link
│   └── workflows/
│       └── auto-add-to-project.yml  # Auto-adds new issues to the board
├── README.md                     # Hebrew guide for product team
└── SETUP_GUIDE.md                # This file

clinic-ai-fe (frontend repo)
└── .github/workflows/
    └── project-board-automation.yml  # Tracks rejections on status changes
```

## Step-by-Step Recreation

### 1. Create a GitHub Project (v2)

```bash
gh project create --owner <ORG> --title "<PROJECT NAME>" --format json
```

Save the project `id` (e.g., `PVT_kwDO...`) and `number` from the output. You'll need these throughout.

Add a description:
```bash
gh project edit <NUMBER> --owner <ORG> --description "<DESCRIPTION>"
```

### 2. Configure Status Columns

The project comes with a default Status field (Todo, In Progress, Done). Update it via GraphQL.

First, get the Status field ID:
```bash
gh api graphql -f query='
query {
  organization(login: "<ORG>") {
    projectV2(number: <NUMBER>) {
      id
      field(name: "Status") {
        ... on ProjectV2SingleSelectField {
          id
          options { id name }
        }
      }
    }
  }
}'
```

Then update the options:
```bash
gh api graphql -f query='
mutation {
  updateProjectV2Field(input: {
    fieldId: "<STATUS_FIELD_ID>",
    singleSelectOptions: [
      {name: "Backlog", color: GRAY, description: "New tickets, not yet prioritized"},
      {name: "Ready for Work", color: BLUE, description: "Prioritized, ready to pick up"},
      {name: "In Progress", color: ORANGE, description: "Actively being worked on"},
      {name: "Ready for Product Test", color: YELLOW, description: "Dev done, awaiting product verification"},
      {name: "Done", color: GREEN, description: "Accepted and complete"}
    ]
  }) {
    projectV2Field {
      ... on ProjectV2SingleSelectField {
        options { id name color }
      }
    }
  }
}'
```

Save the option IDs from the response — you'll need them for the automation workflows.

### 3. Add Custom Fields

Create fields using `createProjectV2Field`. Available `dataType` values: `SINGLE_SELECT`, `NUMBER`, `TEXT`, `DATE`.

```bash
gh api graphql -f query='
mutation {
  createProjectV2Field(input: {
    projectId: "<PROJECT_ID>",
    dataType: SINGLE_SELECT,
    name: "Priority",
    singleSelectOptions: [
      {name: "Critical", color: RED},
      {name: "High", color: ORANGE},
      {name: "Medium", color: YELLOW},
      {name: "Low", color: GRAY}
    ]
  }) {
    projectV2Field { ... on ProjectV2SingleSelectField { id name } }
  }
}'
```

Repeat for other fields (Area, Category, etc.). Note: "Type" is a reserved name — use "Category" instead.

For the rejections counter:
```bash
gh api graphql -f query='
mutation {
  createProjectV2Field(input: {
    projectId: "<PROJECT_ID>",
    dataType: NUMBER,
    name: "Rejections"
  }) {
    projectV2Field { ... on ProjectV2Field { id name } }
  }
}'
```

### 4. Create the Kanban View

This **cannot** be done via API. Go to the project URL in a browser:
1. Click the `+` tab → select "Board"
2. It uses the Status field as columns automatically

### 5. Create the Tracker Repo

```bash
gh repo create <ORG>/<TRACKER_REPO> --public \
  --description "Issue tracker for <PROJECT>"
```

Link it to the project:
```bash
gh project link <NUMBER> --owner <ORG> --repo <ORG>/<TRACKER_REPO>
```

Optionally unlink code repos so product only sees the tracker:
```bash
gh project unlink <NUMBER> --owner <ORG> --repo <ORG>/<CODE_REPO>
```

### 6. Add Issue Templates

Create `.github/ISSUE_TEMPLATE/` in the tracker repo with YAML form templates. Key features:
- Use `type: dropdown` for structured fields
- Use `type: textarea` for free text, with `render: shell` for code blocks
- Use `type: input` for single-line free text (good for "Other" fallbacks)
- Set `labels` at the top to auto-label based on template
- Use `config.yml` to disable blank issues

See the templates in this repo for examples.

### 7. Add Labels

```bash
gh label create "bug" --description "Something isn't working" --color "d73a4a" --repo <ORG>/<TRACKER_REPO>
gh label create "feature" --description "New functionality" --color "0e8a16" --repo <ORG>/<TRACKER_REPO>
# ... etc
```

Delete default GitHub labels that aren't useful:
```bash
for label in "documentation" "duplicate" "enhancement" "good first issue" "help wanted" "invalid" "question" "wontfix"; do
  gh label delete "$label" --repo <ORG>/<TRACKER_REPO> --yes
done
```

### 8. Group Repos with Topics

```bash
gh repo edit <ORG>/<REPO> --add-topic <PROJECT_TAG>
```

Apply the same topic to all related repos. Find them all at:
`github.com/search?q=topic:<PROJECT_TAG>+org:<ORG>`

### 9. Create the Auto-Add Workflow

In the **tracker repo**, create `.github/workflows/auto-add-to-project.yml`. This workflow:
- Triggers on `issues: [opened]`
- Uses GraphQL `addProjectV2ItemById` mutation to add the issue to the project
- Sets the Status to Backlog
- Optionally sets Category based on the template label

See `auto-add-to-project.yml` in this repo for the full implementation.

### 10. Create the Board Automation Workflow

In one of the **code repos** (or the tracker repo), create `.github/workflows/project-board-automation.yml`. This workflow:
- Triggers on `projects_v2_item: [edited]`
- Detects when the Status field changes
- If an item moves to "Ready for Work" (bounce-back), increments the Rejections field and comments on the issue

See `project-board-automation.yml` in the `clinic-ai-fe` repo for the full implementation.

**Important:** The `projects_v2_item` event only fires in repos linked to the project OR repos in the same org. The workflow must be on the default branch.

### 11. Set Up the PAT Secret

The workflows need a Personal Access Token with `repo` and `project` scopes.

1. Create a PAT (classic) at github.com/settings/tokens
2. Set it as a secret on relevant repos:
```bash
echo "<TOKEN>" | gh secret set PROJECT_TOKEN --repo <ORG>/<REPO>
```

Or at the org level (requires admin:org scope):
```bash
gh secret set PROJECT_TOKEN --org <ORG> --visibility selected --repos <REPO1>,<REPO2>
```

## Key IDs Reference (LoVeT)

These are the IDs for the LoVeT project. A new project will have different IDs.

| Resource | ID |
|---|---|
| Project | `PVT_kwDOBqFAks4BPjQd` |
| Status field | `PVTSSF_lADOBqFAks4BPjQdzg97V88` |
| Priority field | `PVTSSF_lADOBqFAks4BPjQdzg97XZc` |
| Area field | `PVTSSF_lADOBqFAks4BPjQdzg97XZg` |
| Category field | `PVTSSF_lADOBqFAks4BPjQdzg97Xcc` |
| Rejections field | `PVTF_lADOBqFAks4BPjQdzg97YcM` |

### Status Option IDs

| Status | ID |
|---|---|
| Backlog | `38269463` |
| Ready for Work | `bdd5ee37` |
| In Progress | `cb998c15` |
| Ready for Product Test | `68d3b1ba` |
| Done | `5fb0d65e` |

## Limitations

- **Board views** cannot be created via API — must be done in the browser
- **`projects_v2_item` events** don't include the previous field value, only the current one. The bounce-back automation triggers whenever an item moves TO "Ready for Work" via the Status field, regardless of where it came from.
- **"Type" is reserved** — GitHub reserves this field name. Use "Category" or similar.
- **Token scopes** — the default `GITHUB_TOKEN` in Actions doesn't have project permissions. A PAT or GitHub App token is required.
