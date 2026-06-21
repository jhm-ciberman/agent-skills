# List repos with pending releases

Scan all repos under a GitHub org or account and show pending changelogs for each repo that has unreleased commits.

**OPTIONAL ARGUMENT:** the org or user login. If omitted, the owner of the current repository is used.

## Instructions

1. **Resolve the owner** (argument, else the current repo):
   ```bash
   gh repo view --json owner --jq '.owner.login'
   ```

2. Use this single GraphQL query to get all repos with their HEAD commit, default branch, and latest release in one call. `repositoryOwner` works for both orgs and user accounts:
   ```bash
   gh api graphql -f query='
   {
     repositoryOwner(login: "OWNER") {
       repositories(first: 100, isArchived: false, isFork: false) {
         nodes {
           name
           defaultBranchRef { name target { oid } }
           releases(first: 1, orderBy: {field: CREATED_AT, direction: DESC}) {
             nodes { tagName publishedAt tagCommit { oid } }
           }
         }
       }
     }
   }'
   ```

3. Compare HEAD oid vs release tagCommit oid. If they differ, the repo has pending changes. Only process those repos.

4. For each repo with pending changes, get the commit count. Use the repo's own default branch from the query, not a hardcoded `main`:
   ```bash
   gh api repos/$owner/$repo/compare/$tag...$default_branch --jq '.ahead_by'
   ```

5. Generate release notes using the GitHub API:
   ```bash
   gh api repos/$owner/$repo/releases/generate-notes \
     -f tag_name="vNEXT" \
     -f target_commitish="$default_branch" \
     -f previous_tag_name="$tag" \
     --jq '.body'
   ```

6. Present each repo's pending changelog with a visible header table:

   ```
   ┌──────────────────────────────────────────────────────────┬────────────────┐
   │ https://github.com/<owner>/<repo>                         │ <last-tag>     │
   ├──────────────────────────────────────────────────────────┴────────────────┤
   │ X commits pending                                                          │
   └────────────────────────────────────────────────────────────────────────────┘

   [Generated release notes - the ### sections like New Features 🚀, Bug Fixes 🐛, Dependency Updates 📦, Other Changes 🛠️]
   ```

7. Only show repos that have pending commits. Skip repos that are up to date or have no releases.

8. At the end, show a summary line:
   ```
   ---
   Summary: X repos with pending releases (Y total commits)
   ```

9. Sort repos by number of pending commits descending (most pending first).

Note: the `tag_name` for generate-notes can be any valid tag string. It's only used for the changelog link, not actually created.
