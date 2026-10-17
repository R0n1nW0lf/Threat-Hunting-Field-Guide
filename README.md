# Threat Hunting Field Guide

A practical reference for SOC analysts investigating endpoint, network, authentication, email, and DNS activity. Use it to choose evidence, build searches, test explanations, and document defensible findings.

**The tool provides evidence. The analyst decides what the evidence means.**

**QUICK FIND:** [SIEM / Wazuh / OpenSearch Filter Cheat Sheet](#siem--wazuh--opensearch-filter-cheat-sheet) · [All handbook sections](#quick-find)

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
| SIEM/Wazuh/OpenSearch filter cheat sheet: question → field → example → verification | [SIEM / Wazuh / OpenSearch Filter Cheat Sheet](#siem--wazuh--opensearch-filter-cheat-sheet) |
| A complete example hunt | [Worked Hunt from Hypothesis to Action](#worked-hunt-from-hypothesis-to-action) |
| Endpoint process, hash, DNS, and firewall correlation | [EDR and CTI Correlation: Process to Firewall](#edr-and-cti-correlation-process-to-firewall) |
| Email security: phishing, BEC, and investigation | [Threat Hunting with Email Security](#threat-hunting-with-email-security) |
| Email-to-endpoint correlation and DDEI/Sysmon field examples | [Email-to-endpoint correlation: practical search patterns](#email-to-endpoint-correlation-practical-search-patterns) |
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

<a id="wazuh--opensearch-threat-hunting-filter-field-guide"></a>

## SIEM / Wazuh / OpenSearch Filter Cheat Sheet

**Question → source → confirmed field → filter → raw-event verification → correlation.**

This cheat sheet is a starting point for investigations, not a universal schema. Wazuh alert metadata, decoder output, vendor logs, and normalized SIEM documents have different fields. Every example assumes that the named field and sample value exist in the selected data. Addresses, hostnames, accounts, and signatures below are illustrative; replace them with observed values.

### Establish source and field availability

1. Select the index pattern/data view and a time range containing a known event. Wazuh alerts are detections, not a complete record of all collected activity. Archives require separate collection and indexing configuration; inventory/state indices answer different questions.
2. Expand representative events from **each** source. Record the index, timestamp, `agent.id/name/ip`, `location`, `decoder.name`, `predecoder.program_name`, and rule metadata where present.
3. Check the mapping: keyword versus analyzed text, numeric versus string, date versus text, and IP versus keyword. A field visible in JSON may be unindexed or unavailable in another index. Do not add `.keyword` unless that subfield exists.
4. Confirm exact spelling, capitalization, group membership, and sample values. `rule.groups` is plural and usually an array; `rule.group` is not a substitute. Groups describe rule classifications, not necessarily vendor product names.
5. Use a known record to test each filter before combining conditions. Decoder/rule testing with [wazuh-logtest](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html) checks extraction and matching; it does not verify downstream indexing or enrichment.

| Source | Fields to inspect when present | Availability boundary |
| --- | --- | --- |
| Wazuh alerts | `rule.groups/description/id/level`, `agent.*`, `decoder.name`, `location`, `full_log` | Metadata describes the alert and reporting context; manager-local events may use agent ID `000`. |
| Firewall, WAF, IPS/IDS | `data.srcip/dstip/srcport/dstport/action`; vendor-specific fields | Decoder-dependent. ECS pipelines may instead use `source.ip`, `destination.ip`, `event.action`; verify rather than mixing schemas. |
| Windows via Wazuh EventChannel | `data.win.system.*`, `data.win.eventdata.*` | Provider, channel, event type, and collection settings determine payload fields. |
| Linux service/syslog | `predecoder.program_name`, `data.srcip`, `data.dstuser`, rule groups | SSH, VPN, and FTP parsers need not expose the same account or result fields. |
| Linux Audit | `data.audit.*` | Requires audit rules, collection, and compatible decoding; argument coverage varies. |
| Package/inventory data | Package-manager messages or inventory/state fields | Inventory presence is different from a historical installation event. |
| Enrichment | `GeoLocation.country_name`, source-specific country fields | Optional pipeline output; determine the enriched IP and database provenance. |

### Discover search mode and syntax

Read the language selector before pasting a query. OpenSearch Discover offers DQL and Lucene query-string modes; Kibana KQL, Splunk SPL, SQL/PPL, and JSON Query DSL are separate languages. Filter controls are useful for selecting recorded values and avoiding escaping mistakes. Search-bar filters do not perform joins or calculate distinct counts.

| Intent | DQL | Lucene query string |
| --- | --- | --- |
| Match a group | `rule.groups: sshd` | `rule.groups:sshd` |
| Require two conditions | `rule.groups: sshd AND data.srcip: "192.0.2.10"` | `rule.groups:sshd AND data.srcip:"192.0.2.10"` |
| Either group | `rule.groups: (sshd OR firewall)` | `rule.groups:(sshd OR firewall)` |
| Field exists | `data.srcip:*` | `_exists_:data.srcip` |
| Present but not a value | `data.action:* AND NOT data.action: deny` | `_exists_:data.action AND NOT data.action:deny` |
| Numeric threshold | `rule.level >= 10` | `rule.level:[10 TO *]` |
| Prefix term | `agent.name: web*` | `agent.name:web*` |
| Absolute date interval, if time field is `timestamp` | `timestamp >= "2026-01-01T00:00:00Z" AND timestamp < "2026-01-02T00:00:00Z"` | `timestamp:[2026-01-01T00:00:00Z TO 2026-01-02T00:00:00Z}` |

Use explicit Boolean operators and parentheses. Negation alone can include records missing the field. Quoted text on an analyzed field is a phrase search, not byte-for-byte equality; use a confirmed keyword field or appropriate exact-value filter when required. Escape reserved characters in paths, URLs, and combined hash values. Wildcards inside quoted phrases are not wildcard searches. DQL supports `*`; Lucene also supports `?`, with different capabilities and restrictions. Check version/settings and scope expensive leading-wildcard searches. See the [official DQL and Lucene comparison](https://docs.opensearch.org/latest/dashboards/dql/).

**Examples in the following tables use DQL unless explicitly labeled Lucene.** Where a row requires a source constraint, add the source/decoder/location filter established from your own records.

### Rule, host, and raw evidence questions

| Analyst question | Field/filter | Example | Verification |
| --- | --- | --- | --- |
| Which category should I start with? | `rule.groups` | `rule.groups: sshd` | Inspect the array and description; installed rules determine group values. |
| What did the detection describe? | `rule.description` | `rule.description: "authentication failed"` | Read the complete description and raw event; wording and analysis affect matches. |
| Which events share this detection? | `rule.id` | `rule.id: "5710"` | Use an ID from a known event and inspect its installed rule; IDs are Wazuh detections, not vendor signatures. |
| Which alerts merit priority review? | `rule.level` | `rule.level >= 10` | Confirm numeric mapping; severity is a triage aid, not proof of compromise. |
| Which monitored endpoint reported it? | `agent.id/name/ip` | `agent.id: "007"` or `agent.name: "web01"` | Check asset identity and address history; the agent may collect logs for another device. |
| Which application or collector produced it? | `predecoder.program_name`, `decoder.name`, `location` | `predecoder.program_name: sshd` | Compare original header, decoder, and collection path; predecoder fields may be absent for JSON/EventChannel logs. |
| What detail was not parsed? | `full_log` | `full_log: "Failed password"` | Inspect the complete message and source record. Text search is a lead; `full_log` may be omitted or not searchable. |
| Which ATT&CK mapping accompanies it? | `rule.mitre.id/technique/tactic` when present | `rule.mitre.id: "T1059.001"` | Validate actual behavior against the mapping; absence of a tag does not exclude the behavior. |

**Critical distinction:** `agent.ip` identifies reporting context; `data.srcip` and `data.dstip` identify addresses decoded from the event. In an SSH log, the monitored server can be the destination even when no `data.dstip` exists.

### Network, WAF, IPS/IDS, firewall, and country questions

| Analyst question | Field/filter | Example | Verification |
| --- | --- | --- | --- |
| Where did this activity originate? | `data.srcip` if decoded | `data.srcip: "192.0.2.10"` | Validate observation point, NAT/proxy context, and raw source address. |
| Which destination was contacted? | `data.dstip` if decoded | `data.dstip: "203.0.113.20"` | Identify the source host too; many hosts can share a destination. |
| Which service/port was targeted? | `data.dstport`; `data.srcport` for sender port | `data.dstport: 22 AND data.srcip: "192.0.2.10"` | Check field type, protocol, and direction; a port is not an authentication outcome. |
| What did the firewall decide? | Confirmed firewall source plus `data.action` | `rule.groups: firewall AND data.action: (deny OR dropped)` | Use actual action values and policy/session context. Allowed traffic need not complete; a drop applies at that control. |
| What WAF attack category was reported? | **Conditional** `data.attacktype` or actual parser field | `data.attacktype: "SQL Injection"` | First inspect a WAF event showing this exact key/value. Check request URI, method, rule match, action, and application response. |
| Which WAF/IPS signature repeated? | **Conditional** `data.attackid` or vendor signature field | `data.attackid: "12345"` | Verify product, parser, signature name/version, and raw ID. This is not a Wazuh `rule.id` or MITRE technique ID. |
| Which IDS alerts concern this peer? | Confirmed IDS source; signature and tuple fields | `data.srcip: "192.0.2.10" AND data.dstip: "203.0.113.20"` plus source filter | Inspect signature logic, ports, protocol, sensor placement, and packet/flow evidence; alert does not imply prevention. |
| Did an IPS enforce a block? | Confirmed IPS source and recorded action | `data.action: dropped` plus IPS source filter | Check inline enforcement and matching session. Alert-only deployments do not block. |
| What country did the device assign to the source? | Conditional `data.srccountry` | `data.srccountry: "United States"` | Compare raw device record and source IP; use `data.dstcountry` only if actually parsed and needed. |
| What country did enrichment assign? | Conditional `GeoLocation.country_name` | `GeoLocation.country_name: "United States"` | Verify which IP was enriched, database age, and disagreement with device values; country is not actor location. |

**WAF parser checkpoint:** `data.attacktype` and `data.attackid` are examples of possible decoded keys, not standard fields guaranteed for every WAF. A vendor can log an attack ID without the installed decoder extracting it. Inspect both the raw message and indexed document, test the installed decoder, then use the actual searchable path. Do not substitute `data.attack`, a nested signature path, or an ECS field without checking it. The [Wazuh FortiGate decoder source](https://github.com/wazuh/wazuh/blob/master/ruleset/decoders/0100-fortigate_decoders.xml) illustrates how extraction depends on message format; compare against the deployed version.

### Windows process, DNS, network, and hash questions

For every Sysmon row, constrain the source to the verified Sysmon provider/channel, for example `data.win.system.providerName: "Microsoft-Windows-Sysmon"` when that path exists. Event IDs are provider-specific. Confirm collection and configuration before interpreting a missing event.

| Analyst question | Field/filter | Example | Verification |
| --- | --- | --- | --- |
| What process was created? | Sysmon Event 1; `image`, `commandLine`, parent and user fields | `data.win.system.eventID: "1" AND agent.name: "web01"` plus Sysmon source | Read `data.win.eventdata.image/commandLine/parentImage/parentCommandLine/user`; preserve paths and arguments. |
| Did a process use an encoded command? | Event 1 and command line | `data.win.eventdata.commandLine: *EncodedCommand*` plus Event 1/source | Inspect complete invocation; decode recorded bytes safely, then corroborate effects. |
| Which process requested a domain? | Sysmon Event 22; `queryName` | `data.win.system.eventID: "22" AND data.win.eventdata.queryName: "example.org"` plus Sysmon source | Inspect `image`, `processGuid`, `queryStatus`, and `queryResults` when available; lookup is not proof of a click or connection. |
| Which subdomains were queried? | Event 22, domain field | `data.win.eventdata.queryName: *.example.org` plus Event 22/source | Review full names; search apex `example.org` separately. Check case, trailing dot, mapping, and process. |
| Which IP/port did a process contact? | Sysmon Event 3; endpoint-specific address fields | `data.win.system.eventID: "3" AND data.win.eventdata.destinationIp: "203.0.113.20"` plus Sysmon source | Inspect `sourceIp/destinationIp/sourcePort/destinationPort/protocol/initiated/image/processGuid`. Network-event configuration affects visibility. |
| What connects DNS, process, and network events? | `agent.id` plus `data.win.eventdata.processGuid` | `agent.id: "007" AND data.win.eventdata.processGuid: "<observed-guid>"` | Remove the Event ID constraint to view the sequence. Prefer GUID and host over PID alone; PIDs can be reused. |
| Which executable hash was recorded? | Event 1; conditional `data.win.eventdata.hashes` | `data.win.eventdata.hashes:*` plus Event 1/source | Read algorithm labels and complete digests; configured algorithms and event type determine availability. |
| Which FIM record matches a digest? | Conditional `syscheck.sha256_after` or actual hash field | `syscheck.sha256_after: "<complete-sha256>"` | Confirm algorithm, before/after meaning, path, and event time. A FIM match is not evidence of execution. |

See [Microsoft's Sysmon event reference](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) for event semantics and configuration.

### Authentication, Linux Audit, and package questions

| Analyst question | Field/filter | Example | Verification |
| --- | --- | --- | --- |
| Which SSH logins failed from this IP? | Verified `sshd` and failure groups plus origin | `rule.groups: sshd AND rule.groups: authentication_failed AND data.srcip: "192.0.2.10"` | Inspect `full_log`, target account, method, host, and timestamps; distinguish invalid user from bad credentials. |
| Did a successful SSH login follow? | Verified success group, host, origin/account | `rule.groups: sshd AND rule.groups: authentication_success AND agent.name: "linux01"` | Confirm accepted authentication and session timing; an earlier failure and later success may involve different users. |
| Which account was targeted? | Conditional `data.dstuser` or source's user field | `data.dstuser: "analyst"` plus authentication source | Validate account role, realm, and raw spelling; not every parser uses `dstuser`. |
| Who connected over VPN? | Actual VPN decoder/source; user, outcome, session, assigned address | `data.srcip: "192.0.2.10"` plus verified VPN source | Inspect external peer versus tunnel-assigned IP, account, MFA result, session ID, connect/disconnect times. There is no universal VPN field/group. |
| Which FTP authentications failed or succeeded? | Actual FTP source and verified outcome groups | `rule.groups: ftp AND rule.groups: authentication_failed` **only if both groups occur** | Confirm daemon, account, client address, raw response, and session; firewall TCP/21 traffic does not prove a login. |
| What Linux command ran? | Conditional `data.audit.command/exe` | `data.audit.exe: "/usr/bin/id"` plus Audit source | Inspect syscall success/exit and records from the same audit event; command spelling alone is insufficient. |
| What arguments and targets were used? | `data.audit.execve.a0/a1/a2` and later args | `data.audit.execve.a0: "nmap"` when recorded | Reconstruct all available arguments in order; check argc, encoding, truncation, and related CWD/PATH records. |
| Which identity executed it? | Conditional `data.audit.auid/uid/euid` and session fields | `data.audit.auid: "1001"` when parsed | Map numeric IDs on that host at that time. Login identity, real UID, and effective UID have different meanings. |
| Was a configuration file changed before restart? | Same host/time; Audit execution/write records and FIM | `agent.name: "linux01" AND data.audit.execve.a1: "/etc/ssh/sshd_config"` when present | An editor argument indicates interaction, not a verified write. Confirm content change and service restart separately. |
| Was a package installed? | Actual yum/dnf/apt/dpkg logs; verified rule/program fields | `predecoder.program_name: yum` or `rule.groups: yum` if observed | Read transaction operation, package/version, outcome, user, and time; other package managers need their own source filters. |
| Was an installation merely attempted? | Audit command execution versus package transaction outcome | `data.audit.execve.a0: "apt"` if present | Compare arguments and return status with package-manager logs. Inventory presence alone does not establish install time or actor. |

Audit coverage depends on configured rules and collection. Consult [Wazuh's Audit collection guidance](https://documentation.wazuh.com/current/user-manual/capabilities/system-calls-monitoring/audit-configuration.html). Related records can be split or combined by the pipeline; correlate host plus audit serial/session and time where available.

### IOC and wildcard pivots

| Analyst question | Field/filter | Example | Verification |
| --- | --- | --- | --- |
| Where did an IP IOC appear? | Source and destination paths for each source | `data.srcip: "203.0.113.20" OR data.dstip: "203.0.113.20"` | Run equivalent Sysmon/ECS searches separately when schemas differ; classify origin versus destination. |
| Where did a domain IOC appear? | DNS name, proxy host/URL, email URL fields actually present | `data.win.eventdata.queryName: "example.org"` plus Sysmon source | Check full name and observation type; domain resolution, delivery, and browsing are different actions. |
| Is a hash embedded in a combined field? | Confirmed searchable combined hash field | **Lucene:** `data.win.eventdata.hashes:*SHA256\=<complete-sha256>*` | Replace placeholder with full digest. Escape literal `=` in Lucene; use an existing keyword representation for substring matching when possible. |
| Can raw text locate a known record? | `full_log` scoped to source/host/time | `full_log: *<complete-hex-digest>*` after validating wildcard support | Compare algorithm and full digest in raw record. Analyzed text, leading-wildcard settings, or omitted raw logs can cause misses. |

Start with exact algorithm-specific fields when available. Use wildcards to discover representation problems, then confirm the complete value; a prefix or substring match is not a verified IOC match. Combined hashes may include SHA1, MD5, SHA256, or other configured algorithms. Keep algorithm, intelligence provenance, and observation time with the finding. Avoid bare all-field wildcard searches as a default; narrow the population and field first. See [OpenSearch query-string behavior](https://docs.opensearch.org/latest/query-dsl/full-text/query-string/) for escaping and wildcard controls.

### Time scopes, counts, and unique values

| Analyst question | Field/filter or analysis | Example | Verification |
| --- | --- | --- | --- |
| What happened during the incident? | Discover time picker using confirmed date field | Absolute UTC window: 2026-01-01 00:00:00 to 2026-01-02 00:00:00 | Record displayed timezone, boundary inclusion, and chosen field. Wazuh `timestamp`, source event time, and `@timestamp` are not automatically equivalent. |
| What happened around an anchor event? | Same entity; widen before/after timestamp | Host plus a window five minutes before and after the event | Allow for clock skew, ingestion delay, and long sessions; justify window size. |
| How many records matched? | Total document count under saved filters/time | Count failed-authentication alert documents | Check result caps, duplicate ingestion, and aggregated alerts; document count is not always attempt/session count. |
| Which value occurred most often? | Terms aggregation with document counts | Group failures by confirmed source-IP field | Top values can omit the long tail. Increase limits/check other buckets; define tie handling and denominator. |
| How many distinct destinations/hosts/users? | Cardinality or complete deduplicated export | Unique `data.dstip` among this source's records | Cardinality can be approximate; use exhaustive deduplication when exactness matters. Missing fields do not become known entities. |
| Is the rate unusual? | Date histogram and relevant entity grouping | Failures per five-minute interval, compared with prior comparable periods | Normalize for coverage, interval, host role, and working hours; separate document rate from true activity rate. |

If results unexpectedly disappear, clear stale pinned filters, confirm index selection and retention, widen time, and test a known record. Remove one constraint at a time. An empty search supports only “no matching indexed records under these conditions,” not “the activity never occurred.”

### Cross-source correlations and verification

Discover filters select documents; they do not automatically join related events. Build a timeline using stable entities and independent checks.

| Analyst question | Pivot/filter | Example sequence | Verification |
| --- | --- | --- | --- |
| Did a VPN session lead to internal activity? | Account, assigned tunnel IP, session interval | VPN success → assigned IP → firewall tuple → SSH login → Audit activity | Confirm address lease at event time, NAT, target host, account/session, and clock alignment. |
| Did a process resolve and contact a destination? | Host + process GUID, then time and network tuple | Event 1 → Event 22 → Event 3 → firewall session | Compare DNS answer valid then, host addresses, protocol, ports, and direction; shared IP alone is weak linkage. |
| Did an email lead to endpoint execution? | Recipient/host mapping, domain/hash, narrow timeline | Delivery → DNS/browser evidence → file/hash → process creation | Verify each transition independently; delivery or lookup does not establish a click or execution. |
| Did a WAF/IDS alert correspond to application impact? | Request/session ID if present; tuple, URL, time | Signature alert → enforcement action → application/access logs → endpoint activity | Match the same request/flow and inspect response or host effects; a signature label does not prove successful exploitation. |
| Did a login precede installation or changes? | Host, account/session, time | Authentication → Audit command → package transaction/FIM → service activity | Account names alone do not establish causality; confirm outcomes and approved change context. |

Preserve useful investigative discipline: discover unknown Windows Event IDs by enumerating host telemetry and inspecting provider/channel/content; remove the original source filter when pivoting to another source. Reconstruct Audit arguments rather than relying on a base command. Test benign explanations, distinguish observations from inference, and use ATT&CK mappings to describe supported behavior.

For every finding, save **question, index/data view, mode, exact query and filters, time boundaries/timezone, count definition, event references, raw values, correlation keys, competing explanation, and remaining gaps**. SIEM supports collection and correlation; SOAR can automate repeatable follow-up, while the analyst remains responsible for the conclusion.

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
- Pivot from a threat-group IOC hash back into Sysmon process events. For a hash embedded in a combined `SHA1=...,MD5=...,SHA256=...` value, use the mode-specific, field-scoped patterns in [IOC and wildcard pivots](#ioc-and-wildcard-pivots). Confirm the complete algorithm and digest in the matching event before trusting it.
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

### Email-to-endpoint correlation: practical search patterns

Use this workflow when an email campaign may involve extortion, impersonation, phishing links, or suspicious endpoint activity. The field names below are **examples observed in a DDEI-to-Wazuh and Sysmon pipeline**, not universal names. Inspect a raw event before applying them elsewhere.

| Investigation question | Candidate field or source | What to verify |
| --- | --- | --- |
| Which sender or service sent the message? | DDEI `data.srcuser`, sender domain | Inspect the complete address and message metadata; a service domain is not proof of sender identity. |
| Who received it? | DDEI `data.dstuser` | Aggregate **unique recipient values**, not just the number of matching events. |
| Was it delivered or quarantined? | DDEI `data.action_taken` | Separate `delivered` from `quarantined`; confirm final disposition and message ID where available. |
| Did someone reply? | Reverse direction: `data.dstuser` equals original sender; inspect `data.srcuser` | Check `data.event_id` (for example `EMAIL_SEND`), subject/thread and timestamps. An outgoing message alone does not prove compromise. |
| Does the subject contain a suspicious link? | DDEI `data.mail_subject`, extracted URL | Preserve the **full hostname** and compare its registered domain and spelling with the legitimate organization. |
| Did an endpoint resolve the linked name? | Sysmon Event ID 22, `data.win.eventdata.queryName` | Confirm the queried hostname, `queryResults`, process image, user, timestamp and reporting endpoint. |
| Who performed the DNS lookup? | `data.win.eventdata.user`, `data.win.eventdata.image`, `agent.ip` | Distinguish the account/process and querying host from the **DNS answer IP**; validate host attribution when fields disagree. |
| Was related software installed? | Windows `MsiInstaller` event/message or software inventory | Confirm product name and successful installation in the original event. Presence of legitimate software alone is not malicious. |
| Is an indicator associated with a threat actor? | CTI domain/IP/hash records | Record source, confidence and observation date; intelligence attribution is not independent proof of who sent the email. |

**Discover, then narrow.** When the parser is unknown, a bounded wildcard search for an organization domain or a term such as `*install*` can help locate representative events. Inspect the raw event to discover the actual field path, then switch to precise field filters. Broad or leading wildcards may be expensive, incomplete, or unsupported depending on the search mode and mappings.

**Count the right thing.** Event hits are not necessarily unique messages, recipients, or installations. Email detection and send records may describe related activity. Prefer a unique message identifier when available; use a terms/Top Values aggregation for distinct recipients, and inspect the underlying records.

**Correlate in order.** Sender and recipients → delivery disposition → reply or other user interaction → subject URL and lookalike domain → endpoint DNS/process/user → network or browser evidence → any installation or execution → intelligence enrichment. Match host identity and timestamps at each pivot; don't connect events solely because they mention the same domain.

**Evidence boundaries:** `delivered` does not mean opened; a reply is interaction but not credential theft; Sysmon DNS Event 22 establishes a lookup, not necessarily a click; a DNS answer is not the querying system; software installation does not establish malicious use; a CTI actor tag does not establish attribution. Escalate based on corroborated behavior and impact.

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
