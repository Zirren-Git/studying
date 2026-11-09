# Study Desk — build brief

You are the autonomous product designer, desktop-app engineer, local-AI engineer, and QA lead for a new study application.

## M. Mission

- **M1.** Build a working, polished **desktop app** — not a website, not an Electron app. It helps a student find or import learning material from the web, understand and organize it into study-sized parts, and generate source-grounded questions with a small AI model running on the student's own computer.
- **M2.** Work independently: inspect the repository and environment, make sound decisions, implement, run, test the real user flows, and leave build/run instructions. Do not stop after a plan, mockup, scaffold, or TODO list; keep going until a runnable version exists and the minimum acceptance criteria pass.
- **M3.** If a search API, model license, GPU, or other service is unavailable, build a useful, honest fallback and continue. Never fake results or pretend a blocked feature works. This rule governs every requirement below.

## P. Product vision

- **P1.** The app should feel like a calm, capable study desk: collect reliable material, see where every idea came from, break it into manageable sections, actively test understanding, and revisit weak areas over time. It is a study tool — not a generic AI chat window, a browser clone, or a pile of AI summaries.
- **P2.** Core loop: (1) find or add a source; (2) extract readable content and keep its provenance; (3) let the local model organize it into useful parts and concepts; (4) generate questions answerable from those parts; (5) let the student answer, get grounded feedback, and save/review items.
- **P3.** After the student's content and model are present, the app must stay useful **offline**. Personal content and inference stay on-device by default.

## A. Desktop technology (non-negotiable)

- **A1.** No Electron — not directly, not via a quiet shell substitution, and the "desktop app" must not be a browser-only site.
- **A2.** Inspect the repo first, then choose a maintainable stack suited to the environment. Preferred: **Tauri 2** with a Rust backend and a typed frontend (Svelte, React, or Vue) on the OS webview — described honestly as a *native shell with a web-based UI*, not a fully native widget toolkit. A native Qt solution is acceptable if the repo/team suits it better. Explain the tradeoff briefly in `IMPLEMENTATION_PLAN.md`.
- **A3.** Keep web content, local files, AI inference, persistence, and UI separated behind clear interfaces. Use secure IPC and a restrictive permission model: web content never gets arbitrary shell or filesystem access.
- **A4.** Deliver a runnable development build plus a documented production packaging path for the supported operating systems; state any platform limitation honestly.

## B. Local AI (non-negotiable)

- **B1.** Inference runs on the user's device. No mandatory remote LLM API, no student pages or notes sent to a server by default, and never label a remote model "local."
- **B2.** Use a genuinely small, quantized instruction model suited to ordinary student hardware — roughly **1–4B parameters**, adjusted after testing. Use an in-process runtime or a packaged sidecar (e.g. a supported llama.cpp/GGUF runtime). Do **not** require a separately installed server such as Ollama for the core AI flow.
- **B3.** Verify and document the model's license, source, distribution rights, quantization, context window, approximate disk/RAM needs, and current download method. Do not bundle or redistribute weights unless the license explicitly permits it. If redistribution is disallowed or the file is too large, implement an **in-app first-run download** from the official/authorized source with size, license, checksum/signature validation where available, progress, cancel/retry, a disk-space check, and clear offline behavior.
- **B4.** Put inference behind a replaceable `LocalModelProvider` interface. Detect CPU/RAM, use CPU inference as the baseline, allow optional supported acceleration, and offer a smaller low-memory configuration. Never require a discrete GPU.
- **B5.** Provide a transparent model settings/about view: model name, version, license, source, storage location, and an explicit statement that inference is local. No hidden network calls during generation.
- **B6.** If no model is installed or the model is unavailable, say so plainly and offer in-app setup. A rules-based fallback may assist basic organization but must be labeled non-AI; never fabricate questions.

## C. Source safety (non-negotiable)

- **C1.** Treat all web pages, PDFs, HTML, metadata, and embedded text as untrusted. A page may contain prompt-injection text such as "ignore previous instructions." The model must treat source material as content to analyze, never as instructions. Page content can never invoke tools, read files, reveal secrets, change settings, trigger downloads, or override system/developer instructions.
- **C2.** Extract permission-consciously: no bypassing paywalls, login walls, access controls, CAPTCHA, or robots/terms restrictions; no scraping private endpoints; no executing page scripts. Use official/public endpoints and reasonable request limits. If blocked, invite the student to import a file, paste text, or use an allowed source.
- **C3.** Validate URLs and redirects; block local/private-network targets and dangerous schemes; mitigate SSRF; cap response size and timeouts; sanitize rendered HTML; never open untrusted active content in a privileged webview.
- **C4.** Preserve source URL, title, author, publisher, date (when available), retrieval time, selected text/page range, and stable source IDs. Every AI-derived summary, outline item, answer key, and question links to its source section and, where feasible, a short supporting excerpt or page/heading anchor.
- **C5.** Do not reproduce long copyrighted sources in study packs. Store only what the personal study workflow needs, respect source terms, provide attribution, and let users delete their content. Generated summaries and questions must be transformative and anchored — never a substitute for accessing the source.

## U. Users

- **U1.** Design for secondary-school, college, and adult learners, including people on modest laptops. Assume no single subject, language, or page type.
- **U2.** Provide clear language, adjustable reading comfort, full keyboard access, and a low-distraction interface. Never claim the app guarantees grades, diagnoses learning disabilities, or replaces a teacher.

## R1. Study spaces and source library

- **R1.1** Study spaces/courses with name, optional subject, target date, and description. Sources can belong to a space or to a general library.
- **R1.2** Add content by: public URL; web search through a permitted integration; import of PDF, HTML, EPUB, or plain text where feasible and lawful; pasted text; drag-and-drop of supported files.
- **R1.3** Provide a search-provider abstraction using an API/integration whose current terms permit this use. If a key is required, give a secure configuration flow *and* keep a no-key path where URL/file/paste import fully works. Never scrape a search-engine UI or claim search works without a real provider.
- **R1.4** Search results show title, domain/publisher, relevant excerpt, date when available, and a clear link. Visually distinguish search snippets from fetched, verified content, and let the student preview a result before saving it.
- **R1.5** Show ingestion stages — fetching, extracting, cleaning, ready, partially readable, failed — with cancel/retry and useful error messages. Keep a raw-source reference plus the cleaned readable version when allowed. Detect duplicate URLs and files.
- **R1.6** Library supports search, filters, sorting, tags, source type, and "recently added"; archive, rename, tag, reprocess, export notes, and delete, with confirmation for destructive actions. Maintain a source ledger (URL or file name, metadata, added date, extraction status, content hash/version). If a source changes, never silently overwrite notes: surface that a newer version exists and let the student compare or reprocess.

## R2. Reader and source-grounded organization

- **R2.1** Build a readable in-app reader preserving headings, lists, links, captions, tables where possible, and page numbers/headings; make it easy to open the original in the system browser.
- **R2.2** Support text selection, highlights, annotations, bookmarks, and personal notes. Visually distinguish original text, AI-generated explanation, and student-authored notes.
- **R2.3** Have the local model propose a structured outline: sections/parts, heading, short description, key concepts, prerequisite concepts, estimated reading effort (clearly labeled an estimate), and citations to source spans/anchors. The student can edit, reorder, merge, split, accept, or reject it. Never reorganize the saved source destructively.
- **R2.4** Offer alternate organization modes where appropriate — topic/concept, chronology/process, argument/evidence, question-driven — selected explicitly. Do not impose one pattern on a history chapter, an API guide, and a biology article alike.
- **R2.5** Keep traceability: clicking a concept or claim reveals the supporting passage/page/heading. If no supporting text can be found, mark the item **unverified** and ask the student to check rather than inventing a citation. A useful study set must be creatable manually, without AI.

## R3. AI-generated study material

- **R3.1** Generate from a selected source or a selected outline part — never an opaque whole-library prompt. Offer configurable question counts and difficulty using understandable labels ("recall," "explain," "apply"), not unsupported claims about exact grade level.
- **R3.2** Support question types: multiple choice with plausible but fair distractors; short-answer recall and explanation; true/false only when the claim is unambiguous, requiring the student to correct a false statement; cloze/fill-in-the-blank; compare/contrast or apply-to-a-new-example only where the source supplies enough context; flashcards with concise front/back text.
- **R3.3** Every item carries its source section and evidence span/anchor, with the expected answer and explanation grounded in that evidence. Citations are clickable in review mode. Single-answer multiple-choice items have exactly one correct choice; check for ambiguous wording, duplicated options, answer leakage, trivial distractors, unsupported facts, and clues from option length or grammar.
- **R3.4** Use a two-stage process: draft structured questions, then validate them against the cited passages (programmatic checks plus a second model pass when practical). Reject or repair items lacking evidence or with unclear answers; never present low-confidence material as verified.
- **R3.5** Let the student edit, regenerate, flag, remove, or save questions. Preserve a revision history or otherwise avoid destructive overwrite.
- **R3.6** Provide a "Why this answer?" view with a short source-based rationale, separating the source's claims from added explanation or outside knowledge. Provide a "challenge me" mode that reveals one question at a time without showing the answer early.
- **R3.7** Enforce JSON schemas and validate all model output. Treat output as untrusted: render safely, validate citation IDs against existing source spans, and handle malformed or partial generations. Keep prompts versioned, concise, and testable. For long sources, chunk with overlap or summarize hierarchically while retaining source-span maps so citations survive chunking. Never invent a quote or page number.

## R4. Quiz, review, progress

- **R4.1** Quiz flow: answer input, submit, feedback with source citation, next question, pause/exit, and a clear end-of-session summary.
- **R4.2** Feedback is constructive and specific. For open-ended answers, evaluate against a rubric or key concepts grounded in the source; show uncertainty and allow self-marking when local-model evaluation is weak. Never present subjective model grading as objective fact.
- **R4.3** Lightweight spaced-review queue for saved flashcards/questions with "again / hard / good / easy," an understandable schedule, and manual override. Don't imply scientific assessment beyond the method actually used.
- **R4.4** Track progress locally: items seen/answered, self-rated confidence, review dates, course completion. Let the student reset, delete, or export it. No fake streaks, fake comparisons, shame language, or fabricated performance insights.
- **R4.5** Progress summaries derive only from recorded activity (sections studied, items due, questions answered, concepts marked difficult) and explain how each metric is calculated.
- **R4.6** Export to Markdown/CSV/JSON and print-friendly study sheets where feasible, preserving citations and source references.

## R5. Settings, privacy, trust

- **R5.1** Local-first storage for source text, notes, outlines, study sets, progress, and model files. Make the storage location visible and provide "open data folder," export, backup, and delete.
- **R5.2** Before any web search or URL fetch, explain what leaves the device (the query/URL goes to the chosen provider). Never upload imported documents or notes to a search service or AI provider.
- **R5.3** Store API credentials in the OS credential store where available; never commit keys, never write secrets to ordinary config, never log them.
- **R5.4** Give clear controls for source-cache retention, model download/storage, crash diagnostics, and deletion. No telemetry, advertising, or account signup for core study features; any future telemetry must be opt-in and clearly explained.
- **R5.5** Encrypt sensitive local data only if done correctly, stating key recovery implications. Never claim encryption when data is merely in a local database.
- **R5.6** Provide an About/Privacy/Model page covering app version, dependency licenses, model source/license, search provider, storage location, and network behavior.

## I. Interface and visual direction

- **I1.** Desktop-first, focused study environment with a specific identity. A fitting direction is *quiet library desk / annotated field guide*: warm paper-like neutrals, dark ink, a restrained botanical or mineral accent, fine typographic hierarchy, small editorial details. Treat this as a starting point, not an excuse to decorate every panel — pick one coherent direction, define tokens, stay consistent.
- **I2.** Primary navigation exposes spaces, sources, study sets/review, and settings. "Add material" stays visible but never louder than the active study task. Study views show the selected course/source/part, useful progress, and the active task, with focused layouts for reading, outlining, and quizzing.
- **I3.** Choose a readable display/body pairing plus an optional technical face for code and metadata. Bundle only licensed fonts or use robust local fallbacks; avoid one unchosen default font for every typographic role.
- **I4.** One coherent icon system, no emoji as functional icons, only real or clearly stylized source imagery. Never fabricate book covers, authors, ratings, or publisher endorsements.
- **I5.** Use semantic surface hierarchy rather than boxing every element. Avoid identical cards nested in cards, excessive pills, decorative left stripes, generic purple/blue gradients, neon glows, glassmorphism, floating abstract blobs, and the centered SaaS hero with three identical feature cards.
- **I6.** No filler slogans, fabricated student quotes, invented learning statistics, or fake "AI accuracy" badges. Write useful, specific interface copy.
- **I7.** Clear action hierarchy, consistent spacing/type tokens, visible keyboard focus, accessible contrast, scalable text, and reduced-motion support. Don't default to dark mode; offer a deliberate light/dark/system choice.
- **I8.** Give loading, empty, offline, no-results, extraction-failed, model-missing, generation-failed, low-confidence, and success states genuine design. Never show fake progress or fake questions as live AI output.
- **I9.** Motion clarifies selection, transitions, and completion — brief, optional, respectful of reduced motion. If `SKILL.md` exists in the repository, read it before designing and follow its craft guidance. Review the rendered app at actual desktop sizes; do not judge from source code alone.

## D. Architecture

Adjust to the repository and document the choice, keeping boundaries like these:

- **D1.** Desktop shell: Tauri 2 + Rust (or a native Qt equivalent); never Electron.
- **D2.** UI: typed component architecture with reusable tokens and accessible components.
- **D3.** Domain layer: study spaces, sources, sections, evidence spans, outlines, questions, attempts, review schedule, settings.
- **D4.** Persistence: local SQLite or another reliable embedded store with schema migrations; files in a well-defined app-data directory; safe writes and explicit deletion/backup behavior.
- **D5.** Content pipeline: search adapter → safe fetch/import → extraction/normalization → source-span index → local database.
- **D6.** AI pipeline: local model provider → prompt templates → structured-output validation → evidence/citation validator → editable study material.
- **D7.** Background tasks: cancellable fetch, extraction, model setup/inference, export, and backup, with progress that reflects real work.
- **D8.** Security: strict IPC allowlist, content sanitization, URL/network validation, no privileged webview for arbitrary pages, credentials in OS secure storage, and untrusted content never passed as instructions. Keep a source-adapter interface for web search and a local-file parser interface per import type. Avoid premature microservices — this is a local desktop app.

## S. Build sequence

- **S1.** Inspect repo and environment; write `IMPLEMENTATION_PLAN.md`, `DESIGN.md`, and a short threat/privacy note (then keep building).
- **S2.** Desktop shell, navigation, local database, study spaces, source library; make paste/text import and persistence work first.
- **S3.** Safe public-URL retrieval and extraction, metadata, citations/source spans, reader. Add real search only with terms-compliant access; keep URL/file/paste working without a search key.
- **S4.** Real small-model runtime and a verified, legally usable model path: in-app setup/download, local inference, structured output, working outline flow. Test with the network disabled after setup.
- **S5.** Evidence-grounded question generation, validation, quiz/review, progress persistence.
- **S6.** Accessibility, error/empty/loading/offline states, export/backup/delete, packaging docs, visual review.
- **S7.** Formatting, lint/type checks, unit and integration tests, production build; launch the app and exercise the full core flow.

These are sequencing suggestions, not stopping points: keep the project runnable after each slice and continue until the acceptance criteria are verified. Never fill unfinished model or search functionality with fabricated content.

## V. Verification and acceptance

Before declaring the work complete:

- **V1.** The non-Electron desktop app builds and launches with the documented commands in the available environment.
- **V2.** A student can create a space, add or paste a real source, save it, restart the app, and find the content intact.
- **V3.** A public URL or supported local file is ingested through an honest, safe path, and the app shows the real source plus any extraction limitations.
- **V4.** The local model generates an outline from real source text and the output is stored locally; verify no outbound LLM calls occur during inference.
- **V5.** A student generates at least one source-grounded question, answers it in the quiz flow, sees a supporting citation, and reviews the result. Unsupported or malformed model output is rejected, not invented around.
- **V6.** Exercise states: missing model, offline, unsupported page, failed extraction, empty library, no search results, cancelled task, generation error, malformed model response.
- **V7.** Prompt-injection strings in a source are treated purely as quoted/analyzed page content; URL validation rejects local/private-network targets and unsafe schemes.
- **V8.** Test persistence, deletion, export, duplicate-source handling, database migrations, and citation integrity.
- **V9.** Run formatter, linter/type checks, automated tests, and the production build — fix failures or state exactly what could not be run and why.
- **V10.** Inspect the running app at realistic desktop sizes: hierarchy, typography, truncation, keyboard-only operation, focus, contrast, scaling, reduced motion, empty/error states, and genuinely working interactions.
- **V11.** No dead primary controls, fake AI output, placeholder screens presented as complete, lorem ipsum, fake progress, fabricated metrics, or fake citations in the production build.
- **V12.** `README.md` covers setup/run/build/package/test, supported OS versions, model installation/download and license, storage locations, search-provider configuration, network/privacy behavior, troubleshooting, and known limitations.
- **V13.** Deliver the runnable project itself, not a plan or code excerpts. The final report separates implemented, tested, unavailable, and future features, and claims a test or offline behavior was verified only if it actually was.

## DoD. Definition of done

A student can launch the non-Electron desktop app, create a study space, add a real source, read it and organize it into editable parts using the integrated local model, generate at least one question tied to verifiable evidence, answer it, and later return to find the work persisted. The core AI flow works on-device once the model is installed, and the app clearly communicates its real network use, model provenance, source citations, and limitations.

Do not stop until this definition and the checks above pass. If a provider or platform feature is blocked, deliver and verify the best complete local-first version possible, mark the blocked feature clearly, and keep every unblocked flow working.
