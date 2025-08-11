An on-premise software stack aimed to automate common SOC investigation tasks with AI agents, specifically by performing automated investigations in Security Information and Event Management (SIEM) systems. This software stack ingests security case information, generates investigation tasks, and automatically perform relevant investigations in SIEM. Models used including LLM for generation tasks and custom-designed classifier model for security event correlation.

---

## The Problem

- A single attacker can trigger a security incident, but incident response often requires **many SOC analysts**.
- Analysts must review and investigate **large volumes of security logs**, which leads to:
  - **Information fatigue**: risking missed evidence and reduced effectiveness
  - **Limited specialization**: analysts may not have expertise across all areas
  - **High effort**: correlating security logs is a **labor-intensive** task

---

## Features

### Investigation Procedures Generation
- Ingests security case data to **automatically generate investigation plans**
- Helps reduce time and effort required to structure an effective incident response

### Automatic Investigation
- Analyzes case data to generate SIEM queries
- Helps SOC analysts filter relevant logs, reducing manual workload
- Acts as a knowledge base, potentially uncovering missed evidence

---

## Supported platforms

### SOAR
- [The Hive](https://strangebee.com/)

### SIEM
- [Wazuh](https://wazuh.com/)

---

## Tentative Goals

- Develop a de-identification component to remove sensitive artifacts from case data and requests
