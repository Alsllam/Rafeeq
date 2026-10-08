# Rafeeq docs

No stack skill applies here. These documents are the source of truth for every other folder: code follows the docs, and a behaviour change updates the docs in the same PR.

## Planned documents

| File | Content | Step |
|---|---|---|
| `SRS.md` | Vision, personas, roles and permissions, functional requirements per module, AI and safety requirements, emergency escalation, privacy and consent, non-functional requirements, MVP and roadmap | 2 |
| `safety.md` | Every case where the AI must refuse, hand over to a human, or trigger an alert, with Arabic and English test cases | 3 |
| `brand/` | Logo SVGs (all variants), color tokens (light / dark / high-contrast), fonts and type scale, motion tokens | 4 |
| `architecture.md` | Reminders, check-ins and alert escalation: phone vs server, offline and dead-battery behaviour | 5 |
| `adr/NNNN-title.md` | Architecture decision records (vendors, regions, trade-offs) | as needed |

## Conventions

- Markdown, one H1 per file, numbered sections so code and tests can cite them (`SRS §5.3`, `safety S-12`).
- Requirement ids: `FR-{MODULE}-{NN}` (e.g. `FR-ALR-03`), `NFR-{NN}`, `AI-{NN}`, `PRV-{NN}`. Safety cases: `S-{NN}`. Ids never get reused after deletion.
- Use the glossary in the root `CLAUDE.md`. Write "elderly person" or "parent" consistently with it.
- Arabic examples are written as natural Arabic (Gulf/Saudi and MSA where both matter), not translated word by word, and are marked with their variety: `[SA]`, `[Gulf]`, `[MSA]`.
- Safety test cases are machine-readable tables (id, language, input, expected action) so `ai-service/evals` can generate datasets from them.
- Diagrams in Mermaid, so they diff and render on GitHub.
- No real personal data in examples. Invented names, `+9665000000XX` numbers.
