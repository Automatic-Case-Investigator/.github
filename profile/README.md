# ACI — Automatic Case Investigator

An on-premise AI stack that automates SOC investigation workflows. ACI ingests security case data from a SOAR platform, performs iterative log analysis in a SIEM, enriches findings with threat intelligence, and produces a written investigation report — with no analyst in the loop.

---

## The Problem

A single attacker can trigger a security incident, but effective incident response demands multiple skilled analysts working in parallel. As alert volumes scale, three compounding problems emerge:

| Problem | Impact |
|---|---|
| **Information fatigue** | Analysts reviewing thousands of log lines risk missing critical evidence |
| **Limited specialization** | No single analyst has deep expertise across every attack surface |
| **High manual effort** | Correlating events across a SIEM is slow and repetitive by nature |

ACI addresses all three by delegating the log-analysis and correlation work to AI agents, freeing analysts to focus on decisions that require human judgment.

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
│              Django · Redis · Pebble                    │
│  SOAR/SIEM integration, job scheduling, connectors      │
└──────┬──────────────────┬──────────────────┬────────────┘
       │ REST             │ REST             │ REST
┌──────▼──────┐  ┌────────▼───────────────┐ │
│    SOAR     │  │    ACI AI Backend      │ │
│  (The Hive) │  │  Django · Ollama       │ │
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
| [ACI Backend](./ACI_Backend) | Coordinates investigations, manages SOAR/SIEM connections, schedules async jobs, dispatches reports |
| [ACI AI Backend](./ACI_AI_Backend) | Runs the AI inference pipeline — task generation, SIEM query generation, relevancy filtering, report writing |
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

- **Live investigation view** — stream agent reasoning and intermediate query results to the UI as the investigation runs
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

<img width="744" height="577" alt="Screenshot 2026-05-28 at 5 04 22 PM" src="https://github.com/user-attachments/assets/4b987a77-c05a-4203-b3b0-ab29bb6ab094" />
