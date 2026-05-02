> Command-line packet analyzer (sniffer) used to capture, filter, and inspect TCP/IP packets flowing through a network interface in real time or from saved capture files.

---

# Metadata

| Field            | Value                                        |
| ---------------- | -------------------------------------------- |
| **Category**     | `network-forensics / traffic-analysis / packet-capture` |
| **Platform**     | Linux / macOS / BSD / Cross-platform         |
| **Language**     | C                                            |
| **License**      | BSD-3-Clause                                 |
| **Version**      | 4.99.x (latest)                              |
| **Authors**      | The Tcpdump Group                            |
| **Last updated** | 02-05-2025                                   |

---

# Description

**tcpdump** is a low-level, CLI-based packet capture tool that operates at the network and transport layers (L3/L4). It intercepts packets passing through one or more network interfaces and either prints them to stdout or writes them to a `.pcap` file for later analysis.

It solves three core problems:

- **Connectivity debugging** — verify that packets are actually arriving at or leaving a host
- **Protocol analysis** — inspect DNS queries, TCP handshakes, ICMP messages, and more
- **Security monitoring** — detect port scans, DNS exfiltration, cleartext credentials, and anomalous traffic

tcpdump uses the **Berkeley Packet Filter (BPF)** engine to filter traffic in the kernel before it reaches userspace — making it extremely efficient even on high-traffic hosts.

---

# Installation

```bash
# Debian / Ubuntu
sudo apt install tcpdump

# RHEL / CentOS / Fedora
sudo dnf install tcpdump

# macOS (pre-installed; or via Homebrew)
brew install tcpdump

# Verify installation
tcpdump --version
```

> **Prerequisites:** Requires `root` or `CAP_NET_RAW` capability to open network interfaces. On Linux you can grant it without sudo: `sudo setcap cap_net_raw+ep $(which tcpdump)`

---

# Basic Usage

> [!warning]
> tcpdump is fully CLI-based. Always use `sudo` unless CAP_NET_RAW has been granted explicitly.

```bash
# Minimal — capture on all interfaces
sudo tcpdump -i any

# Capture with no DNS resolution, verbose, 100 packets max
sudo tcpdump -n -v -c 100 -i any
```

## Prerequisite Concepts

Before using tcpdump effectively, understand these building blocks:

| Concept | Explanation |
|---|---|
| **Packet** | A chunk of data with a header (routing info) and a payload (actual data) |
| **Interface** | The network "ear" — `eth0`, `wlan0`, `lo`, `any` |
| **IP:Port** | Address + door — e.g. `192.168.1.5.443` means IP `192.168.1.5`, port `443` |
| **Protocol** | TCP (reliable), UDP (fast), ICMP (diagnostic), ARP (Layer 2 discovery) |
| **BPF Filter** | Expression written after all flags — tells tcpdump what to keep |

### List available interfaces

```bash
sudo tcpdump -D
```

```
1.eth0
2.wlan0
3.lo (Loopback)
4.any (Pseudo-device that captures on all interfaces)
```

## Common Options

| Flag | Example | Description |
|---|---|---|
| `-i <iface>` | `-i any` | Network interface to capture on. `any` = all |
| `-c <n>` | `-c 10000` | Stop after capturing N packets — **always use in production** |
| `-w <file>` | `-w out.pcap` | Write raw packets to `.pcap` file instead of printing |
| `-r <file>` | `-r out.pcap` | Read and replay packets from a `.pcap` file |
| `-n` | `-n` | Skip DNS reverse lookups — show raw IPs |
| `-v` / `-vv` / `-vvv` | `-vv` | Increase verbosity (TTL, checksum, options…) |
| `-A` | `-A port 80` | Print packet payload as ASCII — HTTP only, not HTTPS |
| `-X` | `-X` | Print payload as HEX |
| `-XX` | `-XX` | Print payload as HEX + ASCII |
| `-e` | `-e -i any` | Include Ethernet frame info (MAC addresses) |
| `-p` | `-p` | Disable promiscuous mode — only your machine's traffic |
| `-t` / `-tttt` | `-tttt` | No timestamp / full date+time timestamp |
| `-G <sec>` | `-G 600` | Rotate output file every N seconds (use with `-w`) |

---

# Analysis

> Each section below covers a distinct feature or use-case of tcpdump, with the relevant flags and annotated sample output.

---

## Packet Output Anatomy

> Understanding what tcpdump prints before you can use it effectively.

### UDP / DNS Packet

```
10:52:13.919782 IP 192.168.1.241.63019 > 192.168.1.1.53: 44000+ A? ask.metafilter.com. (36)
│               │  │                    │                  │      │                    │
│               │  └─ Source IP:port    └─ Dest IP:port    │     └─ DNS query (A
rec) │
│               └─ Protocol (IP)                            └─ DNS Transaction ID        └─ Size (bytes)
└─ Timestamp (HH:MM:SS.microseconds)

# DNS Response:
10:52:13.928894 IP 192.168.1.1.53 > 192.168.1.241.63019: 44000 2/0/0 CNAME metafilter.com., A 54.186.13.33
                                                                 └─────┘ └──────────────────────────────────┘
                                                                 2 ans / 0 auth / 0 additional    Resolved records
```

> **Real-world observation:** 3 queries sent at 10:52:03 → 10:52:08 → 10:52:13, but only the 3rd got a response. This indicates upstream **packet loss** — the client retried every ~5 seconds until the packet got through.

### TCP Packet

```
11:36:26.353797 IP 192.168.1.241.45296 > 192.241.182.146.443: Flags [.], ack 2291349910, win 319, length 0
                                                               └──────┘   └──────────┘   └──────┘  └──────┘
                                                               TCP flag   ACK number     Window    Payload size
                                                               (. = ACK)
```

### TCP Flags Reference

| Symbol | Name | Meaning | When you see it |
|--------|------|---------|-----------------|
| `[S]` | SYN | Initiate connection | Client → Server (handshake step 1) |
| `[S.]` | SYN-ACK | Accept connection | Server → Client (handshake step 2) |
| `[.]` | ACK | Acknowledgement, no data | Everywhere |
| `[P.]` | PSH+ACK | Data being pushed | During data transfer |
| `[F.]` | FIN+ACK | Graceful close | End of connection |
| `[R.]` | RST+ACK | Forced reset | Error or firewall block |
| `[R]` | RST | Reject | Closed port or crash |

### TCP 3-Way Handshake

```
Client                          Server
  │                               │
  │──── [S]  SYN ────────────────►│   "I want to connect"
  │◄─── [S.] SYN-ACK ────────────│   "OK, go ahead"
  │──── [.]  ACK ────────────────►│   "Great, we're connected"
  │    [CONNECTION ESTABLISHED]   │
```

---

## BPF Filters

> Filters are written after all flags. They run inside the kernel via the BPF engine — only matching packets ever reach userspace.

```bash
sudo tcpdump [options] [filter expression]
```

### Host Filters

```bash
tcpdump host 192.168.1.1          # traffic to OR from
tcpdump src host 192.168.1.1      # traffic FROM only
tcpdump dst host 192.168.1.1      # traffic TO only
```

### Port Filters

```bash
tcpdump port 80                   # any direction
tcpdump src port 1234             # outgoing from port
tcpdump dst port 443              # incoming to port
```

### Protocol Filters

```bash
tcpdump tcp
tcpdump udp
tcpdump icmp
tcpdump arp
```

### Subnet Filters

```bash
tcpdump net 192.168.1.0/24
tcpdump src net 10.0.0.0/8 and dst net 172.16.0.0/12
tcpdump broadcast
```

### Combining Filters

```bash
tcpdump host 10.0.0.5 and port 80          # HTTP from specific host
tcpdump port 53 or port 80                 # DNS or HTTP
tcpdump not port 22                        # everything except SSH
tcpdump tcp and dst port 443 and src net 192.168.1.0/24
```

### TCP Flag Filters

```bash
tcpdump 'tcp[tcpflags] & tcp-syn != 0'              # any SYN
tcpdump 'tcp[tcpflags] == tcp-syn'                  # pure SYN (no ACK) — step 1 only
tcpdump 'tcp[tcpflags] & tcp-rst != 0'              # forced resets
tcpdump 'tcp[tcpflags] & tcp-fin != 0'              # graceful closes
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'   # SYN-ACK
```

> **Tip:** Many pure SYN packets with no SYN-ACK response → possible **SYN flood attack**.

> **Note:** Always wrap complex BPF expressions in **single quotes** to protect special characters from shell interpretation.

---

## Raw Byte Offset Filters

> Direct access to specific bytes inside the packet payload. Syntax: `protocol[offset:size]`

```bash
# DNS RCODE = 3 → NXDOMAIN (domain does not exist)
# Byte 11 of UDP payload = DNS flags field; lower 4 bits = RCODE
tcpdump 'udp[11]&0xf==3'

# DNS RCODE = 2 → SERVFAIL
tcpdump 'udp[11]&0xf==2'

# Any DNS error (RCODE != 0)
tcpdump 'udp[11]&0xf!=0'

# HTTP GET requests  (0x47455420 = ASCII "GET ")
tcpdump 'tcp[20:4] == 0x47455420'

# HTTP POST requests
tcpdump 'tcp[20:4] == 0x504f5354'

# ICMP echo request (outgoing ping)
tcpdump 'icmp[icmptype] == icmp-echo'

# ICMP echo reply (incoming ping response)
tcpdump 'icmp[icmptype] == icmp-echoreply'

# Large IP packets > 1400 bytes (potential exfiltration)
tcpdump 'ip[2:2] > 1400'
```

**DNS RCODE values:**

| RCODE | Meaning |
|-------|---------|
| 0 | NOERROR — success |
| 1 | FORMERR — malformed query |
| 2 | SERVFAIL — server-side failure |
| 3 | NXDOMAIN — domain does not exist |
| 5 | REFUSED — server refused to answer |

---

## Output Formats

> Control how much detail tcpdump prints per packet.

```bash
tcpdump -v port 80          # TTL, TOS, IP ID, fragmentation info
tcpdump -vv port 80         # even more detail
tcpdump -vvv port 80        # maximum — checksum validation, all options
tcpdump -A port 80          # ASCII payload (useful for plaintext HTTP)
tcpdump -X port 80          # HEX payload only
tcpdump -XX port 80         # HEX + ASCII side by side
```

### Sample `-XX` output

```
11:42:05.123456 IP 192.168.1.5.54321 > 93.184.216.34.80: Flags [S]
        0x0000:  4500 003c 1234 4000 4006 f3c8 c0a8 0105  E..<.4@.@.......
        0x0010:  5db8 d822 d431 0050 0000 0000 0000 0000  ]..".1.P........
        └────┘   └──────────────────────────────────────┘ └──────────────┘
        Offset   HEX bytes                                ASCII (. = non-printable byte)
```

---

## Saving & Replaying Captures

> The recommended workflow: capture raw with `-w`, analyse offline with Wireshark.

```bash
# Save to file
sudo tcpdump -w capture.pcap

# Save with packet limit (safe for production)
sudo tcpdump -c 10000 -w capture.pcap

# Rotating files — new file every 10 minutes, max 10 MB each
sudo tcpdump -w capture-%Y%m%d-%H%M%S.pcap -G 600 -C 10

# Read back offline
tcpdump -r capture.pcap

# Read with filter applied
tcpdump -r capture.pcap port 80

# Read with full verbosity
tcpdump -r capture.pcap -vv
```

---

# Use Cases in Network Forensics

- **Connectivity Verification** — confirm packets are reaching a service (`tcpdump -n dst port 8080`) before suspecting application-level bugs
- **DNS Analysis** — trace failed lookups, retries, NXDOMAIN floods, and DNS-over-UDP timing issues
- **TCP Session Reconstruction** — capture full sessions with `-w`, then use Wireshark's *Follow TCP Stream* to reassemble the data
- **Anomaly Detection** — identify SYN floods, ARP storms, abnormally large packets, or high-frequency DNS queries
- **IOC Extraction** — extract suspicious IPs, domains, and payload strings from captures for threat-intel correlation
- **Cleartext Credential Detection** — use `-A` on ports 21 (FTP), 23 (Telnet), 80 (HTTP) to spot unencrypted credentials
- **Latency & Retransmission Analysis** — save a `.pcap` and open in Wireshark → Statistics → TCP Stream Graphs to visualise packet loss and RTT

---

# Troubleshooting Workflows

## Workflow 1 — Server Not Responding

```bash
# Step 1: Are packets arriving at all?
sudo tcpdump -n -i any dst port 8080
# Nothing? → firewall or routing issue upstream

# Step 2: Is the server sending responses?
sudo tcpdump -n -i any src port 8080
# Nothing? → application is not listening or has crashed

# Step 3: Are there TCP RSTs? (connection being rejected)
sudo tcpdump -n 'tcp[tcpflags] & tcp-rst != 0 and port 8080'
# RST from server → port closed or process crashed
# RST from client → client-side timeout
```

## Workflow 2 — Slow Network

```bash
# Step 1: Capture traffic for offline analysis
sudo tcpdump -n -w slow.pcap
# Open in Wireshark → Statistics → TCP Stream Graphs → Time Sequence
# Flat lines = sender waiting for ACK = packet loss

# Step 2: Are there ICMP unreachable messages? (something is blocking)
sudo tcpdump -n 'icmp and icmp[icmptype] == icmp-unreach'
```

## Workflow 3 — DNS Not Working

```bash
# Step 1: Am I sending queries at all?
sudo tcpdump -n -i any port 53

# Step 2: Am I receiving responses?
sudo tcpdump -n 'udp src port 53'

# Step 3: What errors am I getting back?
sudo tcpdump -n 'udp[11]&0xf!=0'    # any error
sudo tcpdump -n 'udp[11]&0xf==2'    # SERVFAIL
sudo tcpdump -n 'udp[11]&0xf==3'    # NXDOMAIN
```

---

# Limitations & Caveats

- Requires **root** or `CAP_NET_RAW` — cannot be run by unprivileged users by default.
- `-A` and `-X` only reveal content of **unencrypted** protocols — HTTPS, SSH, and TLS traffic show as gibberish.
- `-i any` is **not supported on macOS** — use a specific interface (`-i en0` for WiFi, `-i en1` for Ethernet).
- Without `-c`, a capture on a busy server can **fill the disk** within minutes — always set a packet or time limit.
- `.pcap` files may contain **passwords, tokens, cookies** — treat them as sensitive and store/transmit securely.
- tcpdump sees packets **after the kernel's firewall (iptables/nftables)** processes them on ingress but **before** on egress — keep this in mind when debugging dropped packets.
- Always wrap complex BPF expressions in **single quotes** to prevent shell expansion of `&`, `|`, `>`.
- For high-throughput links, consider `tcpdump -B <bufsize>` to increase the kernel capture buffer and avoid dropped packets.

---

# References

- [Official tcpdump documentation](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [tcpdump source code (GitHub)](https://github.com/the-tcpdump-group/tcpdump)
- [BPF filter syntax reference](https://www.tcpdump.org/manpages/pcap-filter.7.html)
- [Wireshark — companion GUI for `.pcap` analysis](https://www.wireshark.org)
- [Daniel Miessler's tcpdump primer](https://danielmiessler.com/study/tcpdump/)

---

# Notes

- Use `-n` almost always — without it, tcpdump performs reverse DNS on every IP, which is slow and confusing when debugging DNS itself.
- The "golden workflow" in practice: `sudo tcpdump -c 50000 -w capture.pcap [filter]` on the server, then `scp` the file locally and open in Wireshark.
- For HTTP body inspection, `ngrep` is more ergonomic than `tcpdump -A` for interactive use.
- `nohup sudo tcpdump -w capture-%Y%m%d-%H%M%S.pcap -G 300 -C 50 &` — background long-running capture with rotating files, safe for leaving overnight on a production host.

#networking #tcpdump #packets #bpf #dns #tcp #udp #icmp #security #sysadmin #wireshark #troubleshooting #linux #forensics
