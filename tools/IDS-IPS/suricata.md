> Suricata — open-source IDS/IPS/NSM engine for real-time network traffic analysis, used in network forensics and threat detection.

---

# Metadata
| Field            | Value                                            |
| ---------------- | ------------------------------------------------ |
| **Category**     | `network-forensics / IDS-IPS / traffic-analysis` |
| **Platform**     | Linux / Windows / macOS / Cross-platform         |
| **Language**     | C                                                |
| **License**      | GPL-2.0                                          |
| **Version**      | 1.2 (latest stable)                              |
| **Authors**      | Panagiotis Michalitsios                          |
| **Last updated** | 05-07-2026                                       |

---

# Description

**Suricata** is an open-source engine for **Intrusion Detection (IDS)**, **Intrusion Prevention (IPS)**, and **Network Security Monitoring (NSM)**. It operates at the network level, "reading" traffic either live from an interface or from a pcap file, and passes it through a packet-processing pipeline that culminates in **rule matching**.

Core capabilities:
- Deep packet inspection at the protocol level (HTTP, DNS, TLS, SMB, SSH, etc.)
- Signature-based detection through **rules** (largely compatible with the Snort/ET rule format)
- Production of structured logs (mainly `eve.json`) for SIEM/forensics pipelines
- File extraction, TLS fingerprinting, Lua scripting for custom detection
- Multi-threaded architecture → high performance on 10G+ networks

---

# Architecture — Tree Diagram (Pipeline)

```
Suricata Engine
│
├── 1. Packet Acquisition (Capture)
│   ├── AF_PACKET / PF_RING / NFQUEUE / PCAP / eBPF
│   └── -i <iface>  or  -r <file.pcap>
│
├── 2. Decode
│   ├── Ethernet / IP / IPv6
│   └── Transport layer: TCP / UDP / ICMP
│
├── 3. Stream Engine
│   ├── TCP reassembly (stream reconstruction)
│   └── Flow tracking (state table per connection)
│
├── 4. Application Layer Parsers (Protocol Detection)
│   ├── HTTP / HTTP2
│   ├── DNS
│   ├── TLS/SSL
│   ├── SMB / SSH / FTP / SMTP / NFS / etc.
│   └── Produces structured "transactions" per protocol
│
├── 5. Detection Engine
│   ├── Loads the Rules (.rules files)
│   ├── Multi-Pattern Matching (MPM) — fast content pre-filtering
│   ├── Matches against Signatures (content, pcre, flow, etc.)
│   └── Decision: alert / drop / pass / reject
│
├── 6. Output Engine (Logging)
│   ├── eve.json   → JSON events (alerts, flow, http, dns, tls, files...)
│   ├── fast.log   → short one-line-per-alert log
│   ├── stats.log  → engine/performance statistics
│   ├── suricata.log → engine/runtime log (errors, startup, etc.)
│   └── (optional) pcap logging, file-store for extracted files
│
└── 7. Action
    ├── IDS mode  → detection/alerting only
    └── IPS mode  → drop/reject in inline deployment (NFQUEUE / AF_PACKET inline)
```

---

# Which logs Suricata "reads" / produces

Suricata doesn't "read" logs as input (aside from pcap files as traffic input) — it **produces** the following files, which are used in forensics/SIEM workflows:

| Log file            | Content                                                                      |
| -------------------- | ----------------------------------------------------------------------------- |
| `eve.json`          | Main structured JSON log — alerts, flow, http, dns, tls, ssh, smb, files, stats, anomaly |
| `fast.log`          | Simple, one-line representation of each alert (fast to grep)                 |
| `stats.log`         | Periodic engine metrics (throughput, drops, memory)                          |
| `suricata.log`      | Runtime/engine log — errors, config loading, startup/shutdown                |
| `http.log` (opt.)   | If separately enabled — HTTP requests/responses                              |
| `dns.log` (opt.)    | DNS queries/responses                                                        |
| `tls.log` (opt.)    | TLS handshake metadata (SNI, certificate info)                               |
| `pcap logs` (opt.)  | Per-flow or full pcap capture when rotating pcap logging is enabled          |

> In practice, nearly all of the above are today unified inside `eve.json` via the `event_type` field (e.g. `event_type: alert`, `event_type: dns`, `event_type: flow`), and the separate log files (http.log, dns.log) are considered legacy.

---

# Where rules go

- Stored in `.rules` files (e.g. `/etc/suricata/rules/`)
- Loaded via `suricata.yaml` under the `rule-files:` section
- Can come from: **Emerging Threats (ET)**, **custom local rules**, or the **suricata-update** tool
- Organized into categories (e.g. `emerging-malware.rules`, `emerging-scan.rules`, `local.rules`)

---

# Structure of a Suricata Rule

A rule consists of **3 main parts**:

```
action  proto  src_ip  src_port  ->  dst_ip  dst_port  (options)
  │       │       │        │      │      │       │          │
  │       │       └────────┴──────┘      └───────┴──────────┘
  │       │              HEADER                  OPTIONS
ACTION  PROTOCOL
```

## 1. Action
Determines what Suricata does when the rule matches:

| Action  | Description                                       |
| ------- | ---------------------------------------------------|
| `alert` | Generates an alert event (does not block)          |
| `drop`  | Drops the packet (only in IPS/inline mode)          |
| `reject`| Drops + sends a RST/ICMP unreachable                |
| `pass`  | Allows the packet, stops further inspection         |

## 2. Header
```
proto src_ip src_port direction dst_ip dst_port
```
- **proto**: `tcp`, `udp`, `icmp`, `http`, `dns`, `tls`, etc.
- **src_ip / dst_ip**: IP address or variables (`$HOME_NET`, `$EXTERNAL_NET`, `any`)
- **src_port / dst_port**: port number or `any`
- **direction**: `->` (one-way) or `<>` (both directions)

## 3. Options
Placed inside parentheses, separated by `;`:

| Option                       | Purpose                                             |
| ---------------------------- | --------------------------------------------------- |
| `msg:"..."`                  | Description of the alert                            |
| `content:"..."`              | Searches for a specific byte pattern in the payload |
| `pcre:"/regex/"`             | Regex matching                                      |
| `flow:established,to_server` | Connection state/direction                          |
| `classtype:trojan-activity`  | Threat classification                               |
| `sid:1000001`                | Unique Signature ID (>1000000 for custom rules)     |
| `rev:1`                      | Rule revision number                                |
| `reference:url,...`          | External documentation/CVE                          |
| `metadata:...`               | Additional tags (severity, created_at, etc.)        |

### Full rule example
```
alert tcp $EXTERNAL_NET any -> $HOME_NET any (
    msg:"ET MALWARE Suspicious User-Agent Detected";
    flow:established,to_server;
    content:"User-Agent|3a| BadBot"; http_header;
    classtype:trojan-activity;
    sid:1000001; rev:1;
)
```
Breakdown: TCP traffic from any external IP toward the internal network. If the specified `User-Agent` header is found on an established TCP connection, an alert is generated.

---

# Installation
```bash
# Debian/Ubuntu
sudo apt update
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt install suricata

# Update rulesets (Emerging Threats etc.)
sudo suricata-update
```
> **Note:** Requires `libpcap`, root/`CAP_NET_RAW` for live capture, and a properly configured `suricata.yaml` (HOME_NET, interfaces, rule-files).

---

# Basic Usage
```bash
# Live capture on an interface
sudo suricata -c /etc/suricata/suricata.yaml -i eth0

# Analyzing a pcap file (offline forensics)
suricata -c /etc/suricata/suricata.yaml -r capture.pcap -l /var/log/suricata/
```

## Common Options
| Flag / Option | Description                                    |
| -------------- | ------------------------------------------------ |
| `-i <iface>`   | Live capture from a network interface             |
| `-r <file>`    | Read traffic from a pcap file                     |
| `-l <dir>`     | Output directory for logs                         |
| `-c <config>`  | Path to suricata.yaml config                      |
| `-T`           | Test mode — validates config/rules without running|
| `-v` / `-vvv`  | Verbose logging (more `v` = more verbose)         |
| `--af-packet`  | Use AF_PACKET capture mode                        |

---

# Analysis

## Detection Engine (Rule Matching)
> The core of Suricata. Loads the `.rules` files, compiles the signatures into MPM structures, and matches every packet/stream/transaction against them. This is where the decision to generate an alert/drop is made.

_Sample output (fast.log):_
```
07/05/2026-10:22:14.552341  [**] [1:1000001:1] ET MALWARE Suspicious User-Agent Detected [**] [Classification: Trojan Activity] [Priority: 1] {TCP} 203.0.113.5:443 -> 192.168.1.10:51322
```

---

## Eve.json (Unified Event Log)
> The central JSON log. Each line is a separate JSON event with an `event_type` (alert, flow, http, dns, tls, fileinfo, stats...). Used for forensics/SIEM ingestion (e.g. Elastic, Splunk).

_Sample output:_
```json
{"timestamp":"2026-07-05T10:22:14.552341+0300","event_type":"alert","src_ip":"203.0.113.5","dest_ip":"192.168.1.10","alert":{"signature":"ET MALWARE Suspicious User-Agent Detected","category":"Trojan Activity","severity":1}}
```

---

## Protocol Parsers (App-Layer Logging)
> Parse application-layer protocols (HTTP, DNS, TLS, SMB) and produce transaction-level logs regardless of whether a rule matches. Useful for NSM/visibility, not just detection.

_Sample output (eve.json, event_type=dns):_
```json
{"event_type":"dns","dns":{"type":"query","rrname":"example.com","rrtype":"A"}}
```

---

## File Extraction
> When enabled (`file-store`), extracts files transferred over HTTP/SMB/FTP for later analysis (malware analysis, hash extraction).

```
# Enable in suricata.yaml
file-store:
  enabled: yes
  dir: /var/log/suricata/files
```

---

# Use Cases in Network Forensics
- **Traffic Reconstruction** — reassembling TCP streams from a pcap file to recreate communications
- **Anomaly Detection** — identifying unusual protocol anomalies (e.g. malformed DNS)
- **Protocol Analysis** — decoding HTTP/TLS/DNS transactions for IOC extraction
- **IOC Extraction** — extracting IPs, domains, JA3/TLS fingerprints, file hashes from `eve.json`
- **Retrospective Analysis** — running rules against historical pcap files after an incident

---

# Limitations & Caveats
- Does not perform deep behavioral/ML analysis on its own — it's signature/protocol based
- Performance depends heavily on proper tuning (CPU affinity, ring buffers, thread config)
- Encrypted traffic (without TLS decryption) limits content inspection to metadata (SNI, JA3)
- Requires `CAP_NET_RAW` / root for live capture
- Large rulesets (thousands of rules) can significantly increase CPU load

---

# References
- Official Documentation: https://docs.suricata.io/
- Source Code / Repository: https://github.com/OISF/suricata
- Rule Writing Guide: https://docs.suricata.io/en/latest/rules/intro.html
- Emerging Threats Rulesets: https://rules.emergingthreats.net/

---

# Notes
Good practice: assign custom sids above `1000000` so they don't collide with Emerging Threats/community rules. For pcap-based forensics, `-r` mode with full `eve.json` logging gives the richest picture for retrospective analysis.
