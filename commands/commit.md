Create a git commit for the current staged/unstaged changes, using the linked issue or task for context.

## Usage

```
/commit                          # auto-generate message from diff + linked issue
/commit <hint>                   # use hint to guide the subject line
/commit --co-author              # add Co-Authored-By: Claude to the commit
/commit --co-author <hint>       # both
```

**Examples:**
- `/commit` — fully automatic
- `/commit fix metadata field naming in LNKState` — guided subject line
- `/commit --co-author` — automatic message, Claude listed as co-author
- `/commit --co-author add plan field to LNKState` — guided + co-author

## Instructions

1. Gather context:
   ```
   git status
   git diff HEAD
   ```

2. Find the related issue number from the current branch name (e.g. `4-lnk-04-...` → issue `#4`) and fetch its title and body:
   ```
   gh issue view <number> --json title,body
   ```
   If no issue number is found in the branch name, skip this step.

3. Stage all relevant changed files (prefer specific file names over `git add -A`):
   ```
   git add <files>
   ```

4. Write a commit message that:
   - Has a concise subject line (≤ 72 characters, imperative mood)
   - Has an optional body explaining *why*, not *what*, if the change is non-trivial
   - References the issue number at the end if one was found (e.g. `Closes #4`)
   - **Does NOT include any Co-Authored-By or Claude attribution lines** unless `--co-author` is present in $ARGUMENTS, in which case append: `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

5. Create the commit:
   ```
   git commit -m "$(cat <<'EOF'
   <subject line>

   <optional body>

   <optional: Closes #N>
   EOF
   )"
   ```

6. Confirm success with `git log -1 --oneline`.

## Notes
- If $ARGUMENTS contains a message hint, use it to guide the subject line
- Never use `--no-verify`
- Do not push unless the user explicitly says so
