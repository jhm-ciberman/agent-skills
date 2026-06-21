---
name: create-github-release
description: "Create a GitHub release for a repo with an auto-generated calver tag and generated release notes, behind a confirmation step. Use when cutting a release."
---

# Create a release for a repo

Create a new GitHub release for a repository with an auto-generated calver tag and release notes.

**OPTIONAL ARGUMENT:** the repo as `owner/repo`, or just `repo`. If only a repo name (or nothing) is given, the owner defaults to the current repository's owner.

## Instructions

1. **Resolve owner and repo.**
   - `owner/repo` argument: split it.
   - just `repo`: use it with the current owner.
   - nothing: use the current repository.
   ```bash
   gh repo view --json owner,name --jq '.owner.login + "/" + .name'
   ```

2. **Find the default branch:**
   ```bash
   gh repo view $owner/$repo --json defaultBranchRef --jq '.defaultBranchRef.name'
   ```

3. **Get the latest release tag:**
   ```bash
   gh release view --repo $owner/$repo --json tagName --jq '.tagName'
   ```

4. **Check pending commits** (use the default branch from step 2):
   ```bash
   gh api repos/$owner/$repo/compare/$tag...$default_branch --jq '.ahead_by'
   ```
   If 0 commits pending, show a message and stop.

5. **Generate the next calver tag:**
   - Format: `vYYYY.MM.DDx` where x is a letter (a, b, c, ...).
   - Use today's date. If a release with today's date already exists, increment the letter (e.g. `v2026.01.08a` exists -> use `v2026.01.08b`).

6. **Generate the release notes preview:**
   ```bash
   gh api repos/$owner/$repo/releases/generate-notes \
     -f tag_name="$new_tag" \
     -f target_commitish="$default_branch" \
     -f previous_tag_name="$previous_tag" \
     --jq '.body'
   ```

7. **Show the full preview**: repo URL, new tag, previous tag, commit count, then the full release notes.

8. **Ask the user to confirm** with the AskUserQuestion tool: "Create release" or "Cancel".

9. **Only if confirmed**, create the release:
   ```bash
   gh release create $new_tag \
     --repo $owner/$repo \
     --title "$new_tag" \
     --notes "$release_notes" \
     --target $default_branch
   ```

10. **Show a success message** with the link to the new release.

**IMPORTANT:**
- NEVER create a release without showing the full preview first.
- NEVER create a release without explicit user confirmation.
- Only ONE repo per command execution.
