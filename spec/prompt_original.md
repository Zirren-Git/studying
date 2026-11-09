You are the autonomous product designer, desktop-app engineer, local-AI engineer, and QA lead for a new study application. Build a working, polished desktop app—not a website and not an Electron app. The app helps a student find or import learning materials from the web, understand and organize them into study-sized parts, and generate source-grounded questions with a small AI model that runs locally on the student's computer.

Work independently. Inspect the repository and available environment, make sound decisions, implement the app, run it, test the real user flows, and leave build/run instructions. Do not stop after a plan, mockup, scaffold, or TODO list. Keep working until a runnable version is delivered and the minimum acceptance criteria pass. If an external search API, model license, GPU, or other service is unavailable, build a useful honest fallback and continue; never fake results or pretend a blocked feature works.

Product vision
The app should feel like a calm, capable study desk: a student can collect reliable material, see where every idea came from, break it into manageable sections, actively test understanding, and revisit weak areas over time. It is a study tool, not a generic AI chat window, a web browser clone, or a pile of AI-generated summaries.

The core loop is:

Find or add a source.
Extract readable content and keep its provenance.
Let the local model organize the source into useful parts and concepts.
Generate questions that are answerable from those parts.
Let the student answer, get grounded feedback, and save/review learning items.
The app must remain useful when offline after the student's content and model are available. Personal content and inference should stay on-device by default.

Mandatory non-negotiables
Desktop technology
Do not use Electron. Do not quietly substitute an Electron shell or make the "desktop app" a browser-only site.
Inspect the repository first. Choose a maintainable desktop stack suited to the environment. A preferred option is Tauri 2 with a Rust backend and a typed frontend (Svelte, React, or Vue), using the OS webview; clearly describe this architecture as a native shell with web-based UI, not as a fully native widget toolkit. A native Qt solution is also acceptable if the repository/team is better suited to it. Explain the tradeoff briefly in IMPLEMENTATION_PLAN.md.
Keep web content, local files, AI inference, persistence, and UI separated behind clear interfaces. Use secure IPC and a restrictive application permission model. Do not expose arbitrary shell or filesystem access to web content.
Deliver a runnable development build and documented production packaging path for supported operating systems. State any platform limitation honestly.
Local AI is real and integrated
The built-in AI must run on the user's device. Do not make a remote LLM API mandatory, do not send a student's pages or notes to a server by default, and do not label a remote model as local.
Use a genuinely small, quantized instruction model suitable for ordinary student hardware (target a compact model in roughly the 1–4B parameter range, adjusting after testing). Use an in-process local runtime or a packaged sidecar such as a supported llama.cpp/GGUF-based runtime. Do not require a separate manually installed server like Ollama for the app's core AI flow.
Verify the selected model's license, source, distribution rights, quantization, supported context, approximate disk/RAM needs, and current download method. Do not bundle or redistribute weights unless the license explicitly allows it. If distributing weights is not allowed or too large, integrate an in-app first-run model download from the official/authorized source, with size, license, checksum/signature validation where available, progress, cancel/retry, disk-space check, and clear offline behavior.
Keep the inference layer replaceable behind a LocalModelProvider interface. Detect CPU/RAM capabilities, use CPU inference as the baseline, optionally use supported acceleration, and allow a smaller/low-memory configuration. Do not require a discrete GPU.
Include a transparent model settings/about view showing model name, version, license, source, storage location, and whether inference is local. No hidden network calls during generation.
If a model is not installed or unavailable, say so and offer the in-app model setup. A rules-based fallback may help with basic organization, but label it as non-AI; never fake generated questions.
Reliable, safe source handling
Treat all web pages, PDFs, HTML, metadata, and embedded text as untrusted input. A page can contain malicious prompt-injection text such as "ignore previous instructions." The model must treat source material as content to analyze, not instructions to obey. Never let page content invoke tools, access files, reveal secrets, change settings, trigger downloads, or override system/developer instructions.
Use safe, permission-conscious extraction. Do not bypass paywalls, login walls, access controls, CAPTCHA, or robots/terms restrictions; do not scrape private endpoints; do not execute page scripts. Use official/public endpoints and reasonable request limits. If blocked, invite the student to import a file, paste text, or use an allowed source.
Validate URLs and redirects, block local/private-network targets and dangerous schemes where appropriate, mitigate SSRF, cap response sizes and timeouts, sanitize rendered HTML, and avoid opening untrusted active content in a privileged webview.
Preserve source URL/title/author/publisher/date when available, retrieval time, selected text/page range, and stable source IDs. Every AI-derived summary, outline item, answer key, and question should link to the relevant source section and, where feasible, a short supporting excerpt or page/heading anchor.
Do not reproduce long copyrighted sources in generated study packs. Store only what is needed for the user's personal study workflow, respect source terms, provide attribution, and let users delete their content. Generated summaries/questions must be transformative and anchored, not a substitute for accessing the source.
Target users and product behavior
Design for secondary-school, college, and adult learners, including people studying on modest laptops. Do not assume one subject, one language, or one kind of web page. Provide clear language, adjustable reading comfort, keyboard access, and a low-distraction interface. Avoid claiming that the app guarantees grades, diagnoses learning disabilities, or replaces a teacher.

Core product requirements
1. Study spaces and source library
Create study spaces/courses with a name, optional subject, target date, and description. Let students add sources to a space or a general library.
Add content by: entering a public URL; searching the web using a permitted search integration; importing PDF, HTML, EPUB or plain text where feasible and lawful; or pasting text. Support drag-and-drop for supported local files.
Provide a search provider abstraction. Use an API or integration whose current terms permit this use. If keys are required, provide a secure configuration flow and a no-key path that still supports URL/file/paste import. Do not scrape a search-engine UI or claim search works without a real provider.
Search results should display source title, domain/publisher, relevant excerpt, date when available, and a clear link. Distinguish search-result snippets from fetched/verified source content. Let the student preview a result before saving it.
Show source-ingestion stages (fetching, extracting, cleaning, ready, partially readable, failed) with cancellation/retry and useful errors. Preserve a raw-source reference and a cleaned readable version when allowed. Detect duplicate URLs/files.
Offer library search, filters, sorting, tags, source type, and "recently added." Let students archive, rename, tag, reprocess, export notes, and delete content. Confirm destructive actions.
Keep a source ledger: URL or file name, source metadata, added date, extraction status, and content hash/version when useful. If the source changes, do not silently overwrite notes—show that a newer version is available and let the student compare/reprocess.
2. Reader and source-grounded organization
Build a readable in-app reader for extracted content. Preserve headings, lists, links, captions, tables where possible, and page numbers/headings. Make it easy to inspect the original source in the system browser.
Allow text selection, highlighting, annotations, bookmarks, and personal notes. Visually distinguish original text, AI-generated explanation, and student-authored notes.
Have the local model propose a structured outline: sections/parts, heading, short description, key concepts, prerequisite concepts, estimated reading effort (only if clearly presented as an estimate), and citations to source spans/anchors. Let the student edit, reorder, merge, split, accept, or reject the outline. Do not reorganize the saved source destructively.
Provide useful alternate organization modes where appropriate: topic/concept, chronology/process, argument/evidence, or question-driven. Make the selection explicit; do not force the same outline pattern on a history chapter, API guide, and biology article.
Keep traceability: clicking a concept or claim should reveal the source passage/page/heading that supports it. If the extraction cannot find supporting text, mark the item "unverified" and ask the student to check rather than inventing a citation.
Make it possible to create a useful study set manually without AI.
3. AI-generated study materials
Generate from a selected source or selected outline part, not an opaque whole-library prompt. Include configurable question counts and difficulty, but use understandable labels such as "recall," "explain," and "apply," not unsupported claims about exact grade level.

Support question types such as:

Multiple choice with plausible but fair distractors.
Short-answer recall and explanation.
True/false only when the claim is unambiguous; require the student to correct a false statement.
Cloze/fill in the blank.
Compare/contrast or apply-to-a-new-example questions only when the source supports enough context.
Flashcards with concise front/back text.
For every generated item:

Attach source section and evidence span/anchor.
Keep the expected answer and explanation grounded in the evidence. Show citations in review mode and make them clickable.
Ensure exactly one correct choice in single-answer multiple-choice questions. Check for ambiguous wording, duplicated options, answer leakage, trivial distractors, unsupported facts, and accidental clues from option length/grammar.
Use question generation as a two-stage process: draft structured questions, then validate them against cited source passages (programmatic checks plus a second model pass when practical). Reject/repair items that lack evidence or have unclear answers. Do not silently present low-confidence material as verified.
Allow the student to edit, regenerate, flag, remove, or save questions. Preserve a revision history or avoid destructive overwrite.
Provide a "why this answer?" view that shows a short source-based rationale. Separate the source's claims from additional explanation or outside knowledge.
Include a "challenge me" mode that reveals one question at a time without showing the answer prematurely.
Use structured JSON schemas and validate model output. Treat model output as untrusted: render safely, validate citation IDs against existing source spans, and handle malformed/partial generations. Keep prompts versioned, concise, and testable. Use chunking with overlap or hierarchical summarization for long sources; retain source-span maps so citations survive chunking. Never invent a quote or page number.
4. Quiz, review, and progress
Make a quiz flow with answer input, submit, feedback, source citation, next question, pause/exit, and clear end-of-session summary.
Keep feedback constructive and specific. If the response is open-ended, evaluate against a rubric or key concepts grounded in the source; show uncertainty and allow self-marking when local-model evaluation is weak. Do not present subjective model grading as objective fact.
Support a lightweight spaced-review queue for saved flashcards/questions. Let users mark "again," "hard," "good," or "easy," with an understandable schedule and manual override. Do not imply a scientific assessment beyond the method used.
Track progress locally: seen/answered items, self-rated confidence, review dates, and course completion state. Let the student reset/delete/export it. Do not gamify with fake streaks, fake comparisons, shame language, or fabricated performance insights.
Provide useful progress summaries based only on recorded activity: e.g., sections studied, items due, questions answered, and concepts the student marked difficult. Explain how a metric is calculated.
Offer exports to Markdown/CSV/JSON or print-friendly study sheets where feasible. Keep citations and source references in exports.
5. Settings, privacy, and trust
Default to local-first storage for source text, notes, outlines, generated study sets, progress, and model files. Make storage location visible and provide "open data folder," export, backup, and delete controls.
Before using web search or fetching a URL, explain what leaves the device (the query/URL is sent to the selected provider). Do not upload imported documents or notes to a search service or AI provider.
Store API credentials in the OS credential store where available. Never commit keys, write secrets to ordinary config, or include them in logs.
Provide clear controls for source cache retention, model download/storage, crash diagnostics, and deletion. No telemetry, advertising, or account signup required for core study features. If optional telemetry is later added, it must be opt-in and clearly explained.
Encrypt sensitive local data only if implemented correctly and provide key recovery implications; do not claim encryption when data is merely stored in a local database.
Add an About/Privacy/Model page with app version, dependencies/licenses, model source/license, search provider, storage location, and network behavior.
Interface and visual direction
Create a desktop-first, focused study environment with a specific visual identity. A fitting direction is quiet library desk / annotated field guide: warm paper-like neutrals, dark ink, a restrained botanical or mineral accent, fine typographic hierarchy, and small editorial details. This is a starting point, not an excuse to decorate every panel. Choose one coherent direction, define tokens, and keep it consistent.

The primary navigation should make spaces, sources, study sets/review, and settings easy to find. Keep "add material" visible but not louder than the active study task.
Study views should show the selected course/source/part, useful progress, and the active task. Use focused layouts for reading, outlining, and quiz sessions.
Choose a readable display/body pairing and an optional technical face for code/metadata. Bundle only licensed fonts or use robust local fallbacks. Avoid a single unchosen default font across every typographic role.
Use one coherent icon system, no emoji as functional icons, and only real or clearly stylized source imagery. Never fabricate book covers, source authors, ratings, or publisher endorsements.
Use semantic surface hierarchy instead of boxing every element. Avoid identical cards nested in cards, excessive pills, decorative left stripes, generic purple/blue gradients, neon glows, glassmorphism, floating abstract blobs, and the standard centered SaaS hero with three identical feature cards.
Do not use filler slogans, fabricated student quotes, invented learning statistics, or fake "AI accuracy" badges. Write useful, specific interface copy.
Use a clear action hierarchy, consistent spacing/type tokens, visible keyboard focus, accessible contrast, scalable text, and reduced-motion support. Do not make dark mode the default reflex; offer a deliberate light/dark/system choice.
Give loading, empty, offline, no-results, extraction-failed, model-missing, generation-failed, low-confidence, and success states genuine design. Never display fake progress or fake questions as live AI output.
Motion should clarify selection, transitions, and completion. Keep it brief, optional, and respectful of reduced-motion settings.
If 
SKILL.md
 exists in the provided repository, read it before designing and follow its craft guidance. The rendered app must be reviewed at actual desktop sizes; do not judge it only from source code.

Suggested architecture
Adjust to the repository and document the choice, but keep clear boundaries similar to:

Desktop shell: Tauri 2 + Rust, or a native Qt equivalent; never Electron.
UI: typed component architecture with reusable tokens and accessible components.
Domain layer: study spaces, sources, sections, evidence spans, outlines, questions, attempts, review schedule, settings.
Persistence: local SQLite or another reliable embedded store with schema migrations; files stored in a well-defined app-data directory. Use safe writes and explicit deletion/backup behavior.
Content pipeline: search adapter → safe fetch/import → extraction/normalization → source-span index → local database.
AI pipeline: local model provider → prompt templates → structured output validation → evidence/citation validator → editable study materials.
Background tasks: cancellable fetch, extraction, model setup/inference, export, and backup tasks with progress that corresponds to actual work.
Security: strict IPC allowlist, content sanitization, URL/network validation, no privileged webview for arbitrary pages, credentials in OS secure storage, and no untrusted content passed as instructions.
Keep a source adapter interface for web search and a local file parser interface for import types. Avoid premature microservices; this is a local desktop app.

Build plan: work through to a verified version
Inspect repository and environment. Write IMPLEMENTATION_PLAN.md, DESIGN.md, and a short threat/privacy note. Do not stop there.
Implement the desktop shell, navigation, local database, study spaces, and source library. Make local text/paste import and persistence work first.
Implement safe public URL retrieval and extraction, source metadata, citations/source spans, and the reader. Add a real search integration only with terms-compliant access; keep URL/file/paste paths fully usable if no search key exists.
Integrate an actual small local model runtime and a verified, legally usable model path. Complete in-app model setup/download, local inference, structured output, and a working outline flow. Test with the network disabled after model setup.
Add evidence-grounded question generation, validation, quiz/review, and progress persistence.
Finish accessibility, error/empty/loading/offline states, export/backup/delete, packaging documentation, and visual review.
Run formatting, lint/type checks, unit/integration tests, and a production build. Launch the application and exercise the complete core flow.
These are sequencing suggestions, not stopping points. Keep the project runnable after each slice and continue until the minimum acceptance criteria are verified. Never fill unfinished model or search functionality with fabricated content.

Testing and acceptance criteria
Before saying the work is complete:

The non-Electron desktop app builds and launches using documented commands in the available environment.
A student can create a study space, add/paste a real source, save it, restart the app, and find the content still there.
A public URL or supported local file can be ingested through an honest, safe path. The app shows the real source and any extraction limitations.
The integrated local model can generate an outline from real source text, and output is stored locally. Verify there are no outbound LLM calls during inference.
A student can generate at least one source-grounded question, answer it in a quiz flow, see a supporting citation, and review the result. Reject unsupported or malformed model output rather than inventing.
Exercise missing model, offline, unsupported page, failed extraction, empty library, no search results, cancelled task, generation error, and malformed model response states.
Test source prompt-injection strings and verify they are treated only as quoted/analyzed page content. Test URL validation against local/private network targets and unsafe schemes.
Test persistence, deletion, export, duplicate-source handling, database migrations, and source citation integrity.
Run formatter, linter/type checks, automated tests, and production build; fix failures or state exactly what could not be run and why.
Inspect the actual app at realistic desktop sizes. Check hierarchy, typography, truncation, keyboard-only operation, focus, contrast, scaling, reduced motion, empty/error states, and genuine interaction behavior.
No dead primary controls, fake AI output, placeholder screens presented as complete, lorem ipsum, fake progress, fabricated metrics, or fake citations in production.
Provide README.md with setup/run/build/package/test instructions, supported OS versions, model installation/download and license details, storage locations, search-provider configuration, network/privacy behavior, troubleshooting, and known limitations.
Deliver the runnable project, not just a plan or source excerpts. Final report must distinguish implemented, tested, unavailable, and future features. Do not claim a test or offline behavior was verified unless you actually checked it.
Definition of done
A student can launch the non-Electron desktop application, create a study space, add a real source, read and organize it into editable parts using the integrated local model, generate at least one question tied to verifiable evidence, answer it, and return later to find the work persisted. The core AI flow works on-device once the model is installed. The app clearly communicates its actual network use, model provenance, source citations, and limitations.

Do not stop until this definition of done and the acceptance checks above pass. If a provider or platform feature is blocked, deliver and verify the best complete local-first version possible, mark the blocked feature clearly, and keep all unblocked flows working.
