Create GitHub issues from a markdown task file and apply a label.

## Usage

```
/create-sprint <path-to-file.md>
/create-sprint <path-to-file.md> --label "my-label"
```

**Examples:**
- `/create-sprint docs/sprints-tracking/sprint1.md` — uses label from the file or asks
- `/create-sprint docs/sprints-tracking/sprint1.md --label "sprint-1"` — uses given label

## Label resolution (priority order)

1. `--label "..."` passed in `$ARGUMENTS` — use it as-is
2. `**Label:**` field in the MD file, e.g. `**Label:** sprint-1`
3. Neither found — ask the user: *"What label should be applied to these issues?"*

## Expected Markdown Format

Each `###` heading is one issue. The content under it becomes the issue body.

```markdown
**Label:** sprint-1   ← optional, used if no --label flag

### Issue Title
**Description:** ...
**Technical Tasks:**
* ...
**Acceptance Criteria:**
* ...
```

Flat `-` bullet lists under a `## Tasks` heading are also supported:
```markdown
- Task title: description
```

## Instructions

### Step 1 — Parse arguments and file

Split `$ARGUMENTS` into:
- `file_path` — everything before `--label` (trim whitespace)
- `label_arg` — value after `--label` if present

Read the file at `file_path`. Extract:
- **Label** — from `--label` arg, or `**Label:**` line in the file
- **Issue list** — each `###` section as `{ title, body }`, OR each `-` bullet as `{ title, body }`

If neither a flag nor a `**Label:**` field is found, ask the user for the label before continuing.

### Step 2 — Resolve the repository

```
gh repo view --json nameWithOwner
```

Save `OWNER/REPO`.

### Step 3 — Ensure the label exists

```
gh label list --repo OWNER/REPO --json name
```

If the label is not in the list, create it:

```
gh label create "LABEL" --repo OWNER/REPO --color "0075ca"
```

### Step 4 — Create issues

For each issue in the parsed list:

1. Check if an open issue with the same title already exists:
   ```
   gh issue list --repo OWNER/REPO --search "TITLE" in:title --json number,title
   ```
   If found, skip creation and note it as "already exists".

2. If not found, create it:
   ```
   gh issue create \
     --repo OWNER/REPO \
     --title "TITLE" \
     --body "BODY" \
     --label "LABEL"
   ```

Collect each issue number.

### Step 5 — Report results

```
Label:  LABEL
Issues: X created, Y skipped (already exist)
  ✓ #42  Title 1
  ✓ #43  Title 2
  ~ #11  Title 3  (already existed)
```

If any issue creation failed, report the error inline and continue with the rest.

## Notes
- Never push code or modify branches.
- Do not touch GitHub Projects, iterations, or milestones unless explicitly asked.
