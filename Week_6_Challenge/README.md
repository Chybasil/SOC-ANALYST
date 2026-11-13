# Week 6 Challenge – Analysing DNS logs to spot command-and-control (C2) activity

## NB: This is a fictional case study based on provided documents, not a real vendor audit.

### Scenario: Northwind Healthcare staff laptop, 10.20.4.37 looks ‘chatty’ on DNS, and requires investigation from yesterday’s logs.

# DNS log extract GMT)

### Full-day Summary

| Client | Host | Total queries | Unique names | Top domain (queries) | Record types | NXDOMAIN |
|---|---|---|---|---|---|---|
| 10.20.4.37 | NW-FIN-LT-014 | 2,931 | 1,512 | cdn-sync.example.net (1,440) | A, TXT (1,447) | 0% |
| 10.20.4.52 | NW-ENG-LT-022 | 3,104 | 96 | agent.monitoring-vendor.example.com (288) | A, AAAA | 0.4% |
| 10.20.4.15 | NW-SUP-LT-009 | 2,214 | 143 | mail.google.com (311) | A, AAAA | 1.1% |

### Domain Information

| Domain | Registered | Registrant | Resolves to | On IT's approved software list? |
|---|---|---|---|---|
| cdn-sync.example.net | 6 Nov 2026 (3 days ago) | Privacy-protected | 192.0.2.88 | No |
| agent.monitoring-vendor.example.com | 14 Mar 2019 | MonitorVendor Inc. | 198.51.100.150 | Yes (endpoint monitoring agent) |

## Deliverable 1: Verdict

a.	10.20.20.37 looks infected and suspicious because of number of sequential TXT queries.

b.	That is also an example of DNS-based payload retrieval/command-and-control (C2).

c.	Again, from domain information cdn-syn.example.net is not in its approved software list which 10.20.4.37 responded NO and has 0% NXDOMAIN from full-day summary.

## Deliverable 2: The decoy

a.	10.20.4.52 is a benign in this query. It is legitimate and exists in its approved software list from the domain information and queries for A, AAAA which exists in the domain.

## Deliverable 3: Containment

a.	Extract the exact queried name and check their registration age, reputation, and passive DNS history and whether the TXT answer contain encoded/binary-looking data, base64/hex strings, or command-like text.

b.	Identify the user that generated the TXT query.

c.	Check the new IP, 10.20.4.15 if it resolves to 198.51.100.150 (vendor monitoring agent).

## Deliverable 4: Detection idea

a.	Volume-based rule would detect an extreme high rate of TXT queries from one IP source.
