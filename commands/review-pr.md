Review the pull request $ARGUMENTS using `gh` CLI and post inline code comments plus a summary review on GitHub.

## Usage

```
/review-pr                       # review the open PR on the current branch
/review-pr <number>              # review a specific PR by number
```

**Examples:**
- `/review-pr` — auto-detects the PR for the current branch
- `/review-pr 13` — reviews PR #13

## Instructions

1. Fetch the PR diff and file list:
   ```
   gh pr view $ARGUMENTS --json number,title,body,headRefName,baseRefName
   gh pr diff $ARGUMENTS
   gh pr files $ARGUMENTS
   ```

2. Analyze the diff thoroughly:
   - Look for bugs, logic errors, security issues, performance problems
   - Check code style and consistency with the existing codebase
   - Identify missing tests or documentation
   - Note positive aspects worth highlighting

3. Post **inline comments** on specific lines using:
   ```
   gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
   ```
   Every inline comment body MUST start with:
   > **🤖 Claude Code review:**

   Example comment body:
   ```
   **🤖 Claude Code review:**

   This variable is never used after assignment — consider removing it or checking if it should be passed to the next function.
   ```

4. After all inline comments, submit a **review summary** using:
   ```
   gh pr review $ARGUMENTS --comment --body "..."
   ```
   The summary body MUST start with:
   > **🤖 Claude Code — PR Review Summary**

   Structure the summary as:

   ```
   **🤖 Claude Code — PR Review Summary**

   ## Overview
   <2-3 sentence description of what the PR does>

   ## Issues Found
   - <list of bugs or problems>

   ## Suggestions
   - <list of non-blocking improvements>

   ## Positives
   - <things done well>

   ## Verdict
   <APPROVE / REQUEST CHANGES / COMMENT> — <one sentence reason>
   ```

   Use `--approve`, `--request-changes`, or `--comment` flag on `gh pr review` based on severity.

## Notes
- If $ARGUMENTS is empty, use the current branch's open PR: `gh pr view --json number -q .number`
- Always run `gh repo view --json owner,name` to resolve `{owner}` and `{repo}` before making API calls
- Keep individual inline comments short and focused — one issue per comment
- Do not post a comment if there is nothing meaningful to say about a line
