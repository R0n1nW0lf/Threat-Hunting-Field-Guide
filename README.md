# Threat Hunting Field Guide

A practical reference for SOC analysts investigating endpoint, network, authentication, email, and DNS activity. Use it to choose evidence, build searches, test explanations, and document defensible findings.

**The tool provides evidence. The analyst decides what the evidence means.**

**START HERE:** [Step-by-Step Evidence Tracking: Evidence First](#step-by-step-evidence-tracking-evidence-first) — learn how to locate, narrow, correlate, and verify evidence.

## How to use this handbook

Choose a hunting question, identify the evidence that can answer it, test competing explanations, and record a defensible conclusion. Tool entries emphasize what question the tool answers and what to inspect next.

Filters are starting points. Confirm the search language, field mapping, logging coverage, and time range in the environment being investigated. Record both supporting and conflicting evidence.

## Quick Find

Use these links to jump directly to the section you need during a hunt.

| I am looking for... | Go to |
| --- | --- |
| Evidence-first investigation: step-by-step | [Step-by-Step Evidence Tracking: Evidence First](#step-by-step-evidence-tracking-evidence-first) |
| Threat hunting basics and proactive vs reactive | [Threat Hunting Introduction](#threat-hunting-introduction) |
| Adversary, hypothesis, or IoC hunting methods | [Threat Hunting Methodologies](#threat-hunting-methodologies) |
| Hunt lifecycle and workflow | [Threat Hunting Life Cycle](#threat-hunting-life-cycle) |
| Collection tools and telemetry | [Threat Hunting Tools and Data Collection](#threat-hunting-tools-and-data-collection) |
| Splunk, ELK, or LogRhythm | [Data Analysis Tools](#data-analysis-tools) |
| Wireshark, Snort, Zeek, or Vectra | [Network Monitoring Tools](#network-monitoring-tools) |
| PRTG, Nagios, or monitoring context | [Network and System Monitoring Context](#network-and-system-monitoring-context) |
| Firewall threat hunting: baselines and outbound hypothesis | [Threat Hunting with Firewalls: Outbound Connection Hypothesis](#threat-hunting-with-firewalls-outbound-connection-hypothesis) |
| Source country vs IP geolocation fields | [Country Field Comparison](#country-field-comparison) |
| SIEM source/destination field cheat sheet | [SIEM Field Quick Reference: Traffic Direction](#siem-field-quick-reference-traffic-direction) |
| Wazuh/OpenSearch fields and filters | [Wazuh / OpenSearch Threat Hunting Filter Field Guide](#wazuh--opensearch-threat-hunting-filter-field-guide) |
| A complete example hunt | [Worked Hunt from Hypothesis to Action](#worked-hunt-from-hypothesis-to-action) |
| Endpoint process, hash, DNS, and firewall correlation | [EDR and CTI Correlation: Process to Firewall](#edr-and-cti-correlation-process-to-firewall) |
| Email security: phishing, BEC, and investigation | [Threat Hunting with Email Security](#threat-hunting-with-email-security) |
| DNS security: attacks, detection, and investigation | [Threat Hunting with DNS](#threat-hunting-with-dns) |
| Hunt notes/report template | [Editable Hunt Worksheet](#editable-hunt-worksheet) |

[Back to top](#threat-hunting-field-guide)

## Threat Hunting Introduction

Threat Hunting is a proactive search for adversary activity that may have escaped existing detections. It starts with a reasoned question about the environment and uses available evidence to investigate it. A hunt may discover an incident, a benign explanation, or a gap in visibility.

| Approach | Starting point | Analyst question |
| --- | --- | --- |
| Reactive security | An alert, reported incident, or known problem | What happened, how serious is it, and what response is needed? |
| Proactive security | A hypothesis, TTP, intelligence lead, or unexplained behavior | Could this activity be occurring even though our current detections have not alerted? |

Reactive investigation can produce a proactive hunt: after one confirmed incident, search the wider environment for the same behavior. Proactive does not mean guessing without evidence, and a quiet alert queue does not establish that the environment is safe.

### Threat Hunting Team and Competencies

Assign responsibilities before the hunt begins. In a small team, one person may perform several roles.

| Role | Contribution to the hunt |
| --- | --- |
| Threat hunter or lead | Frames the question, scopes the hunt, tests explanations, and records conclusions. |
| Threat intelligence analyst | Supplies relevant adversary TTPs and IoCs with source, recency, and confidence. |
| Detection or data engineer | Checks telemetry, ingestion, parsing, retention, and repeatable detection logic. |
| SOC analyst and incident responder | Validates findings, investigates impact, and handles escalation and response. |
| System or network owner | Explains normal activity, maintenance, asset purpose, and operational constraints. |

Build competencies in network protocols; Windows and endpoint activity; log searching and correlation; TTPs and MITRE ATT&CK; threat intelligence evaluation; scripting; and clear reporting. Critical thinking connects them: ask what would disprove your explanation, and seek that evidence.

[Back to top](#threat-hunting-field-guide)

## Threat Hunting Methodologies

### Adversary-Centric Approach

Adversary-Centric hunting focuses on the adversary's Tactics, Techniques, and Procedures (TTPs). Use MITRE ATT&CK to organize observed behavior and identify what evidence your environment should contain. A tactic describes an objective, a technique describes a way to achieve it, and a procedure is a specific implementation.

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

These methods can work together. An IoC hit can seed a hypothesis, and a TTP hunt can reveal new indicators. Keep the starting rationale and later pivots in the record.

[Back to top](#threat-hunting-field-guide)

## Threat Hunting Life Cycle

Use a repeatable workflow to keep scope, evidence, decisions, and follow-up actions traceable. Revisit earlier steps when a new finding changes the question or reveals a visibility gap.

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

The reasoning workflow fits inside the broader hunt workflow. Hypothesis helps Plan; Collect Data supports Collect; Analyze and Test occur during Hunt; Conclusion belongs in Report; Action can begin response immediately and feed Improve.

Escalate credible evidence of active compromise promptly. Do not wait for the report to be finished. If required data is absent, record an inconclusive result and a collection improvement rather than treating the hypothesis as disproved.

[Back to top](#threat-hunting-field-guide)

## Threat Hunting Tools and Data Collection

Select tools by the evidence needed: endpoint activity, network observations, authentication records, intelligence context, or centralized search. Check how each source is collected and which parts of the environment it covers.

EDR contributes endpoint context and investigation or response capabilities. CTI supplies leads and context. Integration and automation connect sources, enrichment, searches, and repeatable tasks. Use these categories to choose evidence, not to assume each product has only one function.

| Tool | What question does it answer? | Evidence and next check |
| --- | --- | --- |
| Sysmon | What detailed activity occurred on this Windows host? | Process creation, network connections, and other configured events. Check command line, parent process, user, and timing. |
| Winlogbeat | How can Windows event logs reach our analysis platform? | Reads selected event logs and ships events to configured outputs. Verify the channels, filters, delivery, and parsed fields. |
| NXLog | How can we collect and forward logs from our sources? | Collection and forwarding agent with configured inputs and outputs. Check source selection, parsing, buffering, and delivery. |
| Graylog | Where can we centrally collect, organize, and search logs? | Central log management, parsing, and analysis. Confirm the expected sources arrive and remain searchable. |

Practical distinction: Sysmon generates telemetry. Winlogbeat and NXLog move selected logs. Graylog centralizes and makes them useful. Graylog also supports analysis; tool capabilities can overlap.

### A collection pipeline you can verify

Example: Windows activity → Sysmon events → Winlogbeat or NXLog → an analysis platform. Winlogbeat may send to Elasticsearch directly or through Logstash. Treat this as a design example; verify the actual deployment before relying on it.

- Generate one known benign event on a lab host and find it in the destination.

- Compare source timestamp, ingestion timestamp, hostname, event type, and key fields.

- Check retention and gaps. A shipper cannot recover events that were never logged.

[Back to top](#threat-hunting-field-guide)

## Data Analysis Tools

Analysis platforms help connect observations across time, hosts, users, and data sources. Their usefulness depends on what was collected and how it was parsed. A dashboard cannot establish the truth of a hypothesis without reviewing the underlying events.

### Splunk

Question: “What related events can I find across the collected data?” Use searches, statistics, timelines, and dashboards to identify patterns and pivot from an entity into related activity. For a suspicious destination, find contacting hosts, then correlate process and authentication events.

Confirm the index, source type, time zone, field extraction, and host coverage. Save the search and time boundaries. An empty search can result from an incorrect field or missing source, not just absent activity.

### ELK Stack components

ELK stands for Elasticsearch, Logstash, and Kibana. Beats, including Winlogbeat, are collection agents in the broader Elastic Stack.

| Component | What question does it answer? | Analyst use |
| --- | --- | --- |
| Elasticsearch | Where is the indexed data, and what records match? | Stores, indexes, and searches data. Confirm index selection and field types. |
| Logstash | How should incoming records be processed and routed? | Ingests, transforms, and forwards events. Check parsing and timestamp conversion. |
| Kibana | How can I explore and visualize the indexed events? | Searches and visualizes Elasticsearch data. Inspect raw events behind summaries. |
| Beats / Winlogbeat | How does source telemetry reach the stack? | Lightweight shipping of configured data. Verify inputs, outputs, and delivery. |

### LogRhythm

Question: “Which collected events form a suspicious sequence or relationship?” Use SIEM search and correlation to relate activity across sources and investigate alarms in context. For example, connect unusual authentication with subsequent host or network activity.

Inspect the raw events behind a correlated finding. Normalization, rule configuration, and available sources affect the result. A priority score or alarm is an investigative lead, not a final verdict.

### Make analysis repeatable

Start broad enough to understand the population, then narrow by entity and time. Count affected hosts, compare with peer systems, and inspect representative events. Record queries and exclusions so another analyst can reproduce the result. Use the search language and schema of the actual platform; field names are not interchangeable.

[Back to top](#threat-hunting-field-guide)

## Network Monitoring Tools

| Tool | Core distinction | What question does it answer? |
| --- | --- | --- |
| Wireshark | Dissect actual packets | What exactly is in this captured exchange? |
| Snort | Detect using rules/signatures | Does this traffic match a detection rule/signature? |
| Zeek | Record/describe network activity through detailed logs/events | What network activity occurred, and how do the connections relate? |
| Vectra NDR | Surface anomalous network behavior using AI/ML | Which network behavior looks abnormal or suspicious enough to investigate? |

### Wireshark

Narrow a packet capture by host, destination, protocol, and time. Examine protocol fields, requests and responses, retransmissions, or stream contents where visible. Capture location and encryption limit what you can see. A packet capture describes traffic at its observation point; it does not automatically identify the responsible process.

### Snort

Inspect the rule identifier, rule logic, direction, source and destination, and matching traffic. Correlate the alert with packet, host, and asset context. Snort can operate as IDS or IPS; blocking depends on deployment and configuration. No alert does not prove absence of malicious traffic.

### Zeek

Begin with conn.log for connection metadata; pivot to dns.log or http.log when available. Correlate by time, endpoints, and connection UID where applicable. Look for frequency, volume, unusual peers, and protocol behavior. Zeek logs summarize observations; they are not a replacement for full packet contents.

### Vectra NDR

Network Detection and Response findings highlight behavior for investigation: investigate the behavior explanation, affected entity, timing, and related activity; compare with approved operations. An AI/ML finding still needs corroboration and a defensible analyst conclusion.

[Back to top](#threat-hunting-field-guide)

## Network and System Monitoring Context

### PRTG Network Monitor

Use monitoring sensors to locate changes in traffic, availability, or application performance. Verify the sensor type and collection interval before interpreting an alert.

Question: “Something unusual is happening on the network. Where should I investigate?” Review the affected sensor, time, baseline, and device. A bandwidth change or availability problem can provide a useful starting point. Then collect security evidence that can explain it.

A spike may reflect backups, updates, congestion, or unauthorized transfer. PRTG can point to a condition worth investigating; individual packet dissection is the Wireshark question.

### Nagios

Nagios checks can expose service failures or resource changes that merit investigation. Confirm the configured plugin and threshold. Distinguish Nagios Core from other products when evaluating capabilities and licensing.

Question: “Something abnormal happened to a system or service. Does it have security significance?” Review the check result, state changes, duration, affected hosts, and maintenance context. Pivot to security logs when an outage or resource change has an unexplained cause. Plugins determine what a configured check can observe.

### Move from a condition to evidence

| Starting observation | Next evidence | Question to resolve |
| --- | --- | --- |
| Bandwidth spike | Connection logs, DNS, endpoint process data | Which hosts and processes explain the transfer? |
| Service unavailable | System, application, authentication, and change logs | Was this failure, maintenance, or unauthorized activity? |
| Snort rule match | Rule logic, packet detail, host context | Does the match represent the behavior we suspect? |
| Vectra behavioral finding | Related activity and independent telemetry | Is the behavior authorized and consistent with the asset's role? |

Anomalies establish a reason to investigate. They do not establish attacker intent. Align time zones and use the asset's actual role before connecting events into a narrative.

[Back to top](#threat-hunting-field-guide)

## Wazuh / OpenSearch Threat Hunting Filter Field Guide

These examples use field/value filters and query-string patterns. Select the matching search mode in Discover; DQL, Lucene query strings, and filter controls have different behavior. Confirm field types and ruleset-specific group values before combining conditions. See the [OpenSearch query-string reference](https://docs.opensearch.org/latest/query-dsl/full-text/query-string/).

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

Illustrative pivot: `data.srcip: 172.16.8.190` searches for activity attributed to that source address; inspect the records before deciding whether it represents lateral movement. An `agent.ip` such as `10.10.10.22` identifies the monitored destination system reporting that SSH attempt. Filtering only on `agent.ip: 172.16.8.190` can therefore hide evidence generated by the destination hosts.

### Build filters from the hunting question

Use this reasoning chain:

**Question → evidence type → source/destination → action/result → filter → inspect raw evidence**

Example: “Find failed SSH authentication originating from 172.16.8.190.”

1. Evidence type: SSH → `rule.groups: sshd`
2. Origin: 172.16.8.190 → `data.srcip: 172.16.8.190`
3. Result: failed authentication → `rule.groups: authentication_failed`
4. Inspect the matching events and `full_log`; do not treat the filter itself as the conclusion.

Practical rule: first understand what the question is asking, then identify which field represents each part of the question. Use the filtered results as evidence and verify the underlying event before reporting a conclusion.

### Correlating VPN, SSH, FTP, and Linux Audit evidence

**Translate question wording into stacked filters.** Break the question into evidence concepts, then map each concept to a field or rule group. For example, a service can map to a service-specific `rule.groups` value, a successful login to `authentication_success`, and an account name to `data.dstuser`. Stack only filters justified by the question, then inspect the remaining event.

**Use rule groups to narrow behavior before searching raw text.** Service and behavior groups such as `ftp`, `sshd`, `audit`, `authentication_success`, and `authentication_failed` can quickly reduce the event population. Remove filters that belong to a previous question when the new question changes the behavior being investigated.

**Do not confuse hit count with unique count or frequency.** A question asking for the “most frequent” value requires counting occurrences across the full result set. A question asking for “unique destinations” requires counting distinct destination values. Visible rows or total hits alone are not proof.

**Use `full_log` as a fallback for unparsed evidence.** Wazuh may parse many useful fields while leaving another needed value only in the original log. If the expected field is missing, inspect `full_log` before concluding that the evidence is unavailable. Raw logs may include numeric UIDs and account names. Verify the host account mapping rather than assuming an event supplied both.

**Correlate by time, not just by IP or username.** The same address or account can appear in unrelated activity elsewhere in the hunting window. Anchor pivots to the relevant session or event time and follow activity forward or backward from that point.

**Pivot across telemetry instead of expecting one log source to answer everything.** A practical chain can move from VPN authentication to a tunnel IP, then firewall traffic, SSH authentication, and finally endpoint Audit events. Each source answers a different part of the story.

**Understand endpoint-reported destinations.** In host-generated SSH events, `data.srcip` identifies the connecting system while `agent.ip` identifies the monitored endpoint whose SSH daemon reported the event. A destination does not always appear as `data.dstip`.

**Reconstruct Linux Audit commands from `execve` arguments.** `data.audit.command` may expose only the base command. Read `data.audit.execve.a0`, `a1`, `a2`, and later arguments in order to reconstruct the command line, identify command targets, or recover a full file path passed to an editor. For example, the arguments can distinguish running `ps` from searching its output for a particular process.

**Follow configuration changes into service activity.** When a question asks what happened before a service restart, use the restart as a timeline anchor and inspect preceding Audit activity on the same endpoint. An editor command with a configuration-file argument identifies an attempted interaction with that path. Confirm an actual change through file integrity, audit write records, or a configuration comparison, and correlate the account and session.

**Expand the correct event for enrichment.** Once the event is isolated, expanded fields can provide information not shown in the normal result row, such as IP geolocation. Treat enrichment as context associated with the event and keep the underlying source IP visible.

**SIEM value in threat hunting comes from collection, analysis, and correlation.** SIEM centralizes telemetry so hunters can connect user activity, network traffic, authentication, and endpoint evidence. SOAR complements SIEM by automating and orchestrating repeatable security workflows.

### Discover an unknown Windows Event ID from the evidence

When a hunting question points to a host or behavior but you do **not** know the Event ID, identify the provider, channel, and event content before choosing a numeric filter. Pivot from evidence you already proved.

**Method: known condition → agent/host → all host telemetry → enumerate Event IDs → inspect behavior → identify the matching Event ID**

1. Start with the event that established the condition, such as a system-monitoring alert.
2. Extract the affected hostname or agent name.
3. Remove the source-specific filter if it would hide other telemetry. For example, after identifying a host from a Zabbix alert, remove the Zabbix-only filter before investigating Windows events.
4. Search the affected host across the full hunting time window.
5. Open the available `data.win.system.eventID` field and enumerate the Event IDs present in the narrowed dataset.
6. Filter or inspect those Event IDs and read their underlying event output.
7. Match the event behavior to the hunting question. If the question asks what application was installed, look for an event whose data actually identifies an installation/application.
8. Once the matching event is found, record its Event ID and use that ID as a repeatable filter for similar telemetry.

Record the provider and channel along with the Event ID: numbers are not globally unique across Windows event providers. Confirm that the event describes the behavior being investigated rather than selecting an ID from memory.

Practical warning: a monitoring event may not contain the Windows Event ID at all. The monitoring alert is the starting condition; the endpoint telemetry is the next evidence source.

### Hunt embedded hash IOCs with wildcards

When a known hash IOC is visibly present in telemetry but exact SIEM searches return zero results, test whether the hash is embedded inside a larger searchable value.

If a bare hash or exact field match misses a known record, inspect the indexed representation and active search mode. A query-string pattern such as `*MD5=<hash>*` may help locate a hash embedded in a combined value; replace `<hash>` with a validated hexadecimal digest. This is a search pattern, not a known malicious indicator.

**Why this matters:** Sysmon hash telemetry can be stored as a larger combined value containing multiple hash types, such as `SHA1=...,MD5=...,SHA256=...,IMPHASH=...`. An exact lookup for only the MD5 may therefore fail. In a compatible query-string search, leading and trailing `*` wildcards can match surrounding characters. Analyzer settings, default search fields, and leading-wildcard restrictions affect results; narrow by host and time and verify the raw hash value.

Reusable hunting workflow:

**CTI hash IOC → identify hash type → search `*<TYPE>=<hash>*` → inspect matching event → pivot to `agent.name`, image/path, process, user, and surrounding telemetry**

Do not treat an exact-search miss as proof that an IOC is absent when the telemetry may store that IOC inside a larger field. Test the representation and wildcard search before concluding there is no match.

**Use hypotheses as testable starting points, not conclusions.** Begin with data analysis and preliminary review, use relevant telemetry to test the hypothesis, and revise or create a new hypothesis when the evidence does not validate it. Do not force evidence to fit the original idea.

**Use MITRE ATT&CK to describe behavior, then verify the underlying evidence.** ATT&CK Tactics, Techniques, and Procedures provide a common framework for hunting. A technique label such as Process Discovery is a useful pivot, but the audit or endpoint evidence should show what actually occurred.

[Back to top](#threat-hunting-field-guide)

## Worked Hunt from Hypothesis to Action

Scenario: a workstation appears to contact an external destination repeatedly. The activity might be unauthorized command and control, but software updates or a scheduled health check could produce a similar pattern. This is an illustrative scenario; conclusions depend on the observed evidence.

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

[Back to top](#threat-hunting-field-guide)

## EDR and CTI Correlation: Process to Firewall

Use these Wazuh/OpenSearch filters to reconstruct an endpoint incident from observed telemetry. Rule-group names depend on the installed ruleset; verify the event content rather than assuming a filter proves the behavior.

| Evidence question | Starting filter or field | Verification |
| --- | --- | --- |
| What process ran? | `rule.groups: sysmon_event1` (Sysmon Event ID 1) | `data.win.eventdata.image`, `commandLine`, `parentImage`, `processId` |
| Was PowerShell encoded? | `rule.groups: encoded_command` with `sysmon_event1` | Find `-EncodedCommand` in the actual command line and extract the complete encoded value |
| What IP did the process contact? | `rule.groups: sysmon_event3` (Sysmon Event ID 3), plus image/path | `data.win.eventdata.destinationIp`, `sourceIp`, process identity, timestamp |
| What domain did it resolve? | Sysmon Event ID 22 (DNS query) | `data.win.eventdata.queryName`, `image`, and `queryResults` |
| Did a firewall allow or deny traffic? | `rule.groups: firewall` and `data.dstip: <destination IP>` | `data.action`, `data.srcip`, `data.dstport`, `data.service`, time |

### Encoded PowerShell and CTI pivot

- Extract the value following `-EncodedCommand` from the complete process command line. Base64 is an encoding, not encryption. PowerShell's `-EncodedCommand` normally expects UTF-16LE bytes; verify the original bytes and decoded text rather than assuming every Base64 string is UTF-16LE.
- Read the decoded command to identify URLs, IPs, file paths, and intent. A download command is evidence of attempted download, not by itself proof of successful file creation or execution.
- Look up extracted indicators in Threat Intel. Record the IOC type, source, tag/group attribution, and confidence. Attribution from an IOC tag is not independent proof of the operator.
- Pivot from a threat-group IOC hash back into Sysmon process events. For a hash embedded in a combined `SHA1=...,MD5=...,SHA256=...` value, try `*SHA1=<hash>*` or `*MD5=<hash>*` in the Discover search bar. Confirm the matching event's `hashes` field before trusting it.
- In Sysmon Event 1, `image` identifies the created process; `parentImage` identifies the parent executable. `parentCommandLine` and process GUID/PID can help reconstruct the relationship.

### Correlate endpoint and firewall evidence

**Process create → hash match → parent process → Sysmon Event 3 destination → firewall destination filter → firewall action.**

- Sysmon Event 3 records network connection telemetry, but does not alone establish that the firewall permitted the traffic.
- Filtering firewall events by destination IP alone can return connections from *different* internal source hosts. Inspect **all relevant hits** and compare source IP, destination IP, time, port, and protocol before attributing an allow/deny decision to a specific process or endpoint.
- Hit count is not unique-host count. Do not conclude a particular connection was blocked solely because a firewall event to the same destination says `deny`.

[Back to top](#threat-hunting-field-guide)

## Step-by-Step Evidence Tracking: Evidence First

Use this method for unfamiliar threat-hunting questions. Build an investigation record that another analyst can reproduce from the same evidence.

1. **Translate the question into an evidence target.** Write down the behavior being investigated, the time window, any known host or IOC, and the exact output requested (IP, command, URL, group, executable, technique ID, or action). Separate confirmed facts from assumptions.
2. **Set the correct time range.** Check timezone and ensure the start and end cover the requested activity. A wrong time window can make good filters appear broken.
3. **Start with a confirmed anchor.** Search by a known `agent.ip`, `agent.name`, process, domain, IOC, or relevant event text. Do not begin with an unverified MITRE ID just because one sounds plausible.
4. **Explore before narrowing.** Inspect event counts, timestamps, `rule.description`, `rule.groups`, and available fields. Expand representative raw events; the top-values panel is not a complete list of values.
5. **Narrow one condition at a time.** Add a relevant process, keyword, field, or wildcard search; observe how the hit count changes. If zero results appear, remove the last constraint and reconsider the field, syntax, event source, or time range. Do not treat zero hits as proof the activity never occurred.
6. **Follow the behavior to the next evidence source.** Process creation can reveal `commandLine`, `image`, `originalFileName`, and parent process; DNS or network events can reveal contacted destinations; Windows security and firewall events can establish other actions. Not every source records every field.
7. **Discover and verify MITRE mappings.** First observe what the event actually describes. Read `rule.mitre.id` and `rule.mitre.technique` when available. If absent, research the behavior in MITRE ATT&CK as a **hypothesis**, then check whether the event supports it. Detection mappings can differ and may be missing.
8. **Correlate without conflating.** Compare host, account, PID, process lineage, timestamps, source/destination addresses, and actions across events. `agent.ip` is the reporting endpoint, not automatically the network source; a process execution or connection attempt alone does not prove success.
9. **Return to the exact requested field.** Distinguish `commandLine` (full invocation), `image` (executable path), and `originalFileName` (embedded original filename). Copy the recorded value exactly, including unexpected spellings, and verify it in the raw event.
10. **Report findings and uncertainty.** Record the finding with its supporting timestamp, host, field, and event. State what is observed versus inferred; do not claim intent, success, or attribution without corroboration. Preserve the exact searches, time boundaries, evidence references, and follow-up actions.

[Back to top](#threat-hunting-field-guide)

## Threat Hunting with Firewalls: Outbound Connection Hypothesis

Firewall logs provide source/destination, ports, protocols, timestamps, traffic volume, policy decisions, and patterns. Hunting begins by comparing observations with a baseline rather than assuming unusual traffic is malicious.

### Hypothesis

A specific internal IP address may suddenly generate a high number of outbound connection attempts to external IP addresses on unusual or rarely used ports. This is a testable hypothesis, **not a confirmed incident**.

### Baseline before judging

| Dimension | Establish normal behavior |
| --- | --- |
| Source and destination | Which hosts communicate, and with which external destinations? |
| Port and protocol | Which services and protocols are expected for this host? |
| Time and frequency | When and how often do connections normally occur? |
| Volume | What byte counts, session counts, and transfer sizes are typical? |
| Rules and actions | Which firewall policies usually allow or deny this traffic? |

### Firewall hunting workflow

1. Set the hypothesis, scope, time window, and time zone.
2. Collect firewall events; use centralized SIEM search and network flow, IDS/IPS, or endpoint data where available.
3. Filter for outbound traffic from internal source IPs to external destinations.
4. Compare connection counts, destination diversity, ports, timing, and volume against the baseline.
5. Review firewall action and policy, and identify unusual destinations and ports.
6. Pivot on the source host, destination IPs, timestamps, and relevant CTI indicators.
7. Correlate with flow, DNS, process, authentication, IDS/IPS, and other security evidence as applicable.
8. Test benign explanations, record supporting and conflicting evidence, and report the outcome with confidence and limitations.

### Pattern interpretation

| Observation | Possible lead, not proof |
| --- | --- |
| One host to many external IPs on the same port | Scanning, discovery, or legitimate automated services |
| One host to one external IP on many ports | Possible port scan or troubleshooting |
| Regularly spaced outbound attempts | Possible C2 beaconing or scheduled application activity |
| Unusually large outbound transfers | Possible exfiltration or legitimate backup |
| Numerous denied connections | Blocked attempts, misconfiguration, or unwanted software |
| Unexpected outbound RDP (TCP/3389) | Remote access worth validating against policy |

**Evidence safeguards:** An `allow` decision means the firewall permitted traffic, not necessarily that the session completed. A `deny` or `drop` indicates enforcement at the observed control, not proof of compromise. Firewall connection attempts are not the same as failed authentication attempts; consult service/authentication logs for login outcomes. Field names differ by firewall vendor and parser. Check timestamp consistency, direction, NAT, sensor visibility, and raw events before concluding.

**Related field lookup:** [SIEM Field Quick Reference: Traffic Direction](#siem-field-quick-reference-traffic-direction).

[Back to top](#threat-hunting-field-guide)

## SIEM Field Quick Reference: Traffic Direction

Keep this table on a dedicated notes screen during investigations. These examples reflect Wazuh-style parsed fields; field availability varies by event source.

| Field | Meaning | Quick question |
| --- | --- | --- |
| `data.srcip` | Source IP in a network event | Where did this traffic come from? |
| `data.dstip` | Destination IP in a network event | Where was this traffic going? |
| `agent.ip` | IP of the reporting/monitored agent | Which endpoint reported this event? |
| `data.srcport` | Source port | Which port did the sender use? |
| `data.dstport` | Destination port | Which service/port was targeted? |
| `data.action` | Logged security-device action | Was the traffic allowed, denied, or dropped? |
| `data.win.eventdata.queryName` | Domain requested in a Sysmon DNS query event | Which domain was queried? |
| `data.win.eventdata.image` | Executable path in relevant Sysmon events | Which process was involved? |

### Country Field Comparison

| Field | Where the value comes from | How to use it |
| --- | --- | --- |
| `GeoLocation.country_name` | IP geolocation enrichment (verify which IP the enrichment applies to) | Filter or examine geographic context when the enrichment refers to the source IP. |
| `data.srccountry` | Source-country value parsed from the firewall event | Filter by the country the firewall log attributes to the source IP. |

**Same purpose, different provenance:** Both may indicate the country associated with incoming traffic, but they are not interchangeable in every dataset. A generic geolocation field may refer to a different IP, and geolocation databases or firewall mappings can disagree. Verify the field's actual value and meaning in an expanded event; check `data.srcip`, raw `full_log`, and the log source. Country attribution is not proof of an attacker's physical location or malicious intent.

**Filter pattern:** `data.srccountry: <country>` selects a source-country value when that field is present. Replace the placeholder with the recorded value and quote values containing spaces according to the active search language.

**Direction example:** `192.168.1.10 → 203.0.113.20` means `data.srcip: 192.168.1.10` and `data.dstip: 203.0.113.20` in a network event with those fields.

- **Outbound to a suspicious IP:** start with `data.dstip`; verify the internal `data.srcip`, time, port, and action.
- **Inbound from a suspicious IP:** start with `data.srcip`; verify the internal `data.dstip`, time, port, and action.
- **Endpoint event:** `agent.ip` identifies the reporting endpoint and is **not automatically** the connection source or destination.
- **Evidence caution:** `allow` does not prove a completed session or compromise; `deny`/`dropped` describe the observed enforcement action. Correlate independent events before concluding success.
- **Search caution:** Some event sources use different field names or omit destination fields. Inspect the expanded raw event and confirm the time window, log source, and direction.

[Back to top](#threat-hunting-field-guide)

## Threat Hunting with Email Security

Email investigations begin with a question: **Did this message reach anyone, and did anything suspicious happen afterward?** Work from the message record outward. Do not label an endpoint compromised merely because a suspicious email exists.

### What to examine

| Question | Evidence to inspect |
| --- | --- |
| Who sent it, and who received it? | Sender address and domain, recipients, message timestamp, headers, and authentication results |
| Did the security gateway stop it? | Disposition or final action: blocked, quarantined, or allowed |
| What was the lure? | Subject, URLs, attachments, requested action, and impersonated organization |
| Did activity follow delivery? | Endpoint DNS queries, browser or mail-client events, process creation, network connections, and relevant account activity |
| Could the sender or account be impersonated or compromised? | Authentication patterns, mailbox rules, unusual payment requests, unexpected recipients, and changed communication behavior |

### Investigation workflow

1. **Define the scope.** Select the affected mailbox or domain and exact investigation window. Confirm the environment before counting matches.
2. **Review the message.** Inspect the sender domain, recipient, final action, URLs, attachments, and available authentication checks. Similar-looking domains deserve investigation, not automatic conviction.
3. **Separate gateway outcomes from user actions.** A message marked allowed may have reached a mailbox, but that does not establish that a person opened it, followed a link, or ran a file.
4. **Pivot to the endpoint.** Search for DNS queries involving the suspicious domain, then determine the host and originating process. Check process creation and network events around the same time.
5. **Correlate the network path.** Compare DNS results with firewall destination IP, source IP, port, action, and timestamps. Inspect every relevant event rather than attributing an unrelated connection to the recipient.
6. **Enrich and validate.** Use threat intelligence as a lead. Match file hashes against the *actual recorded endpoint hashes* before claiming the executable matches a known indicator.
7. **Report what is supported.** Document whether the email was blocked or allowed, which hosts showed related behavior, the strongest evidence, alternative explanations, and remaining visibility gaps.

### Patterns and interpretation

| Observation | Analyst interpretation |
| --- | --- |
| Lookalike sender domain | Potential impersonation or phishing; inspect full message context |
| High outbound email volume | Possible compromised account or authorized bulk sending; compare to baseline |
| Unexpected mailbox forwarding rule | Possible persistence or legitimate configuration; confirm ownership |
| Mail-client process from unusual path | Investigative lead; validate publisher, signature, hash, and process lineage |
| DNS request to a domain in the email | Lookup occurred; it does not by itself establish a click |
| Firewall action `allow` | Traffic was permitted at that control; not proof of malware execution |
| Intelligence tag on a domain, IP, or hash | Indicator association; not independent proof of attacker identity |

**Rule of evidence:** Received, allowed, opened, clicked, downloaded, and executed are different stages. Verify each transition independently before reporting it as fact.

[Back to top](#threat-hunting-field-guide)

## Threat Hunting with DNS

Start with two questions: **Which process requested this name, and what happened after the lookup?** Resolver logs explain DNS exchanges; endpoint telemetry helps connect them to a host and process. Establish the observation point before counting clients or attributing activity.

### What to examine

| Question | Evidence to inspect |
| --- | --- |
| Who made the request? | Client address, resolver address, host identity, process image, and process GUID where available |
| What was requested? | Full query name, record type, query time, and parent domain |
| What answer was observed? | Response code, returned records, TTL, and answer time from resolver or packet data |
| Is the pattern unusual for this asset? | Rate, unique names, label lengths, response sizes, failure ratio, and peer-host baseline |
| Did communication follow? | Endpoint network events, proxy records, firewall sessions, and process creation |
| Did resolution settings change? | Resolver configuration, hosts-file changes, administrative activity, and change approvals |

### Investigation workflow

1. **Scope the observation.** Select hosts, resolvers, and time boundaries. Determine whether a logged source is an endpoint or a forwarding resolver; account for NAT and clock differences.
2. **Validate collection.** Identify which sources contain queries, responses, or both. Caches and encrypted DNS can limit resolver visibility. Missing resolver records do not establish that no lookup occurred.
3. **Inspect a representative event.** For Sysmon DNS telemetry, use `data.win.system.eventID: 22` together with the Sysmon provider/channel or equivalent source constraint. Review `data.win.eventdata.queryName`, `data.win.eventdata.queryResults`, `data.win.eventdata.image`, and `agent.ip`. Parsed names depend on the pipeline. Sysmon Event ID 22 can record successful, failed, and cached queries; it is not a complete DNS packet record. [Sysmon event reference](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).
4. **Compare with a baseline.** Group by client, process, domain, and time interval. Inspect full names as well as parent-domain totals. Separate repeated queries from distinct queried names; compare against the asset's role and similar hosts.
5. **Test competing explanations.** Long labels, frequent TXT queries, or many failures warrant inspection, but CDNs, security products, service discovery, and misconfiguration can produce unusual patterns. Examine content, timing, process lineage, and domain ownership context.
6. **Correlate subsequent activity.** Compare answers valid at the event time with Sysmon Event ID 3, proxy, and firewall records. Join on host, process GUID where available, time, destination, port, and protocol. Shared hosting and changing answers make an IP-only match weak evidence.
7. **Report the supported finding.** State whether the evidence establishes a lookup, a connection, or suspected abuse. Preserve queries and raw events, document visibility gaps, and assign response or collection work.

### Patterns and interpretation

| Observation | Analyst interpretation and next check |
| --- | --- |
| Many changing subdomains with long or encoded-looking labels | Possible tunneling or automated application traffic; inspect label content, process, rate, and responses |
| Frequent failures across generated-looking names | Possible domain generation or broken software; check response codes, process lineage, and peer activity |
| Large responses disproportionate to requests | Possible amplification in the right network context; verify packet direction, spoofing evidence, and resolver exposure |
| Unexpected answers or resolver changes | Possible poisoning, hijacking, or authorized change; compare trusted observations, configuration history, and administrative records |
| A name changes from a public to a private address | Possible rebinding or legitimate split DNS; verify the answer sequence and subsequent application requests |
| Lookalike domain | Possible impersonation; correlate email or browser activity and domain context |

### Investigating possible DNS exfiltration

Test whether query labels plausibly carry data, whether the pattern repeats under a consistent domain, and which process generates it. Preserve full names and responses. Compare throughput and timing with ordinary application traffic and look for preceding file access or staging. Entropy, label length, or a tunneling alert alone cannot establish that sensitive information left the host. Record what data, if any, can actually be demonstrated in the observed requests and who controlled the destination only when supported.

**Rule of evidence:** A lookup does not establish a user click, successful connection, malware execution, or exfiltration. A security product's block records enforcement at that control; check the host and other resolution paths before claiming the activity was fully prevented.

[Back to top](#threat-hunting-field-guide)

## Editable Hunt Worksheet

Copy this page for each hunt. Replace the prompts with actual evidence and decisions. Preserve uncertainty and the original time boundaries.

### Hunt definition

Hunt ID and title: ______________________________________________<br>
Analyst and owner: ______________________________________________<br>
Date and time zone: ______________________________________________<br>

Hypothesis or starting IoC/TTP: ___________________________________<br>
Reason for the hunt and intelligence source: ________________________<br>
Systems and time window in scope: _________________________________<br>
Out of scope and stop conditions: __________________________________<br>

### Data and coverage

Sources and collection settings: __________________________________<br>
Retention and sensor/host coverage: _________________________________<br>
Known gaps and timestamp checks: __________________________________<br>

### Investigation record

| Time and query or pivot | Evidence reference | Interpretation and next test |
| --- | --- | --- |
| Enter exact query and time range | Record or file location | Separate observation from inference |
|  |  |  |
|  |  |  |

### Conclusion and action

Outcome: supported / benign explanation supported / unsupported within scope / inconclusive<br>
Confidence and reason: ___________________________________________<br>
Strongest supporting evidence: ____________________________________<br>
Competing explanation and test: ___________________________________<br>
Limitations: ____________________________________________________<br>
Action, owner, and due date: _______________________________________<br>
Detection or collection improvement: _______________________________<br>

### Before closing the hunt

- Can another analyst reproduce the search and locate the evidence?

- Does the conclusion respect the coverage, time window, and uncertainty?

- Are the response, improvement, and follow-up actions assigned?
