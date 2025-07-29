# 👋 Hi There

This organization is focused on researching how to leverage AI to automate the case investigation process within a Security Operations Center (SOC). It explores the integration of widely-used open-source SOC technologies for incident response.

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

## Tentative Goals

- Train an event classifier model to distinguish **true positives** from **false positives**
- Develop a de-identification component to remove sensitive artifacts from case data and requests
