# Description

Traffic analysis tools allow us to **inspect and interpret captured network traffic** in order to understand what happened on a network. Unlike packet capture tools that just record, these tools help us find patterns, anomalies, and extract meaningful data.

The captured data is usually in `.pcap` or `.pcapng` format, or structured logs generated automatically.

### When do we use these tools?
- When we want to analyze a `.pcap` file and understand the traffic
- When we need to identify suspicious connections or unusual behavior
- When we want to extract files, credentials, or sessions from captured traffic
- When we need structured logs for further investigation

---

## Tools

| Tool         | Description                                                          | Platform                | File              |
| ------------ | -------------------------------------------------------------------- | ----------------------- | ----------------- |
| Zeek         | Network security monitoring framework that generates structured logs | Linux / macOS           | `zeek.md`         |
| NetworkMiner | Passive network forensic analyzer — extracts files and credentials   | Linux / Windows         | `networkminer.md` |
| ntopng       | Real-time network traffic monitoring and analysis                    | Linux / Windows / macOS | `ntopng.md`       |

---

*[← Back to main page](_TOOLS.md)*
