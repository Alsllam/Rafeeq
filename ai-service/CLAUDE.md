# Rafeeq AI service

**Skill:** load `.claude/skills/python-azure-rag-service/SKILL.md` before any work here. Follow it for the stack, layout, Azure OpenAI client, retrieval, generation, tools, errors, safety, evaluation and deployment. This file gives the Rafeeq values and the places where Rafeeq deliberately differs. Also read the root `CLAUDE.md` (hard rules) and `docs/safety.md`.

Status: not started.

---

## 1. Names

| Placeholder | Value |
|---|---|
| `{product}` | `rafeeq` |
| Python package | `rafeeq_ai` (`src/rafeeq_ai/`, replaces `ai_service` in the skill layout) |
| Tenant | The **family circle** (`circle_id`). Care companies are an extra grouping above circles, later |
| Audience | `ai-api` |
| Port | 8000 |

## 2. What this service does here

Not a document Q&A assistant. It is a **companion** for one elderly person:
1. **Voice in, voice out:** Azure AI Speech for Arabic STT and TTS.
2. **Conversation:** friendly, respectful daily conversation and scripted check-ins.
3. **Memory per person:** what the person likes to talk about, family names, routines — stored per elderly person, visible and deletable by them and the primary family member.
4. **RAG over that person's own care plan and family notes** (not shared documents). Used to answer "what medicine do I take now?" or "when is my appointment?" by repeating the care plan.
5. **Signals:** extracts mood, sleep, activity and "worrying words" as structured output for the family summary. Never a diagnosis.

## 3. Where Rafeeq differs from the skill

### Voice: Azure AI Speech, not OpenAI audio
- STT: Azure AI Speech with `ar-SA` as primary and Gulf locales (`ar-AE`, `ar-KW`, `ar-QA`, `ar-BH`, `ar-OM`) as candidates for language identification; MSA through `ar-SA`. Use a phrase list per person (family names, medication names from their care plan) to improve recognition.
- TTS: Azure neural Arabic voices (e.g. `ar-SA-HamedNeural`, `ar-SA-ZariyahNeural`), slower rate (about -10% to -15%) and clear pauses by default; rate and voice adjustable per person. Use SSML.
- `ModelRole.stt` and `ModelRole.tts` map to Azure Speech config, not OpenAI deployments. Endpoints `/ai-api/transcribe` and `/ai-api/speak` keep the skill's contract.
- Main endpoint: `POST /ai-api/companion/turn` (audio or text in → safety → reply text + audio + structured signals out, SSE).

### Language and register
- Reply in the person's own variety: Gulf/Saudi Arabic when they speak it, MSA when they use MSA or prefer it (profile setting). Skill §7 "formal MSA" is overridden for conversation; care-plan facts are always repeated exactly as written.
- Formal and warm: use respectful address (حضرتك / يا والدي / يا والدتي per profile preference), never childish, never over-familiar, no emoji.
- Short sentences, one question at a time, patient with repetition.

### Safety layer (stricter than skill §10)
Order on every turn, full list in `docs/safety.md`:
1. **Deterministic emergency detector first**, on the raw transcript, before any model call: phrase lists in Arabic dialects + MSA + English (falling, chest pain, can't breathe, etc.). A hit publishes `EmergencySignalDetected` to the backend immediately and returns a fixed scripted reply. **It does not wait for, and cannot be overruled by, the model.**
2. Prompt Shields / content filter.
3. Medical-intent classifier (`fast` model, structured output): diagnosis, dosage, drug interaction, symptom questions → fixed "please ask your doctor" reply + optional family notification. No generation.
4. Only then the companion model, with instructions that forbid medical advice, and grounded on the care plan for any fact.
5. **Output check:** any medication name, dose or time in the reply must match the care plan exactly; otherwise replace with the safe fallback. Never send unchecked model text to TTS.

- If the model, Search or Speech is down, the service returns scripted fallbacks; the backend emergency path does not depend on this service at all.
- Self-harm or abuse disclosures follow the handover rules in `docs/safety.md`.

### Data and privacy
- Retrieval security filter: `circle_id` **and** `elderly_person_id` from the validated token and grants, never from the body. A relative only retrieves what their grants allow.
- Index: one index for the product, filtered per person; fields `care_plan_version`, `source_type` (`medication`, `appointment`, `task`, `note`, `memory`).
- Care plan changes arrive as `CarePlanChanged` events and re-index that person only. Medication facts are **also** read live from the CarePlans API for answers, so the index never serves a stale dose.
- Raw audio is not stored by default. Transcripts follow the retention set in the SRS. `store=False` always.
- Everything (OpenAI, Speech, Search, SQL, Blob, Redis) in the in-Kingdom region decided in the architecture step.

### Evaluation
`evals/datasets/rafeeq_{ar,en}.jsonl` plus `evals/datasets/rafeeq_safety_{ar,en}.jsonl` generated from `docs/safety.md`. Safety thresholds are absolute: **100%** of emergency phrases detected, **100%** of dose/diagnosis requests refused, 0 invented medication facts. CI fails on any miss.
