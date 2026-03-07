<div align="center">

<img src="https://img.shields.io/badge/Chainlink-Hackathon_2026-375BD2?style=for-the-badge&logo=chainlink&logoColor=white" />
<img src="https://img.shields.io/badge/Track-CRE_%26_AI-00B37E?style=for-the-badge" />
<img src="https://img.shields.io/badge/Stack-React_19_%2B_Node.js_22-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" />
<img src="https://img.shields.io/badge/Deployed-Vercel_%2B_Render-black?style=for-the-badge&logo=vercel" />
<img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" />

# CREator

### Visual Workflow Builder for Chainlink CRE

*The n8n / Node-RED of blockchain automation — powered by the Chainlink Runtime Environment*

</div>

---

## What is CREator?

CREator is a browser-based, no-code workflow editor that lets developers and non-developers design, simulate, and export Chainlink CRE workflows visually. Instead of writing TypeScript by hand and learning the CRE SDK from scratch, users drag nodes onto a canvas, connect them, and click a button to get a production-ready CRE project.

Think of it as the missing graphical layer on top of Chainlink CRE — the same way n8n sits on top of REST APIs or Node-RED sits on top of MQTT and IoT protocols.

---

## Who Does This Help?

| User | Problem Today | What CREator Solves |
|---|---|---|
| **Blockchain developers** | Writing CRE workflow TypeScript from scratch, learning SDK APIs | Generate correct, SDK-compatible code from a visual diagram in seconds |
| **Protocol teams** | Hard to onboard non-Solidity engineers onto CRE | Anyone who understands a flowchart can build a CRE workflow |
| **DeFi builders** | Manually wiring Chainlink Data Feeds, CCIP, and Functions together | Drag, connect, export — no SDK docs required |
| **Hackathon participants** | Limited time to learn a new orchestration framework | Scaffold a complete CRE project in under 5 minutes |
| **Enterprises adopting Web3** | Workflow automation tools (Zapier, n8n) don't speak blockchain | CREator bridges that gap without custom integration code |

---

## Is It Scalable?

Yes, by design.

- **Stateless backend.** The Express server holds no session state. Horizontal scaling on Render or any container platform requires only adding replicas behind a load balancer.
- **Client-side code generation.** The visual-to-TypeScript compiler runs entirely in the browser (no server round-trip). Adding new node types means adding an entry to a catalog object — no schema migrations, no redeployments.
- **Standard CRE output.** Every export produces a plain `@chainlink/cre-sdk` project. CREator is a generator, not a runtime dependency. Generated projects run independently on any CRE node.
- **Node catalog extensibility.** The node library is a declarative array of objects. Adding a new Chainlink capability (e.g., a future Confidential Compute node) requires writing one catalog entry and one code-generation template.
- **AI assistant is model-agnostic.** The ChatBot communicates with an OpenAI-compatible API. Switching from DeepSeek to GPT-4o, Claude, or a local model is a single environment variable change.

---

## System Architecture

```
+-------------------+        +-------------------------+        +---------------------------+
|                   |  REST  |                         |  File  |                           |
|   React Frontend  +------->+   Node.js Backend       +------->+   cre-orchestrator/       |
|   (Vercel)        |        |   (Render / Docker)     |        |   workflows/main.ts       |
|                   |        |                         |        |   workflow.yaml           |
|  - Flow canvas    |        |  - /api/compile         |        |   config.staging.json     |
|  - AI assistant   |        |  - /api/write-file      |        |                           |
|  - Code generator |        |  - /api/simulate        +<-------+   (simulation engine      |
|  - ZIP exporter   |        |  - /api/get-env-config  |        |    reads this directly)   |
|  - Wallet (wagmi) |        |  - /api/set-env-config  |        |                           |
|                   |        |                         |        +---------------------------+
+-------------------+        +-------+-----------------+
         |                           |
         | /api/chat                 | solc compiler
         v                           v
+-------------------+        +---------------------------+
|  Vercel Serverless|        |  solc 0.8.34              |
|  (DeepSeek proxy) |        |  (bundled, no toolchain)  |
+-------------------+        +---------------------------+
         |
         v
+-------------------+
|  DeepSeek API     |
|  (deepseek-chat)  |
+-------------------+
```

---

## Data Flow: Visual Canvas to Running CRE Workflow

```
User drags nodes          Canvas state           Code generator
and connects them   --->  (Zustand store)  --->  codeGenerator.ts
on the React Flow         nodes[], edges[]        produces TypeScript
canvas                                            main.ts + YAML

        |
        v
Backend /api/write-file
writes main.ts to
cre-orchestrator/workflows/

        |
        v
Backend /api/simulate
simulation engine reads main.ts
parses runtime.log() calls
returns CRE-formatted output

        |
        v
SimulationModal displays
[SIMULATION] Simulator Initialized
[USER LOG] Workflow triggered.
[USER LOG] HTTP Data Source (simulated): {...}
[USER LOG] Condition met, proceeding...
[SIMULATION] Execution finished
```

---

## AI-Assisted Workflow Generation

```
User types natural          /api/chat              DeepSeek
language description  --->  Vercel function  --->  deepseek-chat
"monitor ETH/USD                                   returns json-workflow
and trigger CCIP                                   spec block
if price drops"

        |
        v
ChatBot.tsx parseWorkflowJSON()
extracts node/edge spec

        |
        v
buildWorkflowOnCanvas()
places nodes with 400ms
animated delay, connects edges

        |
        v
Fully built canvas —
same as if user had
dragged nodes manually
```

---

## Component Map

```
front-end/src/
|
+-- pages/
|   +-- LandingPage.tsx         Entry point, project overview
|   +-- FlowBuilder.tsx         Main editor: canvas + panels + modals
|
+-- components/
|   +-- NodeLibrary.tsx         Draggable node template panel (left sidebar)
|   +-- ChatBot.tsx             AI assistant (natural language -> canvas)
|   +-- SimulationModal.tsx     Displays CRE simulation output
|   +-- SettingsModal.tsx       Private key configuration
|   +-- ContractEditorModal.tsx Solidity editor, compile, deploy
|   +-- ContractDetailsModal.tsx ABI viewer, Etherscan link, interactions
|   +-- IconMapper.tsx          Central react-icons registry
|   +-- nodes/CustomNode.tsx    React Flow node renderer
|
+-- utils/
|   +-- codeGenerator.ts        Visual flow -> CRE TypeScript + YAML
|   +-- flowValidation.ts       Connection rule enforcement
|   +-- projectTemplateGenerator.ts  ZIP export builder
|
+-- lib/blockchain/
|   +-- compile.ts              Calls backend /api/compile
|   +-- deploy.ts               wagmi/viem contract deployment
|   +-- solidityTemplates.ts    6 built-in Solidity contract templates
|   +-- contractStorage.ts      In-memory deployed contract registry
|
+-- store/flowStore.ts          Zustand: nodes, edges, selectedNode
+-- wagmi.ts                    RainbowKit + wagmi chain configuration
```

---

## Chainlink Integration Points

| Feature | Chainlink Technology Used |
|---|---|
| Workflow orchestration layer | Chainlink Runtime Environment (CRE) |
| Generated workflow SDK | @chainlink/cre-sdk 1.0.9 |
| Cron trigger node | CronCapability from cre-sdk |
| Data Feed node | Chainlink Price Feeds (oracle node type) |
| CCIP node | Chainlink CCIP cross-chain messaging |
| Functions node | Chainlink Functions (off-chain computation) |
| Data Streams node | Chainlink Data Streams |
| All generated workflows | CRE-compatible TypeScript targeting staging/production |

---

## Why CREator Deserves to Win

<div align="center">

![Track 2](https://img.shields.io/badge/Primary-Track_2:_CRE_%26_AI-00B37E?style=for-the-badge)
![Track 7](https://img.shields.io/badge/Also_eligible-Track_7:_Top_10-375BD2?style=for-the-badge)

</div>

---

### Track 2 — CRE & AI

> *"AI-assisted CRE workflow generation"* is listed verbatim in the track brief as an intended use case. CREator is not an adjacent interpretation — it is the canonical implementation of that use case.

**Claim 1: The AI does real work, not decoration.**

Most hackathon AI integrations call an LLM and display its response. CREator's AI assistant does something structurally different: the LLM returns a machine-readable specification (JSON with `nodes` and `edges` arrays), which the frontend then uses to programmatically construct a full React Flow canvas — placing nodes at calculated coordinates, wiring edges between them, and setting all configuration fields. A user saying *"monitor ETH/USD and trigger a CCIP message if price drops below threshold"* gets a fully built, ready-to-simulate CRE workflow in under 10 seconds. The AI is a compiler, not a chatbot.

| What most projects do | What CREator does |
|---|---|
| Ask LLM a question, display text answer | Ask LLM for a workflow spec, parse JSON, build live canvas |
| AI as a documentation layer | AI as a workflow construction engine |
| Human still writes CRE code manually | Canvas state drives code generation automatically |

**Claim 2: The generated output is a deployable CRE artifact, not pseudocode.**

Every workflow created through CREator — whether by drag-and-drop or AI generation — produces a `main.ts` file using real `@chainlink/cre-sdk` imports (`CronCapability`, `Runtime`, `handler`, `Runner`), a `workflow.yaml` with valid target definitions, and a `config.staging.json`. These files can be placed in any CRE workspace and deployed with the CRE CLI immediately. The AI lowers the barrier to CRE adoption by an order of magnitude.

**Claim 3: The AI-to-CRE pipeline is end-to-end and live-deployed.**

The full pipeline — natural language → DeepSeek API → canvas construction → code generation → simulation output — runs on the live Vercel deployment today. No local setup required. Judges can open a browser and test it in 60 seconds.

**Claim 4: CREator solves a real adoption problem for Chainlink.**

CRE adoption is limited by the SDK learning curve. CREator removes that barrier entirely for a large class of users — anyone who can describe a workflow in plain English or recognize shapes on a canvas. More CRE adoption = more value delivered to the Chainlink ecosystem. This is not a demo project; it is infrastructure for onboarding the next wave of CRE developers.

---

### Track 7 — Top 10 Projects

> Track 7 recognizes the best overall CRE projects. The criteria implied by the track are: depth of CRE integration, completeness of the project, and quality of execution.

**Claim 1: CREator covers the full CRE workflow lifecycle — not just one phase.**

| Phase | CREator feature |
|---|---|
| Design | Visual drag-and-drop canvas with 10+ node types (Cron, Data Feed, CCIP, Functions, HTTP, Condition, Transform, Log, Notification, Contract) |
| Validate | Connection rules enforced in real time (`flowValidation.ts`) — only valid CRE topologies can be saved |
| Simulate | Custom Node.js simulation engine returns authentic CRE CLI output format, including `[SIMULATION]` and `[USER LOG]` markers |
| Deploy (contracts) | Built-in Solidity editor with 6 templates, `solc 0.8.34` compile API, and `wagmi`/`viem` deployment to Sepolia |
| Export | One-click ZIP download of a complete `@chainlink/cre-sdk` project (main.ts + workflow.yaml + config + package.json) |

No other hackathon project is likely to cover all five phases with live deployments for each.

**Claim 2: Three production-grade services, not a monorepo prototype.**

CREator is three independently deployed services: a React frontend on Vercel, a Node.js API on Render inside Docker, and a CRE orchestrator workspace. Each service has its own README, environment configuration, and deployment pipeline. The architecture reflects production standards, not a hackathon shortcut.

**Claim 3: The simulation engine is a genuine technical contribution.**

Deploying the official CRE CLI on a headless server is impossible — it requires PowerShell on Windows and browser-based OAuth. CREator solves this by implementing a custom TypeScript parser that reads `main.ts` directly, extracts `runtime.log()` calls, evaluates conditional branches, and produces output indistinguishable from the real CRE CLI. This is a non-trivial engineering solution to a real infrastructure constraint, and it makes simulation available on any platform without any authentication.

**Claim 4: The project is immediately useful to the Chainlink ecosystem beyond the hackathon.**

CREator is not a proof of concept with hardcoded data. It is a general-purpose visual editor for any CRE workflow a developer might want to build. The node catalog, code generator, and simulation engine are all data-driven. The project is designed to grow — adding a new node type is a matter of adding one object to a catalog array and one code template. The groundwork for a community-maintained node library is already in place.

---

## Fulfillment of Common Requirements

| Requirement | Status |
|---|---|
| Build, simulate, or deploy a CRE Workflow | Simulation engine runs on every "Prove" click; export produces a runnable CRE project |
| Integrate blockchain with external data source or LLM | DeepSeek LLM for AI assistant; Chainlink Price Feeds, CCIP, Functions as node types; Solidity contracts deployed to Sepolia |
| Demonstrate simulation | /api/simulate returns authentic CRE CLI-formatted output including [SIMULATION] and [USER LOG] markers |
| Publicly viewable video | See repository README |
| Public source code | This organization — three repositories |
| README with Chainlink file links | See below |

---

## Repository Structure

| Repository | Purpose |
|---|---|
| [front-end](../../front-end) | React 19 + Vite visual editor, AI assistant, wallet integration, ZIP export |
| [back-end](../../back-end) | Node.js + Express API: Solidity compiler, simulation engine, file writer |
| [cre-orchestrator](../../cre-orchestrator) | CRE workspace: generated main.ts, workflow.yaml, config files |

---

## Chainlink Files

| File | Role |
|---|---|
| [cre-orchestrator/workflows/main.ts](../../cre-orchestrator/workflows/main.ts) | CRE workflow entry point — uses CronCapability, Runtime, handler, Runner from @chainlink/cre-sdk |
| [cre-orchestrator/workflows/workflow.yaml](../../cre-orchestrator/workflows/workflow.yaml) | CRE target definitions (staging-settings, production-settings) |
| [cre-orchestrator/workflows/config.staging.json](../../cre-orchestrator/workflows/config.staging.json) | Staging configuration (cron schedule) |
| [cre-orchestrator/workflows/package.json](../../cre-orchestrator/workflows/package.json) | Declares @chainlink/cre-sdk dependency |
| [front-end/src/utils/codeGenerator.ts](../../front-end/src/utils/codeGenerator.ts) | Generates CRE-compatible TypeScript from the visual canvas |
| [back-end/server.production.ts](../../back-end/server.production.ts) | Simulation engine that parses runtime.log() and returns CRE CLI output format |

---

## Live Deployment

<div align="center">

[![Frontend on Vercel](https://img.shields.io/badge/Frontend-Vercel-black?style=for-the-badge&logo=vercel)](https://creator-chainlink.vercel.app)
[![Backend on Render](https://img.shields.io/badge/Backend_API-Render-46E3B7?style=for-the-badge&logo=render)](https://creator-backend.onrender.com/health)
[![Judge Guide](https://img.shields.io/badge/Evaluation_Guide-JUDGES.md-375BD2?style=for-the-badge)](../../front-end/documentation/JUDGES.md)

</div>

---

## Tech Stack

<div align="center">

![React](https://img.shields.io/badge/React_19-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript_5.9-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite_7-646CFF?style=flat-square&logo=vite&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js_22-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express_4-000000?style=flat-square&logo=express&logoColor=white)
![Chainlink](https://img.shields.io/badge/Chainlink_CRE-375BD2?style=flat-square&logo=chainlink&logoColor=white)
![wagmi](https://img.shields.io/badge/wagmi_2-1C1B1F?style=flat-square)
![RainbowKit](https://img.shields.io/badge/RainbowKit_2-7B3FE4?style=flat-square)
![Zustand](https://img.shields.io/badge/Zustand_5-433E38?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)

</div>

---

<div align="center">

Built for the Chainlink Hackathon 2026 &nbsp;|&nbsp; MIT License

</div>