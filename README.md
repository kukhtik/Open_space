<div align="center">

# Open Space

**Maxim Kukhto · [@kukhtik](https://github.com/kukhtik)**

Agentic systems · Offline RAG · Tool-verification loops · Numerical engineering

*Screenshots from working builds; every number from a measured run. Proprietary internals (medical documentation, licensing logic, numerical routines) stay private — interfaces, architectures and metrics are documented here.*

🇷🇺 [Русская версия](README.ru.md)

</div>

---

## Swarm — multi-agent orchestration with machine acceptance

**Problem.** LLM agents report “done” when the work is not done — they fail silently, leave stubs, loop. A single agent grading itself suffers confirmation bias.

**Approach.** Role isolation: the planner decomposes a task into verifiable steps, the executor does the work, and a verifier opens with a fresh context to judge execution — pytest, linter, dry-run — never the agent’s word.

![Swarm — architecture](screenshots/swarm-architecture.png)

| Mechanism | How it works |
|---|---|
| **Role isolation** | Planner / executor / verifier — separate personas and context windows |
| **Closed tool loop** | Every tool output is checked deterministically before the next step |
| **Boundary protection** | `scope.protected` files are SHA-256 pinned; editing one auto-fails the task — both edits and deletions are caught (verified mechanically) |
| **Budget & escalation** | Bounded turns and attempts; after N failures the task escalates to a stronger model or a human, per the task package |

Reference run (`demo-calc-bugfix`): oracle `python test_calc.py` → `ALL TESTS PASSED`, 11/11 checks green, 4 turns, 21 s wall, tamper false.

![Swarm — live kanban board](screenshots/swarm-kanban.png)

**Open code:** [hermes-reform](https://github.com/kukhtik/hermes-reform) — 21 merged reform branches (core stability, security baseline, desktop stability), CI + independent model review. Includes the runbook for the companion multi-profile gateway: one Telegram bot serving 13 agent profiles via DM topics (thread → profile routing).

**Stack:** Python · Hermes Agent (kanban dispatcher, middleware) · pytest as the oracle · SHA-256 tamper checks · JSONL trajectories.

---

## Engineer Companion — offline RAG over medical-equipment documentation

**Problem.** A field engineer servicing a clinical linear accelerator (Varian TrueBeam / VitalBeam) works in an RF-shielded bunker with zero internet, and needs the exact page of a specification in seconds — out of tens of thousands of pages.

**Approach.** Fully local hybrid retrieval with verifiable evidence links: document, page, drawing region. Runs on CPU.

![Engineer Companion — chat with evidence](screenshots/engineer-companion-chat.png)

| Metric | Value |
|---|---|
| Fact-catalog quality | **96 / 100** on the 100-case `grounded-qa-100-v1` benchmark |
| Latency | **p50 130 ms** · p95 1 193 ms |
| Scale | **21 214 facts** over a 71 862-row vector + BM25 index |
| Tests | **768 / 775** passing |

<table>
<tr>
<td width="50%"><img src="screenshots/engineer-companion-cloud.png" alt="Fact cloud"><br><sub>Fact cloud — the Clinac catalog as a node graph</sub></td>
<td width="50%"><img src="screenshots/engineer-companion-library.png" alt="Library"><br><sub>Document library & index statistics</sub></td>
</tr>
</table>

**Stack:** Python · LanceDB (vector) + BM25 (hybrid) · multilingual-e5-small embeddings + mmarco cross-encoder reranker · Gemma-3-4B (GGUF, llama.cpp) · PySide6 desktop shell; Android shell via Compose + ONNX Runtime Mobile.

---

## GeoConverter — numerical geodetic calibration

**Problem.** Converting coordinates between legacy local coordinate systems and modern global datums (WGS-84) requires sub-millimeter precision under strong non-linear distortions. (Exact local systems and zones in use are client-confidential.)

**Approach.** A four-stage numerical pipeline with a round-trip check: the closed route must come back within millimetres.

![GeoConverter — main window](screenshots/geoconverter.png)

Reference points → **Helmert 3D (SVD)** → **Levenberg–Marquardt** → **Nelder–Mead** (tolerance 1e-12) → round-trip error validation.

![GeoConverter — generated report](screenshots/geoconverter-report.png)

**Shipped as a commercial desktop product** (Windows 7/10): hardware-locked activation keys with no external server, a built-in license issuer, PyQt6 UI, and automated parsing of mixed data from AutoCAD, CSV and Excel.

**Stack:** Python · NumPy / SciPy (least_squares, SVD) · PyQt6 · PyInstaller / Nuitka.

---

## VETKA_DWG — survey sketches → CAD drawings

**Problem.** A surveyor draws a field abris (a hand sketch with numbered points) and then spends hours transferring it into CAD.

**Approach.** Photo of the sketch → marker and text recognition → topographic DXF/DWG. **4 hours of manual drafting → 15 minutes.** In production use with surveying companies (Belarus).

![VETKA_DWG — block library audit](screenshots/vetka-dwg-library.png)

The screenshot shows the block-library audit console: **444 / 444 blocks pass all five fidelity axes** against the reference template (GUGK sign library, ШАБЛОН‑2025).

**Stack:** Python · custom CV pipeline (marker detection, OCR) · ezdxf · 444-block GUGK sign library · PySide6 / QML · CI with quality gates.

*Source is private (commercial use); architecture and audit results are documented here.*

---

## PenaPivo / BeerLog — serverless Telegram bot + Mini App

**What it is.** A live Telegram product: a beer-tasting journal with write-ups, statistics, shareable profile cards and ratings.

<table>
<tr>
<td width="24%"><img src="screenshots/penabot-miniapp.png" width="220" alt="Mini App"><br><sub>Mini App — tasting journal (demo data)</sub></td>
<td><img src="screenshots/penabot-card.png" alt="Profile card"><br><sub>Profile card produced by the bot's card module (SVG; demo data)</sub></td>
</tr>
</table>

**Architecture:** Cloudflare Workers (edge) · D1 (SQL) · R2 (backups) · Telegram Mini App with i18n (ru/en/be) · CI: coverage gate **≥84% branches** + mutation testing (Stryker).

**Live bot:** [@PenaPivo_bot](https://t.me/PenaPivo_bot)

**Stack:** JavaScript (ESM) · Cloudflare Workers / D1 / R2 · Node.js · resvg (card rasterization).

---

## VTuber Rigger — AI auto-rigging: model or screenshots → VRM 1.0

**Problem.** Rigging a VTuber avatar — skeleton, skin weights, expressions, spring bones — takes hours of manual work in Blender, and screenshot-based reconstruction from a flat image is harder still.

**Approach.** Two paths, fully local (no paid APIs): **Path A** — a 3D model (GLB/GLTF/VRM/OBJ) → VRM 1.0; **Path B** — front/side/back screenshots → VRM 1.0. K-Means + curvature body segmentation, a 53-bone VRM skeleton predictor, RBF + Laplacian skin weights, 17 expressions, springs for hair/clothing, plus headless-VRM and LPIPS/CLIP perceptual QA gates.

![VTuber Rigger — pipeline run](screenshots/rigger-pipeline.png)

Reference run on a test mesh: `GLB → VRM`, exit 0, **54 nodes, 1 mesh, extensions `VRMC_vrm` + `VRMC_springBone`**.

**Stack:** Python · trimesh · pygltflib · numpy / scipy / scikit-learn · OpenCV · pytest (unit + integration + property + perceptual suites). Public: [github.com/kukhtik/RIGGER](https://github.com/kukhtik/RIGGER).

---

## Neuro Property Trade — deterministic board-game engine + AI seats

**Problem.** A commercial board-game title can't be safely modded for AI play (anti-tamper, delisted versions, no clean state reads). To let an AI seat play fairly, the game itself must be built so the engine — not the AI — owns the state.

**Approach.** A standalone Godot 4 game with a deterministic rules engine: seeded RNG flows through one class, every action goes through an append-only event log (full replay), and AI seats (Neuro / Evil Neuro / human) submit **intents** that the engine validates and executes — the AI can never corrupt state. Original theme, no trademarked names.

![Neuro Property Trade — engine tests and board demo](screenshots/neuro-engine.png)

Live run: **161 headless tests green**, then an ASCII board snapshot (positions, cash, tiles per seat).

**Stack:** Godot 4.7 (GDScript) · headless-testable core · Neuro SDK adapter (WS action protocol, action registry, per-turn context). Public: [github.com/kukhtik/neuro-property-trade](https://github.com/kukhtik/neuro-property-trade).

---

## Stack

| Area | Technologies |
|---|---|
| **Languages** | Python (primary), JavaScript / TypeScript, GDScript, SQL, Java / Kotlin, Rust (basic) |
| **AI / LLM** | llama.cpp / GGUF, ONNX Runtime, LanceDB, embeddings + cross-encoder reranking, agent harnesses (Hermes Agent, Claude Code, Codex) |
| **3D / Game** | Godot 4 (headless-testable deterministic engines), VRM 1.0 / glTF, trimesh, Blender toolchain |
| **Backend** | FastAPI, Cloudflare Workers, D1, R2, SQLite / PostgreSQL, Kafka / gRPC |
| **Interfaces** | PySide6 / PyQt6 / QML, Telegram Mini Apps, Compose (Android) |
| **Numerical** | NumPy / SciPy, Helmert / SVD, Levenberg–Marquardt, Nelder–Mead |
| **Infrastructure** | Docker, Linux / WSL, Git, GitHub Actions / GitLab CI, Cloudflare, systemd |
| **Quality** | pytest, coverage gates, mutation testing, SHA-256 tamper checks, JSONL trajectories |

---

<div align="center">

**[github.com/kukhtik](https://github.com/kukhtik)** · [linkedin.com/in/max-kukhto-294682165](https://www.linkedin.com/in/max-kukhto-294682165/) · maxkukhto@gmail.com · Mogilev, Belarus (remote)

<sub>Interface screenshots are captured from working builds; client-identifying data (site names, local coordinate systems, zone numbers, coordinate catalogs) is masked. The EC chat and BeerLog shots show demo content. Every metric is from a measured run.</sub>

</div>
