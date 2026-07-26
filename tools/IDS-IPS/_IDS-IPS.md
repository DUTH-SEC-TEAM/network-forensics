#  Description

Intrusion Detection and Prevention Systems monitor network traffic for **known attack signatures and suspicious behavior**. An IDS alerts when it detects something malicious, while an IPS can actively block it.

### When do we use these tools?
- When we want to detect known attack patterns in live or captured traffic
- When we need to generate alerts for suspicious network activity
- When we analyze a `.pcap` file looking for indicators of compromise (IOCs)
- When we want to correlate network events with known threat signatures

---

## Tools

| Tool     | Description                                               | Platform                | File          |
| -------- | --------------------------------------------------------- | ----------------------- | ------------- |
| Snort    | Classic signature-based IDS/IPS, industry standard        | Linux / Windows         | `snort.md`    |
| Suricata | High-performance multi-threaded IDS/IPS with JSON logging | Linux / Windows / macOS | `suricata.md` |

---

*[← Back to main page](_TOOLS.md)*
