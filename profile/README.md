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
- Experiment showed continued pretrained model consistently outperforms openai/gpt-oss:20b in investigation planning: https://github.com/Automatic-Case-Investigator/ACI_Training_Experiment

### Automatic Investigation
- Analyzes case data to generate SIEM queries
- Iteratively queries the SIEM and evaluates potential Indicators of Compromise (IoC)
- Helps SOC analysts quickly locate pertinent logs, minimizing manual effort
- Acts as a knowledge base, potentially uncovering missed evidence

---

## Supported platforms

### SOAR
- [The Hive](https://strangebee.com/)


### SIEM
- [Wazuh](https://wazuh.com/)

---

## Tentative Goals
- Developing workflow for automatically investigate cases when they are added to SOAR
- Add settings to allow users to use external models
- Developing a generalized scheme allowing human SOC analysts to establish baselines in their organizational settings
- Developing a pipeline to retrieve relevant SIEM configurations to improve generation quality (Eg. alert rule sets)
- Developing a pipeline to generate incident response reports based on automatic investigation results


## Demo
<img width="1768" height="912" alt="cases" src="https://github.com/user-attachments/assets/eabe5277-a33c-474a-a945-99ca5b39bb4a" />
<p><em>Figure 1. List of security cases retrieved from the SOAR platform (e.g., TheHive).</em></p>

<img width="1456" height="716" alt="case" src="https://github.com/user-attachments/assets/4c8380ac-7cd2-409d-bdce-5455fde8818b" />
<p><em>Figure 2. Detailed information for a selected case retrieved from the SOAR platform (e.g., TheHive).</em></p>

<img width="1456" height="796" alt="automations" src="https://github.com/user-attachments/assets/b603bbe0-b386-412f-a9d2-694d65991878" />
<p><em>Figure 3. Available automation workflows that can be executed for the selected case.</em></p>

<img width="1454" height="717" alt="tasks" src="https://github.com/user-attachments/assets/91edffb9-f79b-47f4-a3e8-454a479168c2" />
<p><em>Figure 4. Investigation tasks automatically generated for the case.</em></p>

<img width="1450" height="754" alt="rev_shell" src="https://github.com/user-attachments/assets/dd521940-1f36-485f-8084-7ae5de611a18" />
<p><em>Figure 5. Automated investigation identifying a potential implanted reverse shell.</em></p>

<img width="1453" height="750" alt="rev_shell_log" src="https://github.com/user-attachments/assets/d6e65c9b-22a1-46d4-a090-df3278667d5b" />
<p><em>Figure 6. Evidence log retrieved by the automated investigation showing the reverse shell activity.</em></p>
