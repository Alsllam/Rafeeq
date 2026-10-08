# Rafeeq backend

**Skill:** load `.claude/skills/dotnet-modular-backend/SKILL.md` before any work here. Follow it for the stack, layout, layers, endpoints, errors, tests and workflow. This file gives the Rafeeq values and the places where Rafeeq deliberately differs. Also read the root `CLAUDE.md` (hard rules).

Status: not started. Reminders, check-ins and alert escalation wait for the approved `docs/architecture.md`.

---

## 1. Names

| Placeholder | Value |
|---|---|
| `{Co}` | `Rafeeq` |
| `{Product}` | `Care` |
| Solution | `backend/Rafeeq.Care.sln` |
| Framework projects | `Rafeeq.Framework.Domain`, `.Application`, `.EntityFrameworkCore` |
| Migrator | `Rafeeq.Care.DbMigrator` |

## 2. Modules

| Module (`{Module}`) | Schema | Host port | Owns |
|---|---|---|---|
| `FamilyCircles` | `family` | 5101 | Elderly person profile, family circles, members, invitations, **grants**, **consent records**, access log shown to the elderly person. User accounts live in the Auth host (ASP.NET Identity) |
| `CarePlans` | `careplans` | 5102 | Medications (name, dose text, schedule, critical flag), appointments, tasks, care notes, care-plan versions |
| `CheckIns` | `checkins` | 5103 | Check-in schedules, check-in sessions, answers, mood/activity signals, medication confirmations (taken / not taken / skipped) |
| `Alerts` | `alerts` | 5104 | Alert rules, alerts, severity, escalation policy and state, acknowledgements, emergency-number config per country |
| `Notifications` | `notifications` | 5105 | Device tokens, push, SMS, voice call dispatch, delivery receipts, quiet hours, templates (ar/en) |
| `Caregivers` | `caregivers` | 5106 | Caregiver invitations, shared task list, shift notes, caregiver-specific grants |

Hosts: `Rafeeq.Care.{Module}.Host` per module, plus `Rafeeq.Care.BFF.Host` (5000), `Rafeeq.Care.Auth.Host` (5001), `Rafeeq.Care.Jobs.Host` (5200).

Cross-module traffic only through MassTransit events or Refit (skill §2). Expected events (refine in architecture step): `MedicationDoseDue`, `MedicationConfirmed`, `MedicationMissed`, `CheckInCompleted`, `CheckInMissed`, `HelpButtonPressed`, `EmergencySignalDetected`, `AlertRaised`, `AlertAcknowledged`, `AlertEscalated`, `GrantChanged`, `ConsentChanged`, `CarePlanChanged`.

## 3. Roles and permissions

Roles: `ElderlyPerson`, `PrimaryFamilyMember`, `FamilyMember`, `Caregiver`, `CareCompanyAdmin`, `CareCompanyStaff`, `PlatformAdmin`. The full matrix lives in `docs/SRS.md`.

- **Permissions are scoped per circle, not global.** A user can be primary in one circle and a relative in another. The permission check is "user X has permission P **in circle C**". Every circle-owned entity implements `ISecuredEntity` with `FamilyCircleId`, and the global query filter uses the caller's grants for that circle.
- Permission cache key includes the circle: `perm:{userId}:{circleId}`. `GrantChanged` evicts it.
- `PlatformAdmin` has **no** default read access to health data. Support access is a time-boxed grant the elderly person or primary member approves, and it is logged.
- Every read of health data by someone other than the elderly person writes an access-log entry (who, what, when) that the elderly person can see.

## 4. Where Rafeeq differs from the skill

- **Sensitive data.** Medication names and doses, notes, check-in answers, transcripts, mood signals, alert details and national IDs are `[Sensitive]`. Never in logs, Hangfire job arguments, MassTransit message headers, push or SMS bodies. Messages carry ids; consumers load the data.
- **Alerts reliability.** The `Alerts` and `Notifications` modules are the critical path:
  - escalation steps are durable (persisted state machine + scheduled jobs), survive restarts, and are idempotent per `AlertId` + step;
  - an emergency alert never waits on the AI service, the read models, or another non-critical module;
  - SMS and voice-call providers sit behind interfaces with at least two implementations planned (primary + fallback). Provider choice is an ADR, ask first.
- **Time.** Store UTC, keep the elderly person's IANA time zone on the profile, and compute dose times in that zone (no DST in KSA, but GCC users may travel).
- **Consent.** Consent is a first-class entity (`ConsentRecord`: subject, purpose, granted by, method — app/voice/representative, version of the text, timestamp, revoked at). Features that need a consent check it, server side.
- **Retention and deletion.** Every module implements export and erase for one elderly person (PDPL data-subject rights), triggered by an event from `FamilyCircles`.
- **No GET reads** still applies (skill §5), and it matters more here: health filters must never land in URLs or logs.
- **Rate limiting** never applies to the help-button / emergency endpoints.

## 5. Localization

`Resources/ar.json` and `Resources/en.json`. Notification templates are per locale and per dialect-neutral formal Arabic. Keep a key namespace per module: `Alerts:Push:HelpPressed`, `CarePlans:Validation:DoseRequired`.

## 6. Tests that are always required here

Besides the skill minimums: permission scoping across two circles (a member of circle A never sees circle B), grant revocation takes effect immediately, escalation state machine (every transition, restart mid-escalation, duplicate events), and "emergency alert still dispatched when AI service is unavailable".
