---
name: org-prs
description: "List all open PRs across a GitHub organization or user account in a table with CI status. Use to review open pull requests across many repos at once."
---

# List all open PRs across a GitHub org or account

Fetch all open PRs across a GitHub organization (or user account) and display them in a table with CI status.

**OPTIONAL ARGUMENT:** the org or user login (e.g. `vercel`). If omitted, the owner of the current repository is used.

## Instructions

1. **Resolve the owner.** Use the argument if given, otherwise read it from the current repo:
   ```bash
   gh repo view --json owner --jq '.owner.login'
   ```

2. **Pick the search scope.** Orgs and personal accounts use different qualifiers:
   ```bash
   gh api users/$owner --jq '.type'   # "Organization" or "User"
   ```
   Use `org:$owner` for an Organization, `user:$owner` for a User.

3. Run this single GraphQL query to get all PRs with status, labels, and author in one call (substitute the scope from step 2 for `org:OWNER`):
   ```bash
   gh api graphql -f query='
   {
     search(query: "org:OWNER is:pr is:open", type: ISSUE, first: 100) {
       nodes {
         ... on PullRequest {
           number
           title
           url
           author { login }
           repository { nameWithOwner }
           labels(first: 5) { nodes { name } }
           commits(last: 1) {
             nodes { commit { statusCheckRollup { state } } }
           }
         }
       }
     }
   }'
   ```

   The `statusCheckRollup.state` will be: SUCCESS, FAILURE, PENDING, or null (no CI).
   The `labels` contain dependabot's language tags like "php", "javascript", "github-actions", etc.

4. Present results in a markdown table grouped by repository with columns:
   - PR link in short format: `[org/repo#123](full-url)` - display text omits github.com
   - Title/Package name prefixed with language emoji (e.g., "🐘 laravel/tinker 2.10.1 → 2.11.0")
   - Author: always show name, with emoji prefix (🙂 human, 🤖 dependabot)
   - Status (text + emoji)

Use ONE language emoji per PR based on labels (e.g., "php" label → 🐘):
- 🐘 php
- 🟨 javascript / typescript
- 🦀 rust
- 🐍 python
- 💎 ruby
- 🎮 c#
- 🔷 go
- ⚙️ github_actions
- ⚫ unknown / no label
- For other labels, pick a fitting emoji.

Use these status indicators:
- PASS = ✅
- FAIL = ❌
- PENDING = ⏳
- NO_CI = ⚪

Do NOT merge anything. Only display the table.
