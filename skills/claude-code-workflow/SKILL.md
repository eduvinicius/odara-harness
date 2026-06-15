# Claude Code Workflow

This file is background knowledge. It is not a slash command. Read it to understand the correct phase sequence for any new feature and to guide the user to the right next step between agent invocations.

---

## The SDD Loop

Five phases, always in this order:

| # | Phase | Agent | Artifact |
|---|-------|-------|----------|
| 1 | **Specify** | `spec-agent` | `docs/specs/<feature>-spec.md` |
| 2 | **Research** *(optional)* | `researcher-agent` | `docs/research/<topic>.md` |
| 3 | **Plan** | `planner-agent` | `docs/tasks/<feature>-plan.md` |
| 4 | **Implement** |`frontend-agent` | committed code |
| 5 | **Validate** | `code-reviewer` | review report |

---

## Phase Entry Conditions

- **Research** is optional. It becomes required when the spec references a library, pattern, or integration the project has not used before. If required, it must complete before Plan starts.
- **Plan** may not start until a spec file exists at `docs/specs/`.
- **Implement** may not start until a task list exists at `docs/tasks/`.
- **Validate** runs after every implementation, regardless of feature size.

---

## Inter-Phase Rules

1. **Never start Plan without a completed spec file** in `docs/specs/`.
2. **Never start Implement without a completed task list** in `docs/tasks/`.
3. **Never skip Validate**, even for small or trivial features.
4. **Run `/clear` between Plan and Implement** to avoid context pollution.

---

## When a Phase Is Skipped

If the user asks to jump ahead or skip a phase, do not refuse. Instead, remind them of the correct sequence:

> "The SDD loop requires [missing phase] before [requested phase]. The correct order is Specify → Research (if needed) → Plan → Implement → Validate. Want to run [missing agent] first?"

---

## What This Skill Does Not Cover

- Bug fixes and trivial changes do not need to go through the full loop.
- This skill does not describe how each agent works internally — see each agent's file in `.claude/agents/`.
- Phase completion is enforced by reminder only, not programmatically.
