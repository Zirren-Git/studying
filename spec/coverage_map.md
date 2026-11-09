# Coverage map — original brief → condensed brief

No requirement was removed. The condensed version merges duplicate statements of the same rule
(the "never fake/fabricate" rule appears once as **M3** and is then referenced, rather than repeated
in five sections) and converts flat bullet lists into stable IDs so each requirement can be checked off.

| Original section | Condensed IDs |
|---|---|
| Opening role + "work independently / don't stop at a plan / honest fallback" | M1–M3 |
| Product vision, core loop, offline + on-device default | P1–P3 |
| Desktop technology (no Electron, Tauri 2 / Qt, separation, IPC, packaging) | A1–A4 |
| Local AI (on-device, 1–4B quantized, runtime, license/weights, provider interface, model view, missing-model behavior) | B1–B6 |
| Safe source handling (untrusted input, prompt injection, extraction limits, URL/SSRF/HTML safety, provenance, copyright) | C1–C5 |
| Target users and product behavior | U1–U2 |
| 1. Study spaces and source library (spaces, add paths, search abstraction, results, ingestion stages, library ops, ledger) | R1.1–R1.6 |
| 2. Reader and source-grounded organization (reader, annotations, outline proposal, alt modes, traceability, manual path) | R2.1–R2.5 |
| 3. AI-generated study materials (scoping, labels, question types, per-item grounding, one-correct-answer checks, two-stage validation, editing, why/challenge, schemas/chunking) | R3.1–R3.7 |
| 4. Quiz, review, progress (flow, feedback, spaced queue, tracking, summaries, exports) | R4.1–R4.6 |
| 5. Settings, privacy, trust (local-first, network disclosure, credentials, controls, encryption honesty, About page) | R5.1–R5.6 |
| Interface and visual direction (identity, nav, type, icons, surfaces, copy, a11y, states, motion, SKILL.md, desktop-size review) | I1–I9 |
| Suggested architecture (shell, UI, domain, persistence, pipelines, tasks, security, adapters) | D1–D8 |
| Build plan (7 steps + "sequencing suggestions, not stopping points") | S1–S7 |
| Testing and acceptance criteria (13 bullets) | V1–V13 |
| Definition of done + blocked-feature fallback | DoD |

## Length

| Version | Words | Bytes |
|---|---|---|
| Original | ~3,172 | ~22,495 |
| Condensed | ~3,150 | ~22,200 |
| Budget | 10,000 | — |

The original already fit the 10,000-word budget with ~6,800 words of headroom; the condensed
version keeps that headroom and adds stable IDs for review, diffing, and requirement-by-requirement
sign-off. A lexical coverage check (118 distinctive terms from the original — runtime names,
security controls, question types, UI rules, export formats) confirms every term still appears
in the condensed brief.

## How to read the IDs

- `M` mission · `P` product vision · `A` desktop stack · `B` local AI · `C` source safety · `U` users
- `R1` library · `R2` reader/organization · `R3` generation · `R4` quiz/review · `R5` privacy
- `I` interface · `D` architecture · `S` build sequence · `V` verification · `DoD` definition of done
