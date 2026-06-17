# Claude Code Workflow

TRIGGER — invoke this skill BEFORE acting on any feature request for the Odara project. Fire whenever: the user asks to build, add, create, or implement a new page, component, UI flow, cart behavior change, WhatsApp integration change, or any other non-trivial feature; OR you are about to spawn `frontend-agent` without first confirming that both a spec file in `odara/docs/specs/` and a task file in `odara/docs/tasks/` already exist for the feature. SKIP for bug fixes, config tweaks, renames, and purely cosmetic changes.

When this skill fires: determine which SDD phase applies (check whether spec and task files exist), then route the user to the correct next agent by spawning it via the Agent tool. Never skip phases or act on a feature request before the required artifacts exist.

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
