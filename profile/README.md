# ACI — Automatic Case Investigator

An on-premise AI stack that automates SOC investigation workflows. ACI ingests security case data from a SOAR platform, performs iterative log analysis in a SIEM, enriches findings with threat intelligence, and produces a written investigation report — with no analyst in the loop.

---

## The Problem

### The Asymmetry Between Attack and Defense

A single attacker can trigger a security incident that demands the simultaneous attention of many skilled defenders. This asymmetry is structural: launching an attack is a concentrated act, while investigating one is a distributed, multi-disciplinary effort. A single intrusion can span initial access, credential abuse, lateral movement, and data exfiltration — each phase leaving traces in different log sources, requiring different domain expertise to interpret. No individual analyst covers all of it with equal depth.

### The Scale Problem

Modern endpoints, network devices, and applications continuously stream telemetry into a SIEM. A mid-sized organization can produce millions of log events per day. When an alert fires, the relevant evidence for that specific incident is a small fraction of that volume — but identifying which events are relevant requires iterative, context-driven querying. Analysts cannot scan raw logs at scale. Without tooling that can execute many targeted queries and filter results automatically, investigations become sampling exercises, and evidence gets missed.

### The Investigative Process Is Inherently Multi-Step

A security investigation is not a lookup — it is a recursive process. A suspicious process execution leads to a query for child processes. That reveals a network connection, which leads to a query for other hosts that contacted the same destination. Those hosts surface additional artifacts, which require threat intelligence lookups. Each step produces new context that shapes the next query. Performing this loop manually is time-intensive, requires sustained attention, and is difficult to hand off between analysts mid-investigation without losing context.

### SIEM Query Expertise Is a Bottleneck

Writing precise SIEM queries demands familiarity with the platform's query language, the organization's log schema, and the specific detection logic in use (e.g., Wazuh rulesets). This expertise is not uniformly distributed across a SOC team. Junior analysts often cannot construct queries independently, senior analysts become the bottleneck, and critical investigation time is lost waiting for query review. Any system that can generate accurate, context-aware queries from a natural-language investigation task removes this bottleneck.

### Documentation and Knowledge Transfer

Incident documentation is consistently the lowest-priority task during an active investigation and the first one dropped under time pressure. Yet the investigation report is the primary artifact from which lessons are extracted, compliance requirements are satisfied, and future analysts learn. When reports are written under pressure, evidence citations are often incomplete or missing. And when a senior analyst who carried institutional knowledge of past incidents leaves the team, that knowledge leaves with them.

### The Data Sovereignty Constraint

Sending internal security logs to external AI services is not acceptable in most regulated environments. Log data contains sensitive operational detail — user activity, network topology, internal hostnames, credential usage patterns. Organizations in finance, healthcare, government, and critical infrastructure require that log data remain entirely on-premise. This rules out cloud-hosted AI investigation tools and creates a hard requirement for a self-hosted inference stack.

### Summary of Requirements

| Requirement | Implication for ACI |
|---|---|
| Investigations require multi-step, adaptive querying | Agent must iterate — each query informs the next |
| Relevant events are sparse within high-volume log streams | Relevancy filtering must run per-query before passing results to the AI |
| SIEM queries require platform and schema expertise | Query generation must consume SIEM configs (rulesets, schema) as context |
| Findings must be traceable to real log events | Every claim in a report must be grounded in a retrieved SIEM event |
| Investigation must begin immediately on case creation | SOAR integration must trigger the pipeline autonomously |
| Logs cannot leave the organization's infrastructure | Inference must run fully on-premise via a self-hosted model server |
| Analyst capacity is limited and unevenly skilled | Automation must handle the full investigation loop, not just assist |

---

## Architecture

ACI is composed of three deployable services:

```
┌─────────────────────────────────────────────────────────┐
│                    ACI Dashboard                        │
│          React 19 · TypeScript · MUI 7                  │
│  Investigation controls, case browsing, job monitoring  │
└────────────────────────┬────────────────────────────────┘
                         │ REST / JWT
┌────────────────────────▼────────────────────────────────┐
│                    ACI Backend                          │
│               Django · Redis · RQ                       │
│  SOAR/SIEM integration, job scheduling, connectors      │
└──────┬──────────────────┬──────────────────┬────────────┘
       │ REST             │ REST             │ REST
┌──────▼──────┐  ┌────────▼───────────────┐ │
│    SOAR     │  │    ACI AI Backend      │ │
│  (The Hive) │  │   Django · vllm        │ │
│             │  │  Query gen, report gen │ │
└─────────────┘  │  relevancy filtering   │ │
                 └────────────────────────┘ │
                                            │
               ┌────────────────────────────▼────────────┐
               │              SIEM Platform               │
               │               (Wazuh)                    │
               └──────────────────────────────────────────┘
```

| Component | Role |
|---|---|
| [ACI Backend](./ACI_Backend) | Coordinates investigations, manages SOAR/SIEM connections, schedules async jobs via RQ, dispatches reports |
| [ACI AI Backend](./ACI_AI_Backend) | Runs the AI inference pipeline via vllm — task generation, SIEM query generation, relevancy filtering, report writing |
| [ACI Dashboard](./ACI_Dashboard) | Web UI for configuring platforms, browsing cases, triggering investigations, and monitoring jobs |

---

## Features

### Investigation Procedure Generation

ACI reads the case title, description, and observables from the SOAR platform and automatically produces a structured investigation plan — a set of tasks and sub-activities that guide the agent through the incident.

A fine-tuning experiment showed a continued-pretraining model consistently outperforms `openai/gpt-oss:20b` on investigation planning. See [ACI Training Experiment](https://github.com/Automatic-Case-Investigator/ACI_Training_Experiment) for methodology and results.

### Automatic Log Investigation

For each investigation activity the agent:

1. Analyzes the case context and any available SIEM configurations (e.g., detection rulesets) to generate targeted queries
2. Executes queries against the SIEM and evaluates returned events for Indicators of Compromise (IoC)
3. Enriches suspicious artifacts using threat intelligence platforms (VirusTotal, AbuseIPDB)
4. Iterates — refining queries based on intermediate findings — until the activity is resolved or the iteration budget is exhausted

The result is a ranked set of relevant log events for each activity, without an analyst having to write a single query.

### Endpoint Behavioral Baselines

ACI computes per-endpoint behavioral baselines from historical SIEM data (configurable lookback window). Baselines capture normal active hours, active days, and field-level distributions. During an investigation, deviations from baseline surface automatically as anomaly signals.

### Consolidated Investigation Reporting

After all activities complete, ACI correlates findings across tasks to produce a holistic incident report. Each claim in the report is backed by real log events retrieved from the SIEM. Reports can be sent automatically to:

- Discord
- Telegram
- Gmail
- Webhook (any HTTP endpoint)

### Autonomous End-to-End Mode

All of the above can run fully autonomously. When a new case is created in the SOAR platform, ACI detects it, runs the full investigation pipeline, and delivers the report — with no human trigger required.

### Live Copilot Mode

Analysts watch the agents investigate in real time via the UI. Analysts can inject directional prompt guidance mid-investigation to steer the agent down specific forensic paths.

---

## Supported Platforms

| Category | Platform |
|---|---|
| SOAR | [The Hive](https://strangebee.com/) |
| SIEM | [Wazuh](https://wazuh.com/) |

---

## Installation

Each component ships with a Docker Compose file. Clone all three repositories and start them in order.

### 1. ACI AI Backend

```bash
cd ACI_AI_Backend
cp sample.env .env          # fill in model paths and API keys
# Linux / Mac
sudo docker compose -f docker-compose.yml build
sudo docker compose -f docker-compose.yml up
# Windows
docker compose -f docker-compose-windows.yml build
docker compose -f docker-compose-windows.yml up
```

### 2. ACI Backend

```bash
cd ACI_Backend
cp sample.env .env          # set AI_BACKEND_URL, Redis config, SIEM/SOAR endpoints
sudo docker compose -f docker-compose.yml build
sudo docker compose -f docker-compose.yml up
```

### 3. ACI Dashboard

```bash
cd ACI_Dashboard
cp sample.env .env          # set REACT_APP_BACKEND_URL
sudo docker compose -f docker-compose.yml build
sudo docker compose -f docker-compose.yml up
```

After all three services are running, open the Dashboard and navigate to **Settings** to configure your SOAR and SIEM connections.

---

## Roadmap

- **External model support** — allow users to point ACI at any OpenAI-compatible inference endpoint instead of the bundled Ollama stack
- **Adversarial validation** — run an APT-style emulation against a test environment and measure detection and reporting accuracy end-to-end

## Demo
<img width="1355" height="675" alt="Screenshot 2026-05-21 at 11 18 55 AM" src="https://github.com/user-attachments/assets/1ff8fd0c-8ef5-4150-add1-0485026633d6" />

<img width="1358" height="677" alt="Screenshot 2026-05-21 at 11 19 11 AM" src="https://github.com/user-attachments/assets/3f8a1599-a1e2-49d3-a14a-b29aeece5379" />

<img width="1356" height="677" alt="Screenshot 2026-05-21 at 11 19 35 AM" src="https://github.com/user-attachments/assets/78e97ef1-17be-4aed-9832-91e306c9e056" />

<img width="1358" height="678" alt="Screenshot 2026-05-21 at 11 21 06 AM" src="https://github.com/user-attachments/assets/8930faa4-953f-4bf5-a6ac-c638287abbb5" />

<img width="1354" height="674" alt="Screenshot 2026-05-21 at 11 21 52 AM" src="https://github.com/user-attachments/assets/20de8cd8-987e-4dcd-bba1-675b668c3714" />

<img width="1360" height="669" alt="Screenshot 2026-05-21 at 11 22 39 AM" src="https://github.com/user-attachments/assets/1a862797-b2db-4c6f-86b6-1def1702c024" />

<img width="1357" height="676" alt="Screenshot 2026-05-21 at 11 22 57 AM" src="https://github.com/user-attachments/assets/020ef38c-77f6-4102-99d3-8f0f175744ae" />

<img width="1465" height="677" alt="Screenshot 2026-06-03 at 5 25 58 PM" src="https://github.com/user-attachments/assets/918e2d75-e3f9-4036-991c-f7e73503a6c6" />

<img width="744" height="577" alt="Screenshot 2026-05-28 at 5 04 22 PM" src="https://github.com/user-attachments/assets/4b987a77-c05a-4203-b3b0-ab29bb6ab094" />
