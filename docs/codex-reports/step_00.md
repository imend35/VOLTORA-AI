# VOLTORA AI — STEP 00

**Status:** completed assessment only  
**Date:** 2026-11-26  
**Scope:** read-only environment discovery and preliminary architecture planning. No dependencies, images, models, services, application code, remote resources, or Git commits were created.

## Executive decision

**GO for STEP 01, with environment prerequisites tracked.** The machine has a suitable Windows/x64 development baseline, working Git, Python, Node.js/npm, Docker CLI/Compose, VS Code, and Codex CLI. It is suitable to establish a reproducible project scaffold and architecture decisions in the next approved step. It is **not yet verified as capable of container execution or local GPU-accelerated inference**: Docker Engine is unavailable and hardware/WSL inspection is access-restricted.

## Inspected environment

| Area | Result | Evidence / certainty |
|---|---|---|
| Working directory | `C:\Users\Esila Nur Demirci\VOLTORA-AI` | Verified |
| Repository | Git repository; branch `main`; no commits | Verified |
| Working tree / project files before this report | No tracked/untracked status output; `rg --files` returned no project files | Verified; this report is the only STEP 00 artifact created |
| Operating system | Windows NT `10.0.26200.10075` | Verified by non-privileged fallback; marketing edition was not readable |
| Architecture | 64-bit operating system and process | Verified |
| Git | `2.53.0.windows.1` | Verified |
| Python default | `3.13.11` | Verified (`python --version`) |
| Python launcher | `py -0p` listed Python `3.14` at `...Python314\python.exe`; the launcher did not list the default 3.13 interpreter | Verified observation; interpreter-selection configuration needs later review |
| pip | `26.2.1`, associated with Python 3.13 | Verified |
| Node.js / npm | Node `v26.8.1`; npm `11.12.1` | Verified |
| Docker CLI / Compose | Docker `29.1.3`; Compose `v5.0.1` | Verified |
| Docker Engine | Not reachable: named pipe `//./pipe/docker_engine` was absent | Verified blocker; no image pull or daemon start attempted |
| Visual Studio Code | `1.146.0`, x64 | Verified |
| Codex | `@openai/codex@0.163.0`; `codex --version` works | Verified |
| WSL | Enumeration returned `E_ACCESSDENIED` | Verification blocked by access policy |
| CPU / RAM | Not determined | CIM queries were denied; no bypass attempted |
| Disk | C: 366.82 GiB free, 585.67 GiB used (about 952.49 GiB total) | Verified at inspection time |
| GPU / VRAM | Not determined | Video-controller CIM query was denied; `nvidia-smi` and known local AI runtime commands were not available on PATH |
| Local AI runtime | No `ollama`, `llama-server`, `vllm`, or `local-ai` command found on PATH | Verified only for PATH-discoverable installs; it does not rule out other installations |

Docker emitted a warning that its user configuration file could not be read due to access restrictions. Its contents were not inspected. Codex likewise warned it could not create PATH aliases because a home directory was unavailable to that process; its executable remains usable in this shell.

## Commands executed

The following non-mutating commands were run from the project directory. No command contacted a customer system, database, or external service.

| Purpose | Commands / APIs |
|---|---|
| Repository and file discovery | `git status --short`, `git branch --show-current`, `git --version`, `rg --files -g !node_modules -g !.git`, `git rev-parse --is-inside-work-tree`, `git log -1 --oneline` |
| OS and capacity | `Get-CimInstance` for OS, system, and video controller; `.NET Environment` architecture/version fallback; `cmd /c ver`; `Get-PSDrive -PSProvider FileSystem` |
| Toolchain | `python --version`, `py -0p`, `python -m pip --version`, `node --version`, `npm --version`, `docker --version`, `docker compose version`, `code --version`, `codex --version`, `npm list -g @openai/codex --depth=0` |
| Platform availability | `docker info --format ...`, `wsl --status`, PATH-scoped `Get-Command` checks for local-AI tools, conditional `nvidia-smi` |

Failed access-controlled commands were not retried with elevated permissions. No secret values, environment variables, credentials, or personal files were captured.

## Compatibility assessment

### Suitable baseline

- **FastAPI / Python:** Python 3.13 can be used for a modern API service, but the project should pin its interpreter and dependencies with a lockfile. For broad enterprise library compatibility, use a project-managed Python **3.12.x or 3.13.x** after confirming the selected ML, database, authentication, and test packages support it. Do not depend on the launcher’s ambiguous default selection.
- **React / TypeScript:** Node 26 and npm operate correctly, so the machine can develop React/TypeScript. For reproducibility and wider ecosystem support, the product should set an explicit Node version in `.nvmrc`/Volta or equivalent and CI. A supported LTS line (commonly Node 24 LTS, subject to confirmation when STEP 01 selects the toolchain) is safer than assuming Node 26 compatibility across all build tools.
- **PostgreSQL / pgvector:** Well suited as the planned primary relational and vector store. No PostgreSQL server, pgvector installation, or container runtime was checked or installed in this step.
- **Docker Compose:** CLI support is present, but Docker Engine must be made available before Compose-based local development, integration tests, or on-premise packaging can be validated.
- **Local embeddings / RAG / ML:** CPU inference is plausible but cannot be sized without CPU/RAM information. GPU acceleration is unverified. Plan for a CPU-capable baseline and optional, separately provisioned GPU inference profile.
- **Business intelligence, enterprise authentication, and automated testing:** The installed language toolchains are adequate for implementation. Compatibility of specific platforms or packages is an unverified design decision, not an established fact.

### Version risks and controls

| Risk | Impact | Recommended control |
|---|---|---|
| Python 3.13 with a launcher also showing 3.14 | Different developers/CI may install or run against different interpreters | Pin the project interpreter, enforce it in CI, and use locked dependencies; avoid 3.14 until the complete package set is verified |
| Node 26 is newer than the ecosystem baseline likely used by many enterprise tools | Build, native-module, or test-tool incompatibilities | Pin an approved active-LTS Node release in the repository and CI; verify before selecting frontend tooling |
| Docker Engine unavailable | Containers, local database, integration testing, and deployment parity cannot run | Treat engine availability as a precondition for container workflow validation; do not bypass policy or start it in STEP 00 |
| CPU/RAM/GPU visibility blocked | Cannot size local model, embeddings, or concurrent inference | Obtain authorized hardware inventory; retain a CPU-only/minimum deployment tier and an optional GPU tier |
| WSL status unavailable | Linux-oriented developer workflows cannot be assumed | Keep Windows-native instructions valid; validate WSL only with authorized access |

## Recommended MVP architecture

Adopt a **modular monolith with explicit domain boundaries**, a metadata/configuration layer, and independently deployable local AI workers. This minimizes early operational burden while preserving a credible migration path for enterprise scale.

```text
Browser (TR/EN)
   │ HTTPS
Enterprise Control Center (React + TypeScript)
   │
Application API (FastAPI modular monolith)
   ├─ Identity, RBAC/ABAC and tenant/company scope
   ├─ Organization, departments, metadata and configuration
   ├─ Forms, validation, workflows, approvals and audit
   ├─ Documents, reporting, regulatory-config versioning
   ├─ Data integration orchestration and business-process contracts
   └─ AI gateway / policy enforcement
   │
PostgreSQL + pgvector ─ object/document storage ─ job queue
   │
Optional isolated local inference service (LLM + embeddings)
```

### Key design choices

- **Company isolation:** Treat each customer deployment as a security boundary; model company/workspace context explicitly. For a multi-company deployment, apply tenant identifiers, database row-level security, scoped encryption keys, and tenant-aware audit trails. The on-premise default should make cross-customer sharing impossible by deployment topology.
- **Metadata-driven management:** Store organization units, department hierarchy, process definitions, form schemas, field rules, workflow states, approval matrices, regulatory obligation versions, report templates, and localization labels as versioned configuration. Do not encode a single EDAŞ/EPAŞ organization in source code.
- **Process interoperability:** Define versioned input/output data contracts, authorization policies, provenance metadata, and lifecycle states. Outputs become discoverable authorized inputs rather than direct table coupling.
- **Forms and workflows:** Use a restricted schema-driven form DSL (for example JSON Schema plus a controlled UI schema) and a BPM/workflow state-machine model. Validation must run server-side; client validation is only usability support. Start with auditable human approval flows rather than an overly general workflow engine.
- **Data integrations:** Use adapter/connector interfaces, staging/quarantine zones, schema validation, idempotency keys, lineage, and retry/dead-letter handling. Never put source credentials in workflow definitions.
- **Reporting:** Separate operational dashboards from governed regulatory reporting. Every regulatory report generation must record configuration/data lineage and require assigned human review and approval before any submission action.
- **AI:** Keep model inference, embedding generation, vector indexes, prompts, retrieved evidence, and AI audit records inside customer-controlled infrastructure. The AI gateway enforces company, role, document, and purpose scope; it is the sole route to local model services.

## Evolution strategy and trade-offs

| Approach | Assessment |
|---|---|
| Modular monolith (recommended MVP) | Strong fit for a configurable enterprise product: one deployable unit, transactional consistency, simpler on-premise operations, and lower support cost. Maintain strict module APIs, separate schemas/namespaces, and asynchronous events so it does not become a tightly coupled monolith. |
| Immediate microservices | Not recommended for MVP. It adds service discovery, distributed tracing, message delivery, deployment, upgrade, security, and air-gap support costs before product-domain boundaries are proven. |
| Selective service extraction (future) | Extract high-load or independently scaling concerns first: inference, ingestion/connectors, document conversion/OCR, reporting/analytics, and possibly workflow execution. Preserve contracts and event schemas from the MVP so extraction is evolutionary. |
| Local LLM runtime | Required capability, but make it optional by deployment profile. CPU-only models may be slow; GPU nodes improve throughput but require validated drivers, VRAM sizing, and vendor-compatible deployment. |

For offline/air-gapped customers, deliver signed, versioned installation bundles; an internal artifact registry/mirror; SBOMs; checksums; license notices; offline activation/update procedures; and compatibility matrices. The product must not require runtime access to a cloud model or telemetry endpoint.

## Security and privacy baseline

- Enforce enterprise authentication through an on-premise-compatible OIDC/SAML provider or customer identity integration; MFA and service-account controls are deployment policies.
- Combine RBAC with attribute/data-level authorization: tenant/company, EDAŞ/EPAŞ workspace, department, process, record classification, purpose, and delegation/approval state must constrain every API and AI retrieval operation.
- Encrypt data in transit (TLS) and at rest using customer-controlled keys where feasible. Separate encryption and signing keys, rotate them, and protect backups equivalently.
- Make audit logs append-oriented, time-synchronized, queryable, and access-controlled. Record identity, authorization result, changed fields, source, configuration/version, workflow actions, data lineage, and AI actions without logging secrets or unnecessary personal content.
- Default to restricted east-west and outbound network policies. Local AI/integration services receive only the minimum routes required; egress is deny-by-default in air-gapped profiles.
- Store application and connector secrets in an approved on-premise secrets manager or OS/platform secret store, never in source, form metadata, logs, or container images.
- Design backup/restore, point-in-time recovery, immutable/offline copies, restore drills, retention, and destruction procedures before production use.
- AI access must inherit source-document and row-level permissions at retrieval time. Use a read-only, allow-listed semantic/data access layer for AI; prohibit arbitrary model-generated SQL against operational databases.
- Treat prompts, uploaded documents, retrieved content, and tool outputs as untrusted. Apply prompt-injection-resistant system policies, tool allow-lists, content classification, context boundaries, output validation, human approval for consequential actions, and full AI provenance.
- Maintain signed releases, SBOMs, dependency/container scanning, provenance/attestation where available, private/offline package mirrors, vulnerability response processes, and offline update verification.

## Licensing and cost considerations

- Select permissively licensed or commercially licensable frameworks, database extensions, workflow components, and AI models only after legal review. Record model weights, tokenizer, embedding model, dataset/use restrictions, redistribution rights, and attribution requirements per release.
- Local inference avoids mandatory per-request API charges and data egress, but shifts cost to hardware, power, GPU/CPU capacity, storage, backups, patching, observability, and support.
- A modular monolith keeps initial infrastructure and customer installation costs lower. Microservices and GPU inference should be introduced only when demonstrated throughput, isolation, or independent lifecycle needs justify their operational cost.
- Regulatory content, submission rules, and official codes must remain customer-reviewed, configurable/versioned reference content sourced from authoritative institutions; this assessment makes no claim about specific obligations or deadlines.

## Internationalization and localization (mandatory architecture)

VOLTORA AI must be internationalized from its first implementation. **Turkish (`tr-TR`) is the default product locale; English (`en-GB`) is mandatory.** The architecture must accept additional BCP 47 locale tags without a structural redesign. Unicode UTF-8 is required for all API payloads, database connections, stored translation resources, exports, document templates, search, and audit records.

### Product requirements and implementation strategy

- Provide a global language selector in the application header. A locale change must take effect at runtime without reauthentication, retain the active route and authorization scope, and update user-facing labels, formats, notifications, and newly rendered AI interactions.
- Store a user’s explicit locale preference (`tr-TR` default) in the identity/profile domain, scoped by user and, where required, customer deployment. A safe order of resolution is: explicit user preference, customer default, then `tr-TR`; browser preference must never override an explicit preference.
- Keep local, versioned translation resources in the customer-controlled deployment. No external translation service is a runtime dependency. Customer content, prompts, documents, or metadata must never be submitted to one without explicit, auditable customer authorization.
- Prohibit user-facing literals in application components, email/notification templates, workflow definitions, report templates, or server validation paths. Code and configuration reference stable message keys; locale resources hold display text. Product policy needs enforcement through linting/review and completeness tests.
- Use locale-aware formatting rather than translated strings for dates, times, quantities, percentages, and currencies. Persist timestamps in UTC plus an explicit IANA time zone where a business event needs one; apply locale and time-zone formatting at the presentation/export boundary. Persist monetary values as decimal amounts plus an ISO 4217 currency code, never a preformatted currency string.
- Model departments, sub-departments, process definitions, field labels/help text, enumerations, validation messages, workflow states/messages, dashboards, report/document templates, notifications, and regulatory reference labels as localized content. Stable identifiers, machine-readable codes, and data contracts must not change when a language changes.
- Regulatory submissions require a separately versioned, authoritative-language template and format designation. A display-language selection must not silently translate legally mandated content, official codes, or required submission formats. Human review remains mandatory.
- For dynamic forms and reports, store translatable display fields independently from the immutable schema/field/report identifiers. Validate all locale catalogs and template placeholders before publication.

### Mandatory-requirement traceability

| Mandatory requirement | Assessment finding / architectural control |
|---|---|
| Turkish (`tr-TR`) and English (`en-GB`); Turkish default | Supported as first-class release locales. Locale resolution defaults to `tr-TR` unless an explicit user or customer setting applies. |
| Header selector; runtime switching; persistent preference | A global React header control updates the active i18n context without re-login; the profile service persists the selected BCP 47 locale and applies it at the next session. |
| Local resources; no mandatory external translation API; no unauthorized data egress | Ship product catalogs locally and store customer metadata translations in PostgreSQL. Outbound translation is disabled by default; any exception requires explicit customer authorization, audit evidence, and policy/legal approval. |
| No hardcoded user-facing text; UTF-8 / Unicode | Require keyed catalogs for all user-visible surfaces and enforce UTF-8 from source through database, APIs, exports, and search. Add static checks and code review controls for component/server/template literals. |
| Locale-aware date, time, number, percentage, and currency | Use browser `Intl` and Python Babel at presentation/export boundaries; retain UTC, IANA time zone, decimal values, and ISO currency codes as canonical data. |
| Multilingual departments, processes, dynamic forms, validation | Use stable IDs plus versioned localized metadata records. Form schemas and validation codes remain language-neutral; labels, help, enum labels, and validation display messages are localized. |
| Multilingual dashboards, reports, exports, notifications, workflows | Treat each as a catalog- or template-backed surface. Locale is carried in job/notification/export requests so asynchronous results use the intended language. |
| Multilingual local AI assistant | Local multilingual LLM/embedding selection, localized system/policy prompts, permission-aware retrieval, and per-language quality testing are required before production enablement. |
| Extensible additional languages | Use BCP 47 locale tags, namespaced keys, plural-aware catalogs, and translation tables keyed by locale rather than language-specific columns or branching. |
| Fallback, missing-key detection, completeness tests | Define an explicit fallback chain, observable missing-key events, and release-blocking automated catalog/placeholder/hardcoded-text tests for both mandated locales. |
| Regulatory language/formats; stable IDs separated from labels | Regulatory template language and prescribed formats are separately versioned and protected from automatic translation. Stable business IDs, codes, and contracts never derive from display text. |

### Recommended component model

| Concern | Recommendation | Offline / commercial assessment |
|---|---|---|
| React user interface | Adopt **i18next** as the translation engine with **react-i18next** bindings, subject to dependency approval in a later step. They provide namespaces, fallback, interpolation, plural handling, and React hooks/components; translation files can remain in the repository/deployment. | Suitable for local/offline operation. Both upstream repositories state the MIT license; retain notices and verify the pinned release’s license in the SBOM and legal review. [i18next docs](https://www.i18next.com/overview/getting-started), [react integration](https://www.i18next.com/react), [i18next license](https://github.com/i18next/i18next/blob/master/LICENSE), [react-i18next license](https://github.com/i18next/react-i18next/blob/master/LICENSE) |
| Browser-side formatting | Use the standards-based `Intl` APIs (`Intl.DateTimeFormat`, `NumberFormat`, `PluralRules`, and `RelativeTimeFormat`) keyed to the active locale. | No external API or customer-data transmission. Browser/runtime locale data availability should be covered by browser support tests. [Intl reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) |
| Python / FastAPI formatting and catalogs | Evaluate **Babel** for server-side locale-aware dates, numbers, currencies, plural rules, and message-catalog tooling. Keep server messages keyed and translated locally; API error contracts should carry stable codes with localized display text selected for the caller’s locale. | Compatible with offline operation. Babel is BSD-style licensed according to its repository; confirm the selected version and third-party notices during legal/SBOM review. [Babel documentation](https://babel.pocoo.org/en/latest/), [Babel license](https://github.com/python-babel/babel/blob/master/LICENSE) |
| Metadata and tenant/customer content | Use PostgreSQL normalized translation tables for governed, queryable localized metadata: `(entity_type, entity_id, field_key, locale, text, version, status, source)`, unique on the stable entity/field/locale key. Use `jsonb` only for bounded, schema-validated locale maps where atomic retrieval is materially simpler. | Fully local. `jsonb` is indexable but should not replace relational lifecycle, review, and completeness controls. [PostgreSQL JSON types](https://www.postgresql.org/docs/current/datatype-json.html) |
| AI assistant | Require local multilingual embedding and generation models with verified Turkish/English performance, a documented commercial license, and separate evaluation per language. System prompts, retrieval filters, policy messages, citations, and human-approval controls must be localized and permission-aware. | Offline-capable by design; model license, hardware profile, benchmark results, and supported languages must be verified when a model is selected. |

### Translation lifecycle, fallback, and quality controls

1. Define immutable, namespaced message keys by domain (for example, `controlCenter.kpi.activeIntegrations`) and a catalog version per deployable release. Never derive stable keys from translated labels.
2. Maintain local `tr-TR` and `en-GB` catalogs for product UI and approved templates. Keep customer-configured translations in the governed metadata store with author/reviewer/publish state, audit history, and effective version.
3. Resolve a missing locale key using the configured fallback chain—`en-GB` to `tr-TR` for English, `tr-TR` for Turkish and other initial fallback cases—while emitting a non-sensitive, observable missing-key event. In production, show a controlled fallback label, never a raw internal key or stack trace.
4. Fail CI/release validation for missing mandatory keys, invalid ICU/interpolation placeholders, inconsistent plural forms, duplicate stable identifiers, unapproved metadata translations, untranslated required report/export templates, or hardcoded user-facing strings. Test the locale switcher and both locales in component, end-to-end, API, document-export, and notification workflows.
5. Add accessibility and layout checks for expansion, Turkish character handling, line wrapping, sorting/search collation expectations, and export rendering. Do not assume English string length or ASCII-only inputs.

### Internationalization risks

| Risk | Mitigation |
|---|---|
| Translation keys leak into end-user screens | Controlled fallback text; missing-key telemetry; release-blocking completeness tests |
| Locale change creates inconsistent authorization or workflow state | Locale controls presentation only; authorizations, identifiers, state transitions, and data contracts remain locale-invariant |
| Dynamic customer metadata is translated without governance | Versioned translation workflow with reviewer approval, audit trail, and role/data scope |
| AI produces poor or unsafe Turkish/English answers | Per-language evaluation suites, source-grounded retrieval, local model selection review, human approval for consequential output, and AI audit records |
| Regulatory text is accidentally translated | Explicit regulatory-language/template metadata and export validation; preserve prescribed official content and format |
| External translation dependency or data egress | Local catalogs by default; network egress denial; any exception requires explicit authorization, contract/legal review, and audit evidence |

This internationalization strategy is compatible with the recommended modular-monolith MVP and local inference boundary. It adds modest catalog/metadata governance work early, but avoids retrofitting every department, workflow, report, and AI surface after the product has accumulated hardcoded language assumptions.

## Risks, blockers, and next action

**Current blockers / unknowns:** Docker Engine is not available; WSL is access-denied; CPU/RAM/GPU/VRAM cannot be independently verified under the current access policy; no local inference runtime is found on PATH. These are not product blockers for planning, but they block performance sizing and container execution validation.

**Recommended next action:** With explicit approval for STEP 01, define the product repository governance and reproducibility baseline: approved project runtime versions, module boundaries, threat-model backlog, configuration/data-contract conventions, initial deployment profiles, and a hardware sizing verification plan. Do not install or scaffold application components until that next scope is approved.

## STEP 01 gate

**Decision: GO (conditional).** Proceed only with explicit approval and preserve the on-premise, offline-capable, locally inferred architecture. Resolve Docker Engine availability and obtain authorized hardware inventory before committing to local LLM model sizes, concurrency targets, or GPU deployment requirements.

---

## STEP 00.1 verification update — 2026-11-26

**Status:** completed assessment only. This update preserves the initial findings above and records the later user-verified environment results and additional read-only checks. Where the constrained assessment shell could not access the same Windows/Docker context, the evidence is labelled rather than reconciled by assumption. The full evidence, recommendations, and risk register are in [STEP 00.1](step_00_1.md).

### Verified update summary

| Area | Updated finding | Evidence |
|---|---|---|
| Hardware | Intel Core i7-1255U (10 physical / 12 logical cores), 15.70 GiB RAM, Intel Iris Xe, driver `31.0.101.4502` | User-verified results; constrained CIM queries remained access-denied |
| Docker | Desktop Linux context, Engine `29.1.3`, 12 CPUs and `8,167,559,168` bytes (about 7.607 GiB) memory available; successful `docker ps`/formatted `docker info` verify connectivity at that time | User-verified results; this shell received named-pipe permission denial and must not negate the successful session |
| Existing containers | Five named containers observed, with 213.51 MiB aggregate point-in-time container memory and 1.81% CPU | User-verified results; no container data was accessed or modified |
| Ports | Proposed `5173`, `8001`, `5434`, `11434`, and `6380` were not listening during the local check. Existing published ports include Portainer on `8000`/`9443`, which listens on all interfaces | Read-only `netstat`; user-verified allocation list |
| Python | `python` is 3.13.11; `pip` 26.2.1 is paired with it; `py -0p` from this shell exposed 3.14; `uv` 0.10.8 exists on PATH | Direct checks; full launcher inventory is user-reported |
| Runtime direction | Pin Python 3.13 for the initial product baseline, prefer Node 24 LTS for reproducibility, and use `uv` with an offline-capable locked workflow subject to later approval | Assessment; no installation/change made |
| Local AI | CPU-first, optional and failure-isolated AI is viable for small quantized models, embeddings, and RAG experimentation; 7B–8B is a constrained/single-user profile, not a throughput commitment | Assessment only; no model download or benchmark |

### Revised STEP 01 gate

**GO (conditional) for STEP 01 planning only.** The verified developer baseline is sufficient to define pinned runtimes, architecture, deployment profiles, and governance. Before containerized integration validation or a local-AI performance commitment, re-check Docker access from the intended developer account and validate model/hardware sizing with authorized measurements. STEP 01 still requires explicit approval.
