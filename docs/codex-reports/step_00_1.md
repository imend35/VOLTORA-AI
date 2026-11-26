# VOLTORA AI — STEP 00.1: Environment Verification & Architecture Report

**Date:** 2026-11-26  
**Status:** assessment complete; no implementation authorized  
**Scope:** read-only environment verification and preliminary architecture/runtime assessment for an on-premise-first electricity distribution and retail-electricity product. Turkish (`tr-TR`) and English (`en-GB`) remain mandatory. No runtime, dependency, model, image, service, container, OS setting, cloud resource, application code, commit, or external customer system was changed.

## Decision

**GO (conditional) for STEP 01 planning; no GO for a production local-AI sizing or container-execution claim yet.**

The verified workstation and Docker Desktop allocation can support repository governance, a modular on-premise application design, CPU-first AI development, and a reproducible future development stack. The 7.607 GiB Docker limit and lack of verified CUDA make GPU-dependent or high-concurrency inference unsuitable assumptions. Core product workflows must remain fully usable when the AI worker is unavailable.

## Evidence and inspected configuration

| Area | Finding | Evidence / confidence |
|---|---|---|
| CPU | 12th Gen Intel Core i7-1255U; 10 physical cores, 12 logical processors | User-verified |
| RAM | 15.70 GiB physical memory | User-verified |
| GPU | Intel Iris Xe Graphics, driver `31.0.101.4502`; no NVIDIA CUDA established | User-verified; `nvidia-smi` was also absent from this shell's PATH |
| Docker | Client/server `29.1.3`; Compose `v5.0.1`; context `desktop-linux`; Docker Desktop OS; 12 logical CPUs; limit `8,167,559,168` bytes (about 7.607 GiB) | User-verified from successful commands |
| Docker connectivity | An initial connection failed, then `docker ps` and formatted `docker info` succeeded | Engine connectivity verified at the successful point in time; initial failure cause remains unknown |
| WSL | WSL 2 default; `OracleLinux_8_10` and `docker-desktop` running; Ubuntu stopped | User-verified |
| Python | Launcher inventory includes 3.14 default, 3.13t, 3.13, 3.11, and Astral CPython 3.11.15; PATH Python `3.13.11`, pip `26.2.1` paired with 3.13 | User-verified; PATH version/pip independently confirmed |
| Node | Node `26.8.1`, npm `11.12.1`; `nvm`, `fnm`, and `volta` unresolved by `Get-Command` | Directly checked; unresolved commands do not prove absence elsewhere |
| uv | `uv 0.10.8` on PATH | Directly checked; its installed-runtime listing could not read its cache due to access restriction |
| Existing AI runtimes | No PATH-resolvable `ollama`, `llama-server`, `llama-cli`, `vllm`, or `local-ai`; no NVIDIA CLI | Directly checked; does not rule out non-PATH installs |

The constrained assessment shell received `Access denied` for CIM hardware queries, WSL enumeration, and the Docker named pipe; it therefore could not repeat the successful user-session Docker checks. It also could not read Docker or uv user cache/configuration. These are verification-context restrictions, **not** a basis to invalidate user-verified Docker availability. No credentials, Docker configuration, container filesystem, logs, volumes, databases, or application data were inspected.

## Commands executed and results

| Purpose | Read-only command / result |
|---|---|
| Hardware | `Get-CimInstance Win32_Processor`, `Win32_ComputerSystem`, and `Win32_VideoController` → access denied in this shell |
| Python/tooling | `py -0p`, `python --version`, `python -m pip --version`, `uv --version`, PATH command discovery → Python 3.13.11/pip 26.2.1; uv 0.10.8; no PATH AI runtime |
| Node | `node --version`, `npm --version`, `Get-Command nvm,fnm,volta` → 26.8.1/11.12.1; no command resolution |
| Docker/WSL | `docker version`, `docker compose version`, `docker context show`, `docker info`, `docker ps`, `docker stats`, `wsl --status`, `wsl -l -v` → CLI/Compose available; constrained shell was denied daemon/WSL access |
| Ports | `netstat -ano` filtered to relevant ports → candidates `5173`, `8001`, `5434`, `11434`, `6380` absent; existing listeners detailed below |
| GPU | `nvidia-smi` → command not recognized |

## Docker, resource, and port assessment

The user-reported existing containers are retained as a point-in-time, non-invasive resource snapshot:

| Container | Memory | CPU |
|---|---:|---:|
| `ai-bap-rubric-test-frontend-1` | 12.32 MiB | 1.68% |
| `ai-bap-rubric-test-api-1` | 116.60 MiB | 0.13% |
| `ai-bap-rubric-test-db-1` | 27.86 MiB | 0.00% |
| `ai-bap-db-1` | 37.91 MiB | 0.00% |
| `portainer` | 18.82 MiB | 0.00% |
| **Total** | **213.51 MiB** | **1.81%** |

These values exclude Docker VM, OS, filesystem cache, and future workload overhead. They are not capacity measurements. No container was stopped, restarted, removed, entered, queried, or modified.

The read-only port check confirmed `8000` and `9443` listening on all IPv4 interfaces (and IPv6), and `5433`, `18000`, `18080`, and `55434` bound to loopback. This aligns with the user-reported Portainer, AI BAP, and PostgreSQL allocation. The proposed VOLTORA ports were not listening at the time checked:

| Candidate | Status | Recommendation |
|---:|---|---|
| 5173 (frontend) | Available at check time | Development-only host mapping; never expose in production |
| 8001 (API) | Available | Bind localhost during development; use reverse proxy/TLS in deployment |
| 5434 (PostgreSQL) | Available | Do not publish except controlled local administration; use internal Compose network by default |
| 11434 (AI runtime) | Available | Bind only loopback/internal network; require application gateway and auth controls |
| 6380 (Redis, if needed) | Available | Do not publish by default; internal network only |

Portainer on `0.0.0.0:8000` and `[::]:8000` plus `9443` is reachable from network paths permitted by the host firewall. This increases the attack surface for a privileged container-management plane. Do not change it in this step; later operations should confirm firewall reachability, require HTTPS/strong administrator authentication, restrict source networks, patch it, and avoid public exposure.

## Runtime compatibility recommendation

| Candidate | Compatibility and maintenance | Reproducibility / deployment recommendation |
|---|---|---|
| Python 3.11.15 | Broad ML/library compatibility, Windows/Linux and Docker friendly; upstream security support ends October 2027 | Acceptable compatibility fallback, but shorter remaining life makes it less desirable for a new multi-year baseline |
| Python 3.13.11 | Current PATH runtime; modern runtime, security-supported to October 2029; package support for selected ML wheels must be checked before lock | **Recommended initial API/tooling baseline**: pin exact patch-compatible range and lock dependencies; run CI on Linux and Windows |
| Python 3.13t | Free-threaded variant has ecosystem/native-extension risk | Do not select as product baseline until the complete dependency set is explicitly validated |
| Python 3.14 | Launcher default, but adoption is not established for the planned stack | Do not rely on launcher default; exclude from initial lock/CI matrix until later compatibility review |
| Node 24 LTS | Production-oriented LTS line; good choice for Vite/React ecosystem after lockfile verification | **Preferred frontend baseline** for long-lived repeatable builds; pin in repository and CI |
| Node 26.8.1 | Installed and, as of this report, an LTS line; newer than the requested stable baseline and may reveal tooling/native addon gaps | Do not make installed version an implicit project requirement; retain only as a separately tested developer runtime |
| Docker/Compose | Compatible with Windows Desktop and Linux deployment patterns once the intended account can reach Engine | Use Compose for local/on-prem profiles, internal networks, non-root images, explicit resource limits, and pinned digests |

Python's currently supported-version schedule lists 3.13 through October 2029 and 3.11 through October 2027. Node's release policy says production applications should use Active or Maintenance LTS; its official schedule lists both 24 and 26 as LTS at this report date. [Python lifecycle](https://devguide.python.org/versions/) and [Node releases](https://nodejs.org/en/about/previous-releases).

**uv assessment:** adopt `uv` for a later approved implementation if its use is compatible with the organization's supply-chain policy. It supplies project/environment management, universal locking, Python-version management, pip-compatible workflows, and documented Docker/FastAPI integrations. Use a committed lockfile, pinned index/mirror, hash/attestation policy where supported, offline wheelhouse or internal repository for air-gapped customers, and explicit interpreter selection. Its presence does not authorize installation or cache repair. [uv documentation](https://docs.astral.sh/uv/).

Unverified compatibility assumptions remain: exact FastAPI/Pydantic/SQLAlchemy/Alembic, React/Vite/Tailwind/i18next/ECharts, PostgreSQL/pgvector, scikit-learn/XGBoost, and chosen inference bindings/model wheels must be validated against the pinned Python/Node/OS/architecture set before final selection in STEP 03. No dependency was installed or compatibility-tested here.

## Local AI feasibility

| Workload | Feasibility on this device | Operating recommendation |
|---|---|---|
| Ollama / llama.cpp | Feasible as optional native Windows CPU-first development runtimes; neither is currently discovered on PATH | Prefer native Windows for developer ergonomics and to avoid consuming the 7.607 GiB Docker allowance; validate later, do not install now |
| Quantized 1B–3B LLM | Plausible for single-user, low-throughput local tasks | Use a bounded worker queue, timeouts, cancellation, and a small context window; benchmark only after approval |
| Quantized 7B–8B LLM | Possible only as a constrained experiment; RAM contention, response latency, and Docker overhead are material | Do not promise interactive multi-user throughput; evaluate native CPU profile before a containerized profile |
| Multilingual embeddings | Strongest initial local-AI fit: compact multilingual encoder plus pgvector/approved vector store | Batch/index asynchronously, retain document permissions and provenance, and test Turkish/English retrieval separately |
| Local RAG | Feasible if documents, chunks, embeddings, ACLs, citations, and source versions remain local | Keep retrieval permission-aware; do not place source data or embeddings in external APIs |
| Independent workers | Recommended | Worker failure/restart must not block API, reporting, entry, dashboards, or workflows |

No AI model was downloaded and no inference benchmark was run. All throughput/latency claims are deliberately absent. Intel Iris Xe does not establish a supported acceleration route for the intended toolchain; CUDA is unverified and `nvidia-smi` is unavailable. Treat CPU execution as the guaranteed baseline, with an optional separately qualified GPU deployment profile for customers needing throughput.

Native inference avoids Docker's 7.607 GiB cap but competes with Windows/application RAM; containerized inference improves isolation and packaging but shares the constrained Docker budget. For this workstation, native Windows is the better exploratory profile; an on-premise customer deployment may use Linux containers or a dedicated GPU node after sizing. In every profile, AI is advisory: it cannot control SCADA, autonomously switch the grid, submit regulatory reports, or perform other consequential actions without policy-gated human review.

## Preliminary stack and operational controls

The proposed React/TypeScript/Vite/Tailwind/i18next/react-i18next/ECharts frontend; FastAPI/Pydantic/SQLAlchemy/Alembic backend; PostgreSQL/pgvector/local storage; and local inference/RAG/scikit-learn/XGBoost workers remain reasonable **candidates**, not approved installed technologies. The modular-monolith-with-independent-workers architecture from STEP 00 remains recommended: it preserves simpler on-premise operation while isolating ingestion, reporting, and AI workloads for later extraction.

- Deploy customer-controlled storage, local catalogs, UTF-8, `tr-TR` default and `en-GB` in every user-facing/API/export surface. Stable keys and regulatory identifiers must remain language-neutral.
- Use Docker internal networks by default; publish only a TLS reverse-proxy boundary. No database, Redis, embedding, or inference host port is a production default.
- Preserve role/tenant/data-level authorization at API and retrieval time. AI must use read-only, allow-listed data tools, permission-filtered context, prompt-injection defenses, output validation, audit provenance, and human approval gates.
- Keep core data entry, workflows, dashboards, reporting, exports, and approvals operational if the local AI runtime is stopped, slow, unavailable, or rejected by policy.
- Use synthetic data only for development. Never send customer data to external AI, and do not access employer systems, secrets, or container application data.

Licensing must be reviewed per exact dependency, container base image, model weights, tokenizer, embedding model, and redistribution method. Maintain notices, SBOMs, model provenance, signed/offline update bundles, vulnerability patch process, and customer-approved retention/backup controls. Local AI removes per-request cloud charges and customer-data egress but transfers cost to CPU/GPU hardware, power, storage, backups, patching, observability, and support. The existing laptop is a development profile, not a production capacity quotation.

## Risks, constraints, and blockers

| Risk / uncertainty | Impact | Required control or next verification |
|---|---|---|
| Docker availability differs by account/context | Local Compose validation can fail despite a prior successful session | Re-run non-mutating Docker health check from the intended developer account before any later container work; do not diagnose initial failure by assumption |
| 7.607 GiB Docker allocation | Database + API + frontend + AI can contend for memory | Keep AI optional/outside Docker during early development; set service limits; size production separately |
| No verified CUDA | No GPU performance/design claim | Maintain CPU baseline; validate NVIDIA hardware, drivers, CUDA/runtime, VRAM, and model support only if a GPU profile is selected |
| Integrated GPU | Vendor/runtime support may be uneven | Do not make Iris Xe an acceleration dependency |
| Python launcher ambiguity / 3.14 default | Non-reproducible developer execution | Pin Python 3.13 explicitly in project/CI and record executable path policy |
| Dependencies/models untested | Build, security, license, and language-quality exposure | Validate before STEP 03 selection; retain lockfiles, SBOM, test matrix, license register, TR/EN evaluation |
| Portainer externally listening | Management-plane exposure | Later restrict network access and administration; no configuration change made here |
| Existing containers and ports | Collision/data-loss risk | Preserve them; allocate proposed ports only after future checks; no inspection or modification |
| Offline/on-premise supply chain | Updates/dependencies may be unavailable or unverifiable | Design internal mirror/wheelhouse, signed artifacts, checksums, SBOMs, and offline update procedure |

## Recommended next action

Upon explicit approval for STEP 01 only, define the reproducibility and governance baseline: pinned Python 3.13 and Node 24 policy, lockfile/SBOM and offline-mirror expectations, module boundaries, threat-model backlog, data/metadata contracts, deployment profiles, port-exposure policy, and an authorized performance-validation plan. Do not scaffold or install services until that scope is separately approved.

