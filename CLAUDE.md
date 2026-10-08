# Rafeeq (رفيق)

An Arabic AI companion for elderly people, and a care-coordination app for their families and caregivers. Saudi Arabia and the GCC first.

This file holds the product-wide rules and values. Each folder has its own `CLAUDE.md` that points to the house skill for that stack and adds Rafeeq-specific values. **When a folder `CLAUDE.md` or this file disagrees with a skill, this repo wins** (the skills say so themselves).

---

## 1. Hard rules (never break these, in any folder)

1. **Not a medical device.** Rafeeq is a companion and care-coordination tool.
   - The AI never diagnoses, never interprets symptoms, never suggests, gives or changes a medication or dose.
   - The AI only repeats what the care plan says, word for word for medication name, dose and time.
   - A medical question gets "please ask your doctor" (in the person's language) and, when relevant, a notification to the family.
   - No marketing text, store listing or UI copy may claim Rafeeq monitors, detects or treats a health condition.
2. **Emergencies skip the AI.** Help button, fall, chest pain, can't breathe, no response to check-ins, and the other signals in `docs/safety.md`:
   - detected by deterministic rules (button, keyword/phrase lists, timers), never by a model decision;
   - raise an alert immediately: push + SMS to the family, then call escalation;
   - the parent app always shows a one-tap call to the local emergency number (per country, see §5);
   - the emergency path must work if the AI service is down, slow, or returns an error.
   - The AI may *also* see the conversation, but it can never cancel, delay or downgrade an alert.
3. **Privacy (Saudi PDPL).**
   - Personal and health data is stored and processed in Saudi Arabia (§6).
   - Explicit, recorded consent from the elderly person (or their legal representative), per purpose.
   - The elderly person can see who has access to what, and revoke it.
   - Family members and caregivers see only what they were granted. Default is least access.
   - Health data is `[Sensitive]` (encrypted at rest), never logged, never sent to analytics or crash reports.
4. **Respectful elderly UX.** Very large text and buttons, high contrast, voice for everything, few screens, tolerant of shaky hands and mistakes (undo, confirm, no tiny targets). Formal, warm Arabic. **Never childish**, never patronising, never "cute".
5. **Reminders and alerts are the product.** A missed medication reminder or a missed alert is a severity-1 bug. Reliability beats features.

---

## 2. Repository map

| Folder | What | Skill (in `.claude/skills/`) | Status |
|---|---|---|---|
| `backend/` | .NET 9 modular backend, YARP BFF, OpenIddict, MassTransit, Hangfire | `dotnet-modular-backend` | not started |
| `mobile/` | One Flutter app, two experiences: **parent mode** and **family mode** (caregivers use family mode with limited permissions) | `flutter-clean-mobile` | not started |
| `ai-service/` | Python FastAPI: Azure OpenAI + Azure AI Speech, per-person memory, RAG over care plan and notes, safety layer | `python-azure-rag-service` | not started |
| `frontend/` | Angular 20 + Nx: family web view, home-care company dashboard | `angular-nx-frontend` | later phase |
| `docs/` | SRS, safety rules, brand kit, architecture, ADRs | — | in progress |

Before working in a folder, read its `CLAUDE.md` and load the matching skill.

---

## 3. Glossary (use these words in code, docs and UI)

| Term | Arabic (UI) | Meaning |
|---|---|---|
| Parent / Elderly person | الوالد / الوالدة، كبير السن | The person cared for. Uses parent mode. In code: `ElderlyPerson`. |
| Family circle | دائرة العائلة | Everyone connected to one elderly person, with their grants. In code: `FamilyCircle`. |
| Primary family member | المسؤول الأساسي | Manages the circle, invites others, manages the care plan. |
| Relative | أحد أفراد العائلة | A family member with granted, usually read-mostly, access. |
| Caregiver | مقدّم الرعاية | Home nurse, domestic worker, clinic staff. Invited with limited permissions. |
| Care company | شركة الرعاية المنزلية | Organisation managing many elderly clients (web dashboard, later). |
| Care plan | خطة الرعاية | Medications, schedules, appointments, tasks, notes. The only source of medical facts the AI may repeat. |
| Check-in | الاطمئنان اليومي | A scheduled voice conversation ("how did you sleep?"). |
| Reminder | تذكير | Medication or appointment prompt with a confirmation (taken / not taken). |
| Alert | تنبيه | A notification to the circle that something is wrong. Has a severity and an escalation path. |
| Grant | صلاحية | What one member may see or do in one circle. |

"Parent" is a product word, not an assumption: the elderly person may be a grandparent, uncle, spouse, or a care-company client.

---

## 4. Shared values

| Value | Setting |
|---|---|
| Product slug | `rafeeq` |
| .NET naming | `{Co}` = `Rafeeq`, `{Product}` = `Care` → `Rafeeq.Care.CheckIns.Host`, `Rafeeq.Framework.Domain` |
| npm scope | `@rafeeq` |
| Dart package | `rafeeq` |
| Python package | `rafeeq_ai` |
| Default locale | `ar-SA`. Also `en`. AI voice: Gulf/Saudi dialects + MSA (see `ai-service/CLAUDE.md`) |
| Time zone | Store UTC. Schedule in the elderly person's own IANA zone (default `Asia/Riyadh`). Reminders are wall-clock times in that zone |
| Calendars | Gregorian + Hijri (Umm al-Qura) on every date shown to people |
| Phone numbers | Store E.164 (`+9665XXXXXXXX`). Saudi mobile input pattern `^05[503649187]\d{7}$`, other GCC patterns per country config |
| Gateway routes | `/api/{module}/**` → module hosts, `/ai-api/**` → ai-service, `/connect/**` → auth host |
| Error shape | `{ "error": { "code", "date", "messages": [], "source" } }` in every service |
| Permission strings | `Permissions.{Area}.{Action}`, identical in backend, mobile, AI service and web |

### Local dev ports

| Service | Port |
|---|---|
| BFF (YARP) | 5000 |
| Auth (OpenIddict) | 5001 |
| FamilyCircles / CarePlans / CheckIns / Alerts / Notifications / Caregivers | 5101 – 5106 |
| Jobs (Hangfire) | 5200 |
| ai-service | 8000 |
| frontend apps | 4200 (family-web), 4300 (care-dashboard) |

---

## 5. Emergency numbers (per-country config, never hard-coded)

Defaults below **must be verified with official sources before launch** and kept in backend configuration, synced to the phone for offline use.

| Country | Ambulance / emergency default |
|---|---|
| Saudi Arabia | 997 (Red Crescent), 911 (unified, where available) |
| UAE | 998 (ambulance), 999 |
| Kuwait | 112 |
| Qatar | 999 |
| Bahrain | 999 |
| Oman | 9999 |

---

## 6. Data residency

- All personal and health data (accounts, circles, care plans, check-in transcripts, voice audio, alerts, notes, embeddings, logs that may contain personal data) is stored in Saudi Arabia.
- Every cloud service on the data path (database, blob, search index, Azure OpenAI, Azure AI Speech, Redis, RabbitMQ, logs/APM) must run in an in-Kingdom region. **The hosting region per service is an open decision** (see `docs/` architecture, step 5). If a service is not available in-Kingdom, stop and raise it; do not silently pick another region.
- Push (FCM/APNs) and SMS/voice providers receive only the minimum: no health details in push or SMS text ("Rafeeq: please open the app — an alert about your mother"), never medication names or symptoms.

---

## 7. Working agreements for Claude

1. **Stop after each step** listed in the current plan so the owner can review. Do not start the next step uninvited.
2. **No code for reminders, check-ins or alert escalation** until the architecture in `docs/architecture.md` is approved.
3. One branch and one pull request per step or feature. Branch names: `docs/{topic}`, `feature/{module}-{short-name}`, `fix/{short-name}`.
4. Every user-facing string exists in Arabic and English in the same change. Arabic is written first and reviewed as Arabic, not translated word by word.
5. Any change touching emergency detection, alerts, reminders or AI safety must update `docs/safety.md` test cases and add tests for them.
6. Never put a secret, key, token or real personal data in the repo, in logs, or in test fixtures. Fixtures use invented names and `+9665000000XX` numbers.
7. Ask before choosing a vendor (SMS, voice call, hosting, analytics). Record decisions as ADRs in `docs/adr/`.
