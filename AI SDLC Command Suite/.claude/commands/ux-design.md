---
allowed-tools: ["Read", "Write", "Glob"]
description: "Phase 5 — UI/UX Design agent invoking command. For any project with user-facing interfaces. Designs the complete UX layer anchored to PRD personas and user stories. Human-gated with Product Owner and user testing validation before Phase 6 begins."
---

# /sdlc:ux-design

Invoke the **UX Design Agent** to execute Phase 5 of the SDLC pipeline.

## Usage

```
/sdlc:ux-design
```

## Prerequisites

- Phase 4 complete: `docs/design/ARCHITECTURE.md` approved

## Outputs

| File | Description |
|------|-------------|
| `docs/ux/USER-JOURNEYS.md` | User flows for each persona |
| `docs/ux/WIREFRAMES.md` | Screen specifications with states (markdown reference) |
| `docs/visuals/ux/SCR-XXX-*.html` | Self-contained HTML wireframe per screen — Tailwind CDN, all states as vertical sections, annotated callouts. Renders via `<iframe>`. Authoritative spec for Phase 6 task sizing and Phase 7 implementation. |
| `docs/ux/DESIGN-SYSTEM.md` | Design tokens and component specs |
| `docs/ux/ACCESSIBILITY.md` | WCAG 2.1 AA compliance checklist |

## Human Gate

Product Owner + user testing validation required. Once approved:
```
/sdlc:task-breakdown
```

---
*AI SDLC Suite — Phase 5 of 14*

Use `@.claude/agents/05_ux_design_agent.md` as the agent for this command.
