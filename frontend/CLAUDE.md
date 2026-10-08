# Rafeeq frontend (web)

**Skill:** load `.claude/skills/angular-nx-frontend/SKILL.md` before any work here. Follow it for the stack, Nx workspace, libraries, proxies, screens, charts, permissions, localization and tests. This file gives the Rafeeq values and the places where Rafeeq deliberately differs. Also read the root `CLAUDE.md` (hard rules).

Status: **later phase.** Do not scaffold until the mobile MVP and backend are in place, unless asked.

---

## 1. Names

| Placeholder | Value |
|---|---|
| Workspace root | `frontend/` itself is the Nx workspace (not `frontend/rafeeq-frontend/`) |
| `{scope}` | `@rafeeq` |
| Apps | `family-web` (port 4200): family and caregiver web view. `care-dashboard` (port 4300): home-care company dashboard for many elderly clients |
| Proxies | one per backend module: `family-circles-proxy`, `care-plans-proxy`, `check-ins-proxy`, `alerts-proxy`, `notifications-proxy`, `caregivers-proxy` |
| `apiName` values | `familyCircles`, `carePlans`, `checkIns`, `alerts`, `notifications`, `caregivers` |

## 2. Where Rafeeq differs from the skill

- **Brand kit already exists** in `docs/brand/` (step 4). Do not design a new one (skill §0); copy tokens and logo into `apps/*/src/assets/brand/` and `theme-layout-generator`.
- Add a **high-contrast** theme next to light / dark / dim; the family web view is also used by older relatives.
- Motion uses Rafeeq's calm tokens from `docs/brand/`, which are slower than skill §0.4.
- **Permissions are per circle.** The `PermissionService` checks the policy against the currently selected circle; switching circles reloads grants. The care-dashboard adds the care-company scope.
- Health data never goes into URLs, browser storage, or analytics. No third-party analytics on these apps without an ADR.
- The family web view is not an emergency channel: alerts are shown, but the guaranteed path is push/SMS/call from the backend.
- Charts (ECharts) for mood/activity trends must not imply clinical meaning: no "risk scores", no red/green health judgements unless the SRS defines them.
