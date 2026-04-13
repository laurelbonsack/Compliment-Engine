# How to Make Skills in Claude Code

Skills (also called slash commands) are reusable instructions you can invoke with `/skill-name` in Claude Code. They extend Claude's capabilities with domain-specific knowledge or step-by-step task procedures.

---

## File Structure

Every skill lives in its own directory containing a `SKILL.md` file:

```
my-skill/
├── SKILL.md           # Required: frontmatter + instructions
├── reference.md       # Optional: supporting content
└── examples/
    └── sample.md      # Optional: example outputs
```

---

## Storage Locations

| Scope | Path |
|-------|------|
| **Personal** (all projects) | `~/.claude/skills/<skill-name>/SKILL.md` |
| **Project** (current project only) | `.claude/skills/<skill-name>/SKILL.md` |

Personal skills take priority over project skills.

---

## SKILL.md Format

```yaml
---
name: my-skill
description: What this skill does and when to use it
---

Your skill instructions here in markdown...
```

The YAML frontmatter controls behavior. The markdown body is what Claude reads and follows when the skill is invoked.

---

## Frontmatter Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Skill name (lowercase, hyphens only, max 64 chars). Defaults to directory name. |
| `description` | string | When to use it. Claude reads this to decide auto-invocation (max 250 chars). |
| `argument-hint` | string | Shown in autocomplete, e.g. `[issue-number]` or `[filename] [format]` |
| `disable-model-invocation` | boolean | `true` = only you can trigger it; Claude won't auto-invoke |
| `user-invocable` | boolean | `false` = hidden from `/` menu; only Claude can auto-invoke |
| `allowed-tools` | string/list | Tools Claude can use without approval prompts while this skill is active |
| `model` | string | Override the model for this skill |
| `effort` | string | Override effort: `low`, `medium`, `high`, or `max` |
| `context` | string | Set to `fork` to run in an isolated subagent context |
| `agent` | string | Subagent type when using `context: fork`: `Explore`, `Plan`, `general-purpose` |
| `paths` | string/list | Glob patterns limiting when the skill activates |
| `shell` | string | Shell for `!` command blocks: `bash` (default) or `powershell` |

---

## Variables and Templating

### Argument Variables

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed to the skill |
| `$ARGUMENTS[0]` | First argument (0-based index) |
| `$0`, `$1`, `$2` | Shorthand for `$ARGUMENTS[N]` |
| `${CLAUDE_SESSION_ID}` | Current session ID |
| `${CLAUDE_SKILL_DIR}` | Directory containing the skill's `SKILL.md` |

### Dynamic Shell Injection

Run shell commands before Claude sees the content — output replaces the placeholder:

**Inline:**
```markdown
Current branch: !`git branch --show-current`
```

**Block:**
````markdown
```!
git status --short
npm --version
```
````

---

## Invocation

**Manual (by you):**
```
/skill-name
/skill-name argument1 argument2
```

**Auto-invocation (by Claude):**  
If `disable-model-invocation` is not `true`, Claude will automatically invoke the skill when your request matches its description.

**Control who can invoke:**
- Default: both you and Claude can invoke
- `disable-model-invocation: true`: only you (via `/skill-name`)
- `user-invocable: false`: only Claude (hidden from `/` menu)

---

## Examples

### Reference Skill (knowledge Claude applies inline)

```yaml
---
name: api-conventions
description: API design patterns and conventions for this codebase
user-invocable: false
---

# API Conventions

## Endpoint naming
- Use nouns for resources: `/users`, `/posts`
- Use hyphens for multi-word names: `/user-preferences`

## Response format
All responses use this structure:
{
  "success": true,
  "data": { ... },
  "error": null
}

## Error codes
- 400: Bad request
- 401: Unauthorized
- 404: Not found
- 500: Server error
```

---

### Task Skill with Arguments

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
allowed-tools: Bash(git *) Bash(gh *)
argument-hint: "[issue-number]"
---

Fix GitHub issue #$0:

1. Read the issue: `gh issue view $0`
2. Create a branch: `git checkout -b fix/issue-$0`
3. Implement the fix following project conventions
4. Write tests for the fix
5. Commit with a clear message
6. Push and open a PR
```

Invoke with: `/fix-issue 123`

---

### Skill with Dynamic Context Injection

```yaml
---
name: pr-review
description: Review the current pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *) Bash(git *)
---

## Pull Request Context

Diff:
!`gh pr diff`

Changed files:
!`gh pr diff --name-only`

Comments:
!`gh pr view --comments`

## Your Task

Review this PR for correctness, readability, and test coverage.
Flag anything that looks like a bug or missing edge case.
```

---

### Skill with Multiple Arguments

```yaml
---
name: migrate-component
description: Migrate a component between frameworks
argument-hint: "[component] [from-framework] [to-framework]"
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

Invoke with: `/migrate-component SearchBar React Vue`

---

## Tips

- Keep `SKILL.md` under 500 lines; move large reference content to separate files
- Front-load the most important use case in the `description` field (Claude reads it to decide when to auto-invoke)
- Use `disable-model-invocation: true` for destructive or high-stakes tasks so Claude never runs them automatically
- Use `context: fork` when the skill should run in an isolated subagent (won't pollute your main conversation context)
- Use `allowed-tools` to pre-approve specific tools so Claude doesn't prompt for permission on every use
- Include `ultrathink` anywhere in the body to enable extended thinking mode for complex analysis tasks

---

## Quick Setup

```bash
# Create a personal skill
mkdir -p ~/.claude/skills/my-skill
touch ~/.claude/skills/my-skill/SKILL.md

# Create a project-scoped skill
mkdir -p .claude/skills/my-skill
touch .claude/skills/my-skill/SKILL.md
```

Then edit `SKILL.md` with your frontmatter and instructions, and invoke it with `/my-skill` in Claude Code.
