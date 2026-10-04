# Threat Hunting Field Guide

A practical standalone handbook for Ronald, developed from the SOC Analyst project study topics and recovered LetsDefend discussion.

**The tool provides evidence. The analyst decides what the evidence means.**

## How to use this handbook

Choose a hunting question, identify the evidence that can answer it, test competing explanations, and record a defensible conclusion. Tool entries emphasize what question the tool answers and what to inspect next.

- **Lesson recap**: material recovered from the LetsDefend discussion.
- **Study recap**: requested study topics and terminology; earlier lesson text was not available in full.
- **Reference notes**: tool behavior supported by official documentation.
- **Practical analyst notes**: added workflows, examples, and exercises rather than lesson quotations.

**Source limitation:** the exact five official LetsDefend Threat Hunting Life Cycle stage names remain unverified. The lifecycle section flags this explicitly and keeps the two requested practical workflows separate from official lesson terminology.

## Contents

- [Threat Hunting Introduction](#threat-hunting-introduction)
- [Threat Hunting Methodologies](#threat-hunting-methodologies)
- [Threat Hunting Life Cycle](#threat-hunting-life-cycle)
- [Threat Hunting Tools and Data Collection](#threat-hunting-tools-and-data-collection)
- [Data Analysis Tools](#data-analysis-tools)
- [Network Monitoring Tools](#network-monitoring-tools)
- [Network and System Monitoring Context](#network-and-system-monitoring-context)
- [Wazuh / OpenSearch Threat Hunting Filter Field Guide](#wazuh--opensearch-threat-hunting-filter-field-guide)
- [Worked Hunt from Hypothesis to Action](#worked-hunt-from-hypothesis-to-action)
- [Ronald Hands On Revisit Plans](#ronald-hands-on-revisit-plans)
- [Editable Hunt Worksheet](#editable-hunt-worksheet)
- [Sources and Terminology Verification](#sources-and-terminology-verification)

**Hands-on revisit priorities:** [Snort, Zeek, and Vectra NDR](#ronald-hands-on-revisit-plans).

Last updated: October 4, 2026.

## Threat Hunting Introduction

**Study recap**

Threat Hunting is a proactive search for adversary activity that may have escaped existing detections. It starts with a reasoned question about the environment and uses available evidence to investigate it. A hunt may discover an incident, a benign explanation, or a gap in visibility.

| Approach | Starting point | Analyst question |
| --- | --- | --- |
| Reactive security | An alert, reported incident, or known problem | What happened, how serious is it, and what response is needed? |
| Proactive security | A hypothesis, TTP, intelligence lead, or unexplained behavior | Could this activity be occurring even though our current detections have not alerted? |

**Practical analyst notes**

Reactive investigation can produce a proactive hunt: after one confirmed incident, search the wider environment for the same behavior. Proactive does not mean guessing without evidence, and a quiet alert queue does not establish that the environment is safe.

### Threat Hunting Team and Competencies

The following is a practical division of responsibilities, not a recovered list of LetsDefend job titles. In a small team, one person may perform several roles.

| Role | Contribution to the hunt |
| --- | --- |
| Threat hunter or lead | Frames the question, scopes the hunt, tests explanations, and records conclusions. |
| Threat intelligence analyst | Supplies relevant adversary TTPs and IoCs with source, recency, and confidence. |
| Detection or data engineer | Checks telemetry, ingestion, parsing, retention, and repeatable detection logic. |
| SOC analyst and incident responder | Validates findings, investigates impact, and handles escalation and response. |
| System or network owner | Explains normal activity, maintenance, asset purpose, and operational constraints. |

Build competencies in network protocols; Windows and endpoint activity; log searching and correlation; TTPs and MITRE ATT&CK; threat intelligence evaluation; scripting; and clear reporting. Critical thinking connects them: ask what would disprove your explanation, and seek that evidence.

## Threat Hunting Methodologies

**Study recap with practical analyst notes**

### Adversary-Centric Approach

Adversary-Centric hunting focuses on the adversary's Tactics, Techniques, and Procedures (TTPs). Use MITRE ATT&CK to organize observed behavior and identify what evidence your environment should contain. A tactic describes an objective, a technique describes a way to achieve it, and a procedure is a specific implementation. [3]

Ask: “If an attacker used this technique against our assets, what observable activity would it leave?” Choose behavior relevant to your systems, then identify the logs required to test it. An ATT&CK mapping organizes evidence; it does not by itself prove compromise or actor attribution.

### Hypothesis-Driven Approach

Hypothesis-Driven hunting begins with a testable proposition. Example: “A workstation may be making repeated outbound connections because an unauthorized process is communicating with external infrastructure.” Specify the host group, time window, expected evidence, and benign alternatives.

Ask: “What would support this hypothesis, and what would weaken it?” Collect connection, process, DNS, and asset context; compare the activity with normal behavior. Refine the hypothesis when the data points elsewhere. Avoid treating every anomaly as confirmation.

### IoC-Based Approach

IoC-Based hunting starts from an Indicator of Compromise, such as a file hash, domain, IP address, or other observable associated with malicious activity. Record the intelligence source, observation time, confidence, and relevance before searching.

Ask: “Did this indicator appear here, and what happened around the match?” Validate the matching field and timing, then pivot to related hosts, users, processes, and destinations. A shared IP or old indicator may have a benign explanation. No match does not rule out a threat that changed infrastructure or artifacts.

| Method | Best starting question | Useful next move |
| --- | --- | --- |
| Adversary-Centric / TTP | How would this adversary behavior appear here? | Map behavior to available telemetry. |
| Hypothesis-Driven | Can the available data test this proposition? | Test supporting and competing explanations. |
| IoC-Based | Where and when did this indicator appear? | Validate the match and expand into behavior. |

Practical analyst note: these methods can work together. An IoC hit can seed a hypothesis, and a TTP hunt can reveal new indicators. Keep the starting rationale and later pivots in the record.

## Threat Hunting Life Cycle

**Lesson terminology verification required**

LetsDefend lists Threat Hunting Life Cycle in its Introduction to Threat Hunting course. The exact five official stage names and their descriptions were not present in the recovered conversation or accessible public course outline. They remain unverified in this edition. [1]

For lesson study or quiz answers, use the exact names shown in Ronald's lesson. The five steps below are the requested practical workflow, not a substitute presented as LetsDefend's official terminology. Once the lesson wording is available, add it alongside this workflow and verify the correspondence.

### Practical workflow for managing a hunt

**Plan → Collect → Hunt → Report → Improve**

| Step | Question | Record or output |
| --- | --- | --- |
| Plan | What are we testing and why? | Hypothesis, scope, time window, owner, expected evidence, and stop conditions. |
| Collect | Do we have trustworthy data? | Sources, logging settings, retention, sensor coverage, timestamps, and gaps. |
| Hunt | What does the activity mean in context? | Queries, pivots, timelines, baseline comparisons, and alternative explanations. |
| Report | What can we support with evidence? | Finding, confidence, supporting evidence, limitations, and assigned actions. |
| Improve | What should change after this hunt? | Detection updates, logging fixes, playbook changes, and follow-up hypotheses. |

### Practical workflow for testing a hypothesis

**Hypothesis → Collect Data → Analyze → Test → Conclusion → Action**

Hypothesis defines a testable claim. Collect Data establishes the relevant evidence and coverage. Analyze organizes events and finds patterns. Test checks the pattern against benign explanations and independent sources. Conclusion states what is supported, unsupported, or inconclusive. Action assigns response or improvement work.

The reasoning workflow fits inside the broader hunt workflow. Hypothesis helps Plan; Collect Data supports Collect; Analyze and Test occur during Hunt; Conclusion belongs in Report; Action can begin response immediately and feed Improve. This is a practical mapping, not an official LetsDefend stage mapping.

**Practical analyst note**

Escalate credible evidence of active compromise promptly. Do not wait for the report to be finished. If required data is absent, record an inconclusive result and a collection improvement rather than treating the hypothesis as disproved.

## Threat Hunting Tools and Data Collection

**Study recap**

Keep the LetsDefend category names: Data Collection Tools; Data Analysis Tools; Network Monitoring Tools; Endpoint Detection and Response (EDR) Tools; Cyber Threat Intelligence (CTI) Tools; and Integration and Automation of Threat Hunting Tools. The public Tools course lists these headings. This edition develops the collection, analysis, and network tools requested so far. [2]

**Practical analyst notes**

EDR contributes endpoint context and investigation or response capabilities. CTI supplies leads and context. Integration and automation connect sources, enrichment, searches, and repeatable tasks. Use these categories to choose evidence, not to assume each product has only one function.

| Tool | What question does it answer? | Evidence and next check |
| --- | --- | --- |
| Sysmon | What detailed activity occurred on this Windows host? | Process creation, network connections, and other configured events. Check command line, parent process, user, and timing. [4] |
| Winlogbeat | How can Windows event logs reach our analysis platform? | Reads selected event logs and ships events to configured outputs. Verify the channels, filters, delivery, and parsed fields. [5] |
| NXLog | How can we collect and forward logs from our sources? | Collection and forwarding agent with configured inputs and outputs. Check source selection, parsing, buffering, and delivery. [6] |
| Graylog | Where can we centrally collect, organize, and search logs? | Central log management, parsing, and analysis. Confirm the expected sources arrive and remain searchable. [7] |

Practical distinction: Sysmon generates telemetry. Winlogbeat and NXLog move selected logs. Graylog centralizes and makes them useful. Graylog also performs analysis, even though it is retained here under the requested Data Collection Tools heading.

### A collection pipeline you can verify

Example: Windows activity → Sysmon events → Winlogbeat or NXLog → an analysis platform. Winlogbeat may send to Elasticsearch directly or through Logstash. Treat this as a design example; verify the actual deployment before relying on it.

- Generate one known benign event on a lab host and find it in the destination.

- Compare source timestamp, ingestion timestamp, hostname, event type, and key fields.

- Check retention and gaps. A shipper cannot recover events that were never logged.

## Data Analysis Tools

**Reference notes with practical analyst guidance**

Analysis platforms help connect observations across time, hosts, users, and data sources. Their usefulness depends on what was collected and how it was parsed. A dashboard cannot establish the truth of a hypothesis without reviewing the underlying events.

### Splunk

Question: “What related events can I find across the collected data?” Use searches, statistics, timelines, and dashboards to identify patterns and pivot from an entity into related activity. For a suspicious destination, find contacting hosts, then correlate process and authentication events. [8]

Practical analyst note: confirm the index, source type, time zone, field extraction, and host coverage. Save the search and time boundaries. An empty search can result from an incorrect field or missing source, not just absent activity.

### ELK Stack components

ELK preserves the lesson abbreviation: Elasticsearch, Logstash, and Kibana. Beats, including Winlogbeat, are collection agents in the broader Elastic Stack. [5, 9]

| Component | What question does it answer? | Analyst use |
| --- | --- | --- |
| Elasticsearch | Where is the indexed data, and what records match? | Stores, indexes, and searches data. Confirm index selection and field types. |
| Logstash | How should incoming records be processed and routed? | Ingests, transforms, and forwards events. Check parsing and timestamp conversion. |
| Kibana | How can I explore and visualize the indexed events? | Searches and visualizes Elasticsearch data. Inspect raw events behind summaries. |
| Beats / Winlogbeat | How does source telemetry reach the stack? | Lightweight shipping of configured data. Verify inputs, outputs, and delivery. |

### LogRhythm

Question: “Which collected events form a suspicious sequence or relationship?” Use SIEM search and correlation to relate activity across sources and investigate alarms in context. For example, connect unusual authentication with subsequent host or network activity. [10]

Practical analyst note: inspect the raw events behind a correlated finding. Normalization, rule configuration, and available sources affect the result. A priority score or alarm is an investigative lead, not a final verdict.

### Make analysis repeatable

Start broad enough to understand the population, then narrow by entity and time. Count affected hosts, compare with peer systems, and inspect representative events. Record queries and exclusions so another analyst can reproduce the result. Use the search language and schema of the actual platform; field names are not interchangeable.

## Network Monitoring Tools

**Study recap and reference notes**

| Tool | Core distinction | What question does it answer? |
| --- | --- | --- |
| Wireshark | Dissect actual packets | What exactly is in this captured exchange? [11] |
| Snort | Detect using rules/signatures | Does this traffic match a detection rule/signature? [12] |
| Zeek | Record/describe network activity through detailed logs/events | What network activity occurred, and how do the connections relate? [13] |
| Vectra NDR | Surface anomalous network behavior using AI/ML | Which network behavior looks abnormal or suspicious enough to investigate? [14] |

### Wireshark

Practical analyst note: narrow a packet capture by host, destination, protocol, and time. Examine protocol fields, requests and responses, retransmissions, or stream contents where visible. Capture location and encryption limit what you can see. A packet capture describes traffic at its observation point; it does not automatically identify the responsible process.

### Snort

Practical analyst note: inspect the rule identifier, rule logic, direction, source and destination, and matching traffic. Correlate the alert with packet, host, and asset context. Snort can operate as IDS or IPS; blocking depends on deployment and configuration. No alert does not prove absence of malicious traffic. [12]

**Ronald's hands-on revisit priority: Snort**

### Zeek

Practical analyst note: begin with conn.log for connection metadata; pivot to dns.log or http.log when available. Correlate by time, endpoints, and connection UID where applicable. Look for frequency, volume, unusual peers, and protocol behavior. Zeek logs summarize observations; they are not a replacement for full packet contents. [13]

**Ronald's hands-on revisit priority: Zeek**

### Vectra NDR

Lesson recap: Network Detection and Response monitors network traffic and uses AI and machine learning to surface abnormal behavior. Practical analyst note: investigate the behavior explanation, affected entity, timing, and related activity; compare with approved operations. An AI/ML finding still needs corroboration and a defensible analyst conclusion. [14]

**Ronald's hands-on revisit priority: Vectra NDR**

## Network and System Monitoring Context

### PRTG Network Monitor

**Lesson recap from the recovered discussion**

The recorded feature names are Comprehensive Monitoring, Sensors, Real-time Alerts, User-Friendly Interface, and Mobile Access. The discussion describes monitoring network devices, traffic, and applications with customizable sensors and alerts for network problems.

**Practical analyst notes**

Question: “Something unusual is happening on the network. Where should I investigate?” Review the affected sensor, time, baseline, and device. A bandwidth change or availability problem can provide a useful starting point. Then collect security evidence that can explain it. [15]

A spike may reflect backups, updates, congestion, or unauthorized transfer. PRTG can point to a condition worth investigating; individual packet dissection is the Wireshark question.

### Nagios

**Lesson recap from the recovered discussion**

The discussion identifies network/device and service monitoring, threshold alerts, customization, plugins/integrations, and open source. Keep the product distinction in mind: Nagios Core is the open-source monitoring project; other Nagios products have their own capabilities and licensing. [16]

**Practical analyst notes**

Question: “Something abnormal happened to a system or service. Does it have security significance?” Review the check result, state changes, duration, affected hosts, and maintenance context. Pivot to security logs when an outage or resource change has an unexplained cause. Plugins determine what a configured check can observe. [16]

### Move from a condition to evidence

| Starting observation | Next evidence | Question to resolve |
| --- | --- | --- |
| Bandwidth spike | Connection logs, DNS, endpoint process data | Which hosts and processes explain the transfer? |
| Service unavailable | System, application, authentication, and change logs | Was this failure, maintenance, or unauthorized activity? |
| Snort rule match | Rule logic, packet detail, host context | Does the match represent the behavior we suspect? |
| Vectra behavioral finding | Related activity and independent telemetry | Is the behavior authorized and consistent with the asset's role? |

Practical analyst rule: anomalies establish a reason to investigate. They do not establish attacker intent. Align time zones and use the asset's actual role before connecting events into a narrative.

## Wazuh / OpenSearch Threat Hunting Filter Field Guide

**Practical analyst notes from the DMZ hands-on hunt**

Use fields to translate a hunting question into evidence. Do not memorize filter combinations without understanding what each field represents.

| Field | Meaning | Threat hunting use |
| --- | --- | --- |
| `rule.groups` | Category Wazuh assigned to an event | Narrow the hunt to event types such as `firewall`, `sshd`, `authentication_failed`, or `yum`. |
| `rule.description` | Human-readable reason the rule fired | Quickly understand what Wazuh detected. |
| `rule.id` | Wazuh rule number | Find events produced by one specific detection rule. |
| `rule.level` | Wazuh alert severity | Help prioritize events while retaining context. |
| `data.srcip` | Source IP | Identify where traffic or activity originated. |
| `data.dstip` | Destination IP | Identify the host or network being targeted. |
| `data.srcport` | Source port | Inspect the originating side of a network connection. |
| `data.dstport` | Destination port | Identify the targeted service or port, such as 22 for SSH. |
| `data.action` | Device action | Filter firewall decisions such as `allow`, `accept`, or `deny`. |
| `data.service` | Identified service | Add protocol/service context such as SSH. |
| `agent.ip` | IP of the Wazuh-monitored host reporting the event | Identify which monitored endpoint produced/reported the event. |
| `agent.name` | Name of the monitored host | Hunt activity on a named system such as `webserver`. |
| `data.dstuser` | Target/destination account | Identify which user account was targeted. |
| `predecoder.program_name` | Program that generated the log | Identify sources such as `sshd` or `yum`. |
| `full_log` | Original/raw log message | Verify details that parsed fields may not fully expose. |
| `data.audit.command` | Command recorded by Linux Audit | Identify an executed command such as `nmap`. |
| `data.audit.exe` | Executable path | Verify which executable actually ran. |
| `data.audit.execve.a0`, `a1`, `a2`... | Command-line arguments | Reconstruct exactly how a command was executed and what it targeted. |

### Critical field distinction

`agent.ip` is not the same thing as `data.srcip` or `data.dstip`.

- `data.srcip` = where the activity came from.
- `data.dstip` = where the activity was going.
- `agent.ip` = which Wazuh-monitored endpoint reported the event.

DMZ hunt example: `data.srcip: 172.16.8.190` showed that the compromised webserver became the source of later SSH activity. An `agent.ip` such as `10.10.10.22` identifies the monitored destination system reporting that SSH attempt. Filtering only on `agent.ip: 172.16.8.190` can therefore hide evidence generated by the destination hosts.

### Build filters from the hunting question

Use this reasoning chain:

**Question → evidence type → source/destination → action/result → filter → inspect raw evidence**

Example: “Find failed SSH authentication originating from 172.16.8.190.”

1. Evidence type: SSH → `rule.groups: sshd`
2. Origin: 172.16.8.190 → `data.srcip: 172.16.8.190`
3. Result: failed authentication → `rule.groups: authentication_failed`
4. Inspect the matching events and `full_log`; do not treat the filter itself as the conclusion.

Practical rule: first understand what the question is asking, then identify which field represents each part of the question. Use the filtered results as evidence and verify the underlying event before reporting a conclusion.

## Worked Hunt from Hypothesis to Action

**Practical analyst example only**

Scenario: a workstation appears to contact an external destination repeatedly. The activity might be unauthorized command and control, but software updates or a scheduled health check could produce a similar pattern. No actual Ronald case or lab result is asserted here.

### Hypothesis and plan

Hypothesis: “An unauthorized process on a workstation is making recurring outbound connections.” Scope the selected Windows hosts and the last 24 hours. Expect repeated connections tied to a process, with context inconsistent with approved software. Record which systems and times are covered before querying.

### Collect Data

Use Zeek connection and DNS logs, configured Sysmon events, and asset inventory. Winlogbeat or NXLog may deliver Windows logs to the selected analysis platform. Preserve a packet capture if already available. Verify the sensor can see this traffic and that endpoint network logging was enabled during the period.

### Analyze

In Splunk, Kibana/Elasticsearch, or LogRhythm, group connections by source and destination. Inspect intervals, duration, volume, and other hosts using the destination. Build a timeline. Pivot from the host and time to the process, parent, user, command line, and DNS observations.

### Test

Compare with approved software and peer hosts. Check scheduled jobs, update services, and change records. Use Wireshark to examine available packet details and Snort alerts to identify any rule matches. Review Vectra findings if the environment provides them. Do not require all tools to agree before examining the underlying evidence.

### Conclusion

| Possible outcome | Defensible wording |
| --- | --- |
| Evidence supports malicious activity | Related host and network evidence supports unauthorized activity; preserve records and escalate under the response process. |
| Benign explanation supported | Observed activity is consistent with the documented updater and expected timing; retain the evidence and rationale. |
| Insufficient visibility | The hypothesis remains inconclusive because process or network telemetry is missing for part of the scope. |

### Action and improvement

Assign an owner to response, logging fixes, or detection tuning. Preserve the evidence and query history. Record the follow-up hypothesis if the hunt reveals a wider pattern. Avoid claiming “the environment is clean” from a limited hunt.

## Ronald Hands On Revisit Plans

**Practical analyst exercises**

Snort, Zeek, and Vectra NDR are Ronald's explicit hands-on revisit priorities. Use a permitted lab or supplied training data. Each exercise should end with an evidence record and a conclusion; these plans do not claim that the exercises have already been completed.

### Snort revisit

Goal: explain why a rule/signature produced an alert. Use a benign lab capture and a simple training rule, or a supplied alert with its matching capture. Record the Snort version, configuration, and rules used.

- Identify the rule ID, matching conditions, traffic direction, and packet or stream evidence.

- Compare one matching exchange with one nonmatching exchange. Explain the difference.

- Use Wireshark to inspect the relevant traffic and write a verdict with a benign alternative.

Completion evidence: the alert, rule explanation, packet reference, and a short conclusion. Ask: “What did this rule establish, and what still needs investigation?”

### Zeek revisit

Goal: reconstruct network activity from detailed logs/events. Process the same permitted capture where practical, so the packet view and log view describe the same traffic.

- Summarize conn.log by source, destination, protocol/service, duration, and byte counts.

- Pivot to available DNS or HTTP records using connection identifiers and timestamps.

- Compare the resulting timeline with Wireshark. Document what the logs preserve and omit.

Completion evidence: a connection timeline, related records, and one explained anomaly or unusual pattern. Ask: “What happened, and which further evidence would help explain it?”

### Vectra NDR revisit

Goal: validate a behavioral detection. Use an authorized training tenant or supplied detection walkthrough. If access is unavailable, record this as a pending hands-on exercise; a documentation review is preparation, not hands-on completion.

- Identify the affected entity, behavior explanation, detection time, and related activity.

- Check approved operations and correlate with independent host or network evidence.

- Record whether evidence supports compromise, supports a benign cause, or is inconclusive.

Completion evidence: detection details, corroboration, limitations, and the analyst decision. Ask: “Why is this behavior suspicious in this environment?”

## Editable Hunt Worksheet

**Practical analyst template**

Copy this page for each hunt. Replace the prompts with actual evidence and decisions. Preserve uncertainty and the original time boundaries.

### Hunt definition

Hunt ID and title: ______________________________________________  
Analyst and owner: ______________________________________________  
Date and time zone: ______________________________________________

Hypothesis or starting IoC/TTP: ___________________________________  
Reason for the hunt and intelligence source: ________________________  
Systems and time window in scope: _________________________________  
Out of scope and stop conditions: __________________________________

### Data and coverage

Sources and collection settings: __________________________________  
Retention and sensor/host coverage: _________________________________  
Known gaps and timestamp checks: __________________________________

### Investigation record

| Time and query or pivot | Evidence reference | Interpretation and next test |
| --- | --- | --- |
| Enter exact query and time range | Record or file location | Separate observation from inference |
|  |  |  |
|  |  |  |

### Conclusion and action

Outcome: supported / benign explanation supported / unsupported within scope / inconclusive  
Confidence and reason: ___________________________________________  
Strongest supporting evidence: ____________________________________  
Competing explanation and test: ___________________________________  
Limitations: ____________________________________________________  
Action, owner, and due date: _______________________________________  
Detection or collection improvement: _______________________________

### Before closing the hunt

- Can another analyst reproduce the search and locate the evidence?

- Does the conclusion respect the coverage, time window, and uncertainty?

- Are the response, improvement, and follow-up actions assigned?

## Sources and Terminology Verification

This handbook combines Ronald's requested study scope, recovered project discussion, and the official references below. Bracketed reference numbers identify supporting documentation; added examples and workflows are marked as practical analyst notes. Reviewed October 2, 2026.

Project reference: the recovered discussion from Continue SOC Analyst Part 4 covered Vectra NDR, PRTG, Nagios, and the handbook request. Earlier lesson material was unavailable; the guide marks the resulting terminology gap.

[1] [LetsDefend Introduction to Threat Hunting](https://app.letsdefend.io/training/lessons/introduction-to-threat-hunting)

[2] [LetsDefend Threat Hunting Tools](https://app.letsdefend.io/training/lessons/threat-hunting-tools)

[3] [MITRE ATT&CK knowledge base](https://attack.mitre.org/)

[4] [Microsoft Sysmon documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

[5] [Elastic Winlogbeat overview](https://www.elastic.co/guide/en/beats/winlogbeat/current/_winlogbeat_overview.html)

[6] [NXLog Windows log collection](https://docs.nxlog.co/userguide/os/windows.html)

[7] [Graylog documentation](https://go2docs.graylog.org/current/what_is_graylog/what_is_graylog.htm)

[8] [Splunk search documentation](https://help.splunk.com/en/splunk-enterprise/search/search-manual)

[9] [Elastic Stack documentation](https://www.elastic.co/docs/get-started/the-stack)

[10] [LogRhythm Advanced Intelligence Engine](https://logrhythm.com/wp-content/uploads/2020/03/advanced-intelligence-engine-data-sheet.pdf)

[11] [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/index.html)

[12] [Snort 3 rules documentation](https://docs.snort.org/start/rules)

[13] [Zeek logs documentation](https://docs.zeek.org/en/v8.2.0/tutorial/logs.html)

[14] [Vectra AI platform](https://www.vectra.ai/platform)

[15] [PRTG sensors](https://www.paessler.com/prtg/features/sensors)

[16] [Nagios plugins](https://www.nagios.org/downloads/nagios-plugins/)

### Open terminology item

Verify all five official LetsDefend Threat Hunting Life Cycle stage names and descriptions from the original lesson before using the lifecycle section for lesson recall. The practical workflows are preserved exactly as requested. The team role matrix is added analyst guidance, not an official lesson list.

