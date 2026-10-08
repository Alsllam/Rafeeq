# Rafeeq mobile app

**Skill:** load `.claude/skills/flutter-clean-mobile/SKILL.md` before any work here. Follow it for the stack, Clean Architecture layout, BLoC, networking, offline sync, security, localization and tests. This file gives the Rafeeq values and the places where Rafeeq deliberately differs. Also read the root `CLAUDE.md` (hard rules).

Status: not started. Reminder, check-in and alert code waits for the approved `docs/architecture.md`.

---

## 1. Names and flavors

| Item | Value |
|---|---|
| Folder | `mobile/` is the Flutter project root |
| Dart package | `rafeeq` |
| App name | رفيق / Rafeeq |
| Bundle / application id | `sa.rafeeq.app` (to confirm before store registration) |
| Flavors | `dev`, `staging`, `prod` (no `uat` unless asked) |
| Platforms | Android + iOS phones. Tablets supported in parent mode (many elderly users prefer a tablet) |

## 2. One app, two experiences

The app has two modes chosen at sign-in by the user's role in the circle, never by a toggle the elderly person can hit by mistake.

| Mode | Who | Shape |
|---|---|---|
| **Parent mode** | `ElderlyPerson` | A single home screen, no bottom nav, no drawer. Voice first. Big help button always visible. At most 2 taps to anything |
| **Family mode** | `PrimaryFamilyMember`, `FamilyMember`, `Caregiver` | Normal app shell: today summary, alerts, care plan, check-in history, trends, circle and permissions. Caregivers see only what their grants allow (`PermissionGate`) |

Folder layout (feature-first, per skill §2): shared features in `lib/features/`, with mode-specific presentation under `lib/features/{feature}/presentation/parent/` and `.../family/` when a feature has both. Separate routers/shells per mode in `lib/app/router/`.

Features (expected): `auth`, `onboarding_consent`, `home_parent`, `voice_companion`, `check_ins`, `medications`, `appointments`, `help_button`, `alerts`, `daily_summary`, `trends`, `care_plan`, `family_circle`, `caregivers`, `tasks_notes`, `settings_accessibility`.

## 3. Where Rafeeq differs from the skill

### Elderly UX (parent mode) — overrides skill §0 sizes and motion
- **Tap targets ≥ 72×72 dp** in parent mode (skill says 48). Primary actions are full-width buttons at least 88 dp tall. Spacing between targets ≥ 16 dp.
- **Text:** base body ≥ 22 sp, primary labels ≥ 28 sp. Layouts must work at system text scale **2.0×** (skill says 1.3×) in parent mode, 1.5× in family mode.
- **High-contrast theme** in addition to light and dark (tokens from `docs/brand/`, step 4). Contrast ≥ 7:1 for text in parent mode (WCAG AAA).
- **Motion is calm and slow** (tokens from `docs/brand/`). No staggered list animations, no bouncy effects, no count-up counters in parent mode. Honour reduced motion always.
- **Forgiving input:** no swipe-only or long-press-only actions, no double-tap, no timeouts that discard input. Destructive or important actions (e.g. "not taken") are confirmed by voice or a second big button, and can be undone for a short window.
- **Voice for everything:** every screen in parent mode can be read aloud and answered by voice. Every button has a spoken label.
- **Tone:** formal, warm Arabic. No emoji, no cartoon mascots, no exclamation-heavy copy.
- Skeletons are fine in family mode. In parent mode prefer a calm static placeholder with a spoken "one moment" message.

### Reminders and notifications
- Medication and appointment reminders must fire **on the device without network**, using `flutter_local_notifications` scheduled from the locally cached care plan. The server is a second line, not the first. Details (exact alarms, reboot, battery optimisation, iOS limits, server fallback) are decided in `docs/architecture.md` — do not implement before approval.
- Notification channels: `critical_medication`, `medication`, `appointment`, `check_in`, `family_alert`, `emergency`, `general`. Emergency and critical channels use the highest importance the platform allows; channel ids never change once shipped.
- Push and notification text never contains health details (root `CLAUDE.md` §6).

### Help button and emergency
- The help button works with no network: it calls the backend when online, otherwise falls back to SMS (where possible) and the one-tap emergency call. Emergency numbers come from the synced country config, with a built-in default.
- The help flow never goes through the AI service.

### Offline sync (skill §6) applies, with these job types
`ConfirmMedication`, `SubmitCheckIn`, `RaiseHelp` (highest priority, sent first), `AddNote`, `CompleteTask`. Each carries a client UUID and the device-local timestamp of the event.

### AI and voice
- All AI and speech calls go to `/ai-api/**` (not `/ai/**` as in skill §10): `/ai-api/companion/turn`, `/ai-api/transcribe`, `/ai-api/speak`. The app never holds a model or speech key.
- Recording: `record`, 16 kHz mono. Show a large, slow level indicator and allow cancel by voice or a big button.
- Chat/voice history on device is minimal, encrypted, and cleared on logout.

### Analytics and crash reporting
`firebase_analytics` events never include health data, names, phone numbers or free text. Crashlytics logs never include request bodies. Consider whether Firebase is acceptable under data residency (open question for step 5); keep it behind interfaces so it can be replaced.

## 4. Localization
`lib/l10n/app_ar.arb` is written first, `app_en.arb` must match. Arabic is the default locale. Hijri + Gregorian on dates, Arabic-Indic digits optional per user setting.

## 5. Tests that are always required here
Besides the skill minimums: golden tests of parent-mode screens at text scale 1.0 / 1.5 / 2.0 in ar, light / dark / high-contrast; reminder scheduling (time zone, reschedule after care-plan change, reboot); help button offline path; permission-gated family-mode screens for a caregiver.
