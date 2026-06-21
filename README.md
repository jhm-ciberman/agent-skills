# agent-skills

A small collection of personal [Claude Code](https://claude.com/claude-code) skills and
commands I use day to day, mostly opinionated documentation helpers. Take what's useful.

## Install (Claude Code)

```
/plugin marketplace add jhm-ciberman/agent-skills
/plugin install devkit@ciberman
```

The skills then live under the `devkit:` prefix, e.g. run `/devkit:php-docblock-writing`
after writing some PHP (or just let Claude reach for it from its description).

## What's inside

### Skills
- **`csharp-docblock-writing`** - review and fix the quality of C# XMLDoc and `//` comments on changed code.
- **`php-docblock-writing`** - the same, for PHP / Laravel docblocks.

### Commands
- **`org-prs`** - list all open PRs across a GitHub org or account, with CI status.
- **`pending-releases`** - find repos with unreleased commits and show their pending changelogs.
- **`create-github-release`** - cut a release with an auto-generated calver tag and notes, behind a confirmation step.
- **`artisan-docs-writer`** - write docs in Laravel's expressive style.

## Using the skills in Codex

The skills are plain `SKILL.md` folders, so they work in Codex too. Point Codex's skills
directory at them:

```bash
ln -s "$(pwd)/devkit/skills" ~/.agents/skills
```

(Or copy the folders in.) Gemini CLI would need a thin `.toml` wrapper per command and
isn't covered here yet.

## License

MIT.
