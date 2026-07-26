# TShark

> TShark is the command-line network protocol analyzer bundled with Wireshark. It captures and decodes packets from a live interface or reads from a saved `.pcap` file — ideal for scripting, automation, and headless server environments.

---

# Metadata

| Field            | Value                                        |
| ---------------- | -------------------------------------------- |
| **Category**     | `network-forensics / traffic-analysis / ...` |
| **Platform**     | Linux / Windows / macOS                      |
| **Language**     | Tshark                                       |
| **License**      | -                                            |
| **Version**      | 1.1                                          |
| **Authors**      | Panagiotis Michalitsios                      |
| **Last updated** | 16/05/2026                                   |

## Prerequisites

- [ ] Wireshark / TShark installed
- [ ] Capture permissions (root or `wireshark` group membership)
- [ ] Basic understanding of TCP/IP and networking

---

## Step 1: Installation

### Linux (Debian / Ubuntu)

```bash
sudo apt update && sudo apt install tshark -y
```

Allow packet capture without root (recommended):

```bash
sudo usermod -aG wireshark $USER
# Log out and back in for the change to take effect
```

> Adding yourself to the `wireshark` group lets you run TShark without `sudo`, which is safer for everyday use.

### Linux (RHEL / CentOS / Fedora)

```bash
sudo dnf install wireshark-cli -y
```

### macOS

```bash
brew install wireshark
```

### Windows

TShark is installed alongside Wireshark. Run it from:

```
C:\Program Files\Wireshark\tshark.exe
```

### Verify installation

```bash
tshark --version
# Prints the TShark version and the libpcap/Wireshark library versions it was built with
```

---

## Step 2: List Capture Interfaces

Before capturing, you need to know which network interfaces are available on your system.

```bash
tshark -D
# Lists all available capture interfaces with their index numbers
```

Example output:
```
1. eth0
2. wlan0
3. lo (Loopback)
4. any
```

```bash
tshark -D -v
# Verbose interface list — shows additional details like interface type and capabilities

ip link show
# Alternative Linux system command to list network interfaces
```

> Use the interface **name** (e.g., `eth0`) or its **index number** (e.g., `1`) with the `-i` flag when capturing.

---

## Basic Syntax

```
tshark [options] [filter expression]
```

| Flag | Description |
|------|-------------|
| `-i <iface>` | Network interface to capture on |
| `-r <file>` | Read packets from a saved `.pcap` file instead of capturing live |
| `-w <file>` | Write raw captured packets to a `.pcap` file |
| `-f <filter>` | Capture filter — BPF syntax, applied *during* capture |
| `-Y <filter>` | Display filter — Wireshark syntax, applied *after* capture |
| `-T <format>` | Output format (`text`, `json`, `fields`, etc.) |
| `-e <field>` | Extract a specific protocol field (used with `-T fields`) |
| `-c <count>` | Stop capturing after N packets |
| `-a <condition>` | Autostop condition (duration, file size, packet count) |
| `-b <condition>` | Ring buffer — rotate output files on a condition |
| `-V` | Verbose mode — print full protocol dissection for each packet |
| `-q` | Quiet mode — suppress per-packet output (useful with `-z` statistics) |
| `-n` | Disable all name resolution (faster, avoids DNS lookups mid-capture) |
| `-N <flags>` | Fine-grained name resolution control: `m`=MAC, `n`=network, `t`=transport |
| `-2` | Two-pass analysis — reads the file twice for more accurate dissection |

---

## Step 3: Live Capture

### Basic capture

```bash
tshark -i eth0
# Starts capturing on eth0 and prints a one-line summary for every packet received
```

```bash
tshark -i any
# Captures on ALL interfaces simultaneously
# Useful when you are unsure which interface carries the traffic you are looking for
```

```bash
tshark -i eth0 -c 100
# Captures exactly 100 packets, then stops automatically
```

```bash
tshark -i eth0 -a duration:30
# Captures for 30 seconds, then stops
# Good for timed snapshots or scripted periodic captures
```

```bash
tshark -i eth0 -w capture.pcap
# Saves all captured packets to capture.pcap — nothing is printed to the terminal
# Open the file later with -r for analysis, or in Wireshark
```

```bash
tshark -i eth0 -a filesize:500000 -w capture.pcap
# Captures until the output file reaches 500 MB, then stops
# filesize is expressed in kilobytes — 500000 KB = ~500 MB
```

### Ring buffer (rotating files)

A ring buffer automatically creates new files and deletes the oldest ones, keeping disk usage under control during long captures.

```bash
tshark -i eth0 -b filesize:100000 -b files:5 -w /tmp/ring.pcap
# Creates up to 5 rotating files of 100 MB each
# When the 6th file would be created, it overwrites the oldest one
# File names are auto-generated: ring_00001_..., ring_00002_..., etc.
```

```bash
tshark -i eth0 -b duration:60 -b files:10 -w /tmp/capture.pcap
# Rotates to a new file every 60 seconds, keeping the last 10 files on disk
# Older files are deleted automatically to maintain the file count limit
```

### Autostop conditions reference

```bash
-a duration:<seconds>    # Stop after N seconds of capture
-a filesize:<KB>         # Stop when the output file reaches N kilobytes
-a files:<count>         # Stop after writing N files (used together with -b)
-a packets:<count>       # Stop after capturing N packets
```

> [!tip]
> You can combine multiple `-a` conditions in the same command. TShark stops as soon as the **first** condition is met.

---

## Step 4: Reading PCAP Files

```bash
tshark -r capture.pcap
# Reads the file and prints a one-line summary for every packet
```

```bash
tshark -r capture.pcap -Y "http"
# Reads the file but only displays packets that match the display filter
# All other packets are silently skipped — the file is not modified
```

```bash
tshark -r capture.pcap -q -z io,stat,1
# Reads quietly (no per-packet output) and prints I/O statistics
# grouped by 1-second intervals — useful for visualising traffic volume over time
```

```bash
tshark -r capture.pcap -V
# Prints the full protocol dissection for every packet
# Equivalent to Wireshark's "Packet Details" panel — very verbose
# Pipe to less or grep when working with large files
```

```bash
tshark -r capture.pcap -Y "frame.number >= 10 && frame.number <= 50"
# Displays only packets 10 through 50
# Useful for zooming in on a specific section of a capture
```

```bash
tshark -r capture.pcap -2 -R "http"
# Two-pass analysis: TShark reads the file twice
# The first pass builds full context (streams, conversations)
# The second pass applies the read filter — more accurate for reassembled streams
```

---

## Display Filters

Display filters use Wireshark's powerful filter language and are applied **after** capture (or on file read). They do not reduce what is captured — only what is shown or extracted.

### Syntax

```
protocol.field  operator  value
```

| Operator | Meaning |
|----------|---------|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` `<` `>=` `<=` | Numeric comparison |
| `&&` or `and` | Both conditions must be true |
| `\|\|` or `or` | At least one condition must be true |
| `!` or `not` | Negation — exclude matching packets |
| `contains` | Field contains a substring |
| `matches` | Field matches a Perl-compatible regular expression |
| `in {a b c}` | Field value is one of the listed values |

### By protocol

```wireshark
ip        # All IPv4 packets
ipv6      # All IPv6 packets
tcp       # All TCP packets (any port)
udp       # All UDP packets
http      # HTTP traffic — decoded by Wireshark, not just port 80
dns       # DNS queries and responses
arp       # ARP requests and replies
icmp      # ICMP messages (ping, unreachable, redirect, etc.)
ftp       # FTP control channel (commands and responses)
ssh       # SSH traffic
tls       # TLS/SSL handshakes and encrypted records
smtp      # SMTP email traffic
```

### By IP address

```wireshark
ip.addr == 192.168.1.1
# Matches packets where 192.168.1.1 appears as source OR destination

ip.src == 10.0.0.1
# Only packets sent FROM 10.0.0.1

ip.dst == 8.8.8.8
# Only packets sent TO 8.8.8.8

ip.src == 192.168.0.0/24
# All packets whose source IP falls within the 192.168.0.x subnet
```

### By port

```wireshark
tcp.port == 80
# TCP packets where port 80 appears as source OR destination

tcp.dstport == 443
# TCP packets going TO port 443 (HTTPS)

udp.port == 53
# UDP packets on port 53 (DNS)

tcp.port in {80 443 8080}
# TCP packets on any of the three listed ports
```

### By MAC address

```wireshark
eth.addr == aa:bb:cc:dd:ee:ff
# Packets where this MAC appears as source OR destination

eth.src == aa:bb:cc:dd:ee:ff
# Only packets sent FROM this MAC address
```

### HTTP

```wireshark
http.request.method == "GET"
# Only HTTP GET requests

http.request.method == "POST"
# Only HTTP POST requests — often contain form submissions or API payloads

http.response.code == 200
# HTTP responses with status 200 OK

http.response.code >= 400
# All client errors (4xx) and server errors (5xx)

http.host contains "google"
# Requests where the Host header contains "google"

http.request.uri contains "login"
# Requests to paths that include the word "login" — useful for spotting auth attempts
```

### DNS

```wireshark
dns.qry.name == "example.com"
# DNS queries for exactly "example.com"

dns.resp.name contains "google"
# DNS responses whose name field contains "google"

dns.flags.response == 0
# Only DNS query packets (not responses)

dns.flags.response == 1
# Only DNS response packets
```

### TCP flags

```wireshark
tcp.flags.syn == 1
# Packets with the SYN flag set (includes SYN-ACK)

tcp.flags.fin == 1
# Packets with the FIN flag set — graceful connection close

tcp.flags.reset == 1
# Packets with the RST flag set — abrupt connection termination

tcp.flags.syn == 1 && tcp.flags.ack == 0
# Pure SYN packets only — the very first packet of a new TCP connection
# Useful for counting how many new connections are being opened
```

### TLS

```wireshark
tls.handshake.type == 1
# TLS Client Hello — the opening message of every TLS handshake

tls.handshake.type == 2
# TLS Server Hello — the server's response to Client Hello

tls.record.version == 0x0303
# TLS 1.2 records (0x0304 = TLS 1.3)

tls.handshake.extensions_server_name
# Packets that contain the SNI extension — reveals the hostname even in encrypted traffic
```

### ICMP

```wireshark
icmp.type == 8
# ICMP Echo Request — the outgoing ping

icmp.type == 0
# ICMP Echo Reply — the response to a ping
```

### Frame size and timing

```wireshark
frame.len > 1000
# Packets larger than 1000 bytes — large transfers, potential data exfiltration

frame.len < 100
# Small packets — keepalives, control traffic, or scanning probes

frame.time_delta > 1.0
# Packets that arrived more than 1 second after the previous packet
# Indicates latency spikes or idle connections
```

### Combining filters

```wireshark
ip.src == 192.168.1.1 && tcp.port == 80
# HTTP traffic from one specific host

(http or dns) && ip.dst == 8.8.8.8
# HTTP or DNS traffic going to 8.8.8.8

not arp && not icmp
# Exclude ARP and ICMP to focus on application-layer traffic
```

> [!tip]
> Full Wireshark display filter reference: https://www.wireshark.org/docs/dfref/

---

## Capture Filters (BPF)

Capture filters use **Berkeley Packet Filter (BPF)** syntax and are applied **during** capture, before any packets are written to disk. They are more efficient than display filters because unmatched packets are discarded immediately by the kernel.

> [!warning]
> BPF syntax is **completely different** from Wireshark display filter syntax. The `-f` flag takes BPF syntax only. Using Wireshark syntax with `-f` will produce a capture error.

```bash
# By host
-f "host 192.168.1.1"
# Matches packets where 192.168.1.1 appears as source OR destination

-f "src host 10.0.0.1"
# Only packets originating FROM 10.0.0.1

-f "dst host 8.8.8.8"
# Only packets destined FOR 8.8.8.8
```

```bash
# By port
-f "port 80"
# TCP or UDP packets where port 80 is source OR destination

-f "tcp port 443"
# Only TCP packets on port 443 (HTTPS)

-f "udp port 53"
# Only UDP packets on port 53 (DNS)

-f "portrange 8000-9000"
# Any packet with a source or destination port in the range 8000–9000
```

```bash
# By protocol
-f "tcp"      # All TCP packets — any port, any host
-f "udp"      # All UDP packets
-f "icmp"     # All ICMP packets
-f "arp"      # All ARP packets
```

```bash
# Combining with boolean operators
-f "tcp port 80 or tcp port 443"
# Captures both HTTP and HTTPS traffic

-f "host 192.168.1.1 and port 22"
# SSH traffic to or from a specific host only

-f "not port 22"
# Everything except SSH — prevents your own management session from polluting the capture
```

```bash
# By subnet
-f "net 192.168.0.0/24"
# All traffic involving any host in the 192.168.0.x network

-f "src net 10.0.0.0/8"
# Only traffic originating from anywhere in 10.x.x.x
```

```bash
# By MAC address
-f "ether host aa:bb:cc:dd:ee:ff"
# All traffic to or from this specific MAC address — works at Layer 2
```

```bash
# By packet size
-f "less 128"
# Only packets smaller than 128 bytes

-f "greater 512"
# Only packets larger than 512 bytes
```

> [!tip]
> You can use both `-f` and `-Y` in the same command. BPF (`-f`) reduces what is captured — saving CPU and disk. The display filter (`-Y`) then refines what is shown from that already-filtered capture.

---

## Output Formatting

The `-T` flag controls how TShark formats its output. The default is `text`.

| Format | Description |
|--------|-------------|
| `text` | One human-readable summary line per packet (default) |
| `fields` | Custom column output — specify which fields to show with `-e` |
| `json` | Full JSON representation of each packet's dissection tree |
| `jsonraw` | JSON with raw byte values included alongside decoded values |
| `pdml` | Verbose XML — mirrors Wireshark's packet detail tree exactly |
| `psml` | Compact XML summary, one element per packet |
| `tabs` | Tab-delimited field output |
| `ek` | Elasticsearch-compatible JSON bulk format |

```bash
tshark -r file.pcap -T json
# Outputs the full dissection of every packet as a JSON array
# Useful for feeding into Python scripts, jq, or SIEM pipelines

tshark -r file.pcap -T json | jq '.[].layers.ip'
# Pipes JSON to jq and extracts only the IP layer from each packet

tshark -r file.pcap -T ek
# Outputs in Elasticsearch bulk format
# Each packet becomes an indexable JSON document, ready to ingest directly
```

---

## Field Extraction

The combination of `-T fields` and one or more `-e <field>` flags lets you build a custom table, extracting only the values you care about from each packet.

```bash
tshark -r file.pcap -T fields -e ip.src -e ip.dst
# Prints two columns: source IP and destination IP, one row per packet
```

```bash
tshark -r file.pcap -T fields -e ip.src -e ip.dst -E header=y
# Same as above, but adds a header row with the field names as column titles
```

```bash
tshark -r file.pcap -T fields -e ip.src -e ip.dst -E separator=","
# Uses a comma as the column separator instead of the default tab
# Output is valid CSV — pipe to a file with > output.csv
```

```bash
tshark -r file.pcap -T fields -e http.host -E quote=d
# Wraps each value in double quotes
# Useful when field values may contain the separator character
```

### Useful fields to extract

```bash
# Layer 2 — Ethernet
eth.src           # Source MAC address
eth.dst           # Destination MAC address

# Layer 3 — IP
ip.src            # Source IP address
ip.dst            # Destination IP address
ip.ttl            # Time to Live — decremented at each hop; useful for traceroute analysis
ip.proto          # Protocol number: 6=TCP, 17=UDP, 1=ICMP

# Layer 4 — TCP
tcp.srcport       # TCP source port
tcp.dstport       # TCP destination port
tcp.flags         # TCP flags as a hex value (e.g., 0x002 = SYN)
tcp.seq           # TCP sequence number
tcp.ack           # TCP acknowledgment number
tcp.len           # Length of the TCP payload in bytes

# Layer 4 — UDP
udp.srcport       # UDP source port
udp.dstport       # UDP destination port
udp.length        # Total length of the UDP datagram including header

# HTTP
http.request.method      # GET, POST, PUT, DELETE, etc.
http.request.uri         # The path portion of the URL (e.g., /api/login)
http.request.full_uri    # The complete URL including scheme and host
http.host                # The value of the HTTP Host header
http.user_agent          # The client's User-Agent string (reveals browser/OS/tool)
http.response.code       # HTTP status code: 200, 301, 404, 500, etc.
http.cookie              # HTTP Cookie header value

# DNS
dns.qry.name      # The domain name being queried
dns.resp.addr     # The IP address returned in the DNS response
dns.qry.type      # Query type: 1=A, 28=AAAA, 15=MX, 5=CNAME, 16=TXT

# Frame metadata
frame.number      # Sequential packet number within the capture file
frame.time        # Absolute timestamp when the packet was captured
frame.time_delta  # Time elapsed since the previous packet (inter-packet gap)
frame.len         # Total length of the frame including all headers, in bytes
frame.protocols   # All protocols in this packet, colon-separated (e.g., eth:ip:tcp:http)

# TLS
tls.handshake.extensions_server_name  # SNI — the hostname the client is connecting to
tls.handshake.ciphersuite             # The cipher suite negotiated for this session
```

### -E output control options

| Option | Values | Description |
|--------|--------|-------------|
| `header=y/n` | `y`, `n` | Print a header row with field names (default: `n`) |
| `separator=<char>` | `,` `\t` `/` | Column delimiter character (default: tab) |
| `quote=d/s/n` | `d`, `s`, `n` | Wrap values in double / single / no quotes |
| `occurrence=f/l/a` | `f`, `l`, `a` | When a field appears multiple times in one packet, show the first / last / all occurrences |

---

## Statistics

The `-z` flag gives access to TShark's built-in statistics engine. Always pair it with `-q` to suppress the per-packet output and show only the summary table.

### I/O statistics

```bash
tshark -r file.pcap -q -z io,stat,1
# Prints a table of packet count and byte count for each 1-second interval
# Good for visualising traffic spikes, bursts, and idle periods
```

```bash
tshark -r file.pcap -q -z io,stat,1,"ip.addr==192.168.1.1"
# Same table, but counts only traffic matching the display filter
# Useful for isolating one host's bandwidth consumption over time
```

```bash
tshark -r file.pcap -q -z "io,stat,5,tcp,udp,icmp"
# Three separate columns in 5-second intervals: TCP, UDP, and ICMP packet counts
# Quickly reveals which protocol dominates the traffic
```

### Protocol hierarchy

```bash
tshark -r file.pcap -q -z io,phs
# Prints a tree showing every protocol detected, with packet and byte counts at each level
# The first thing to run when you open an unknown capture file — gives an instant overview
```

### Conversations

A *conversation* is all the traffic exchanged between two specific endpoints. TShark counts packets, bytes, and duration for each unique pair.

```bash
tshark -r file.pcap -q -z conv,ip
# Lists all pairs of IP addresses that exchanged traffic
# Shows bytes and packets in each direction and total duration

tshark -r file.pcap -q -z conv,tcp
# Same at the TCP level — each entry is a unique (srcIP:port ↔ dstIP:port) flow
# Useful for seeing which specific services were used

tshark -r file.pcap -q -z conv,udp
# UDP conversations — particularly useful for DNS and media streaming analysis

tshark -r file.pcap -q -z conv,eth
# Ethernet-level conversations — useful for detecting broadcast storms or ARP floods
```

### Endpoints

```bash
tshark -r file.pcap -q -z endpoints,ip
# Lists every IP address seen in the capture with total bytes sent and received
# The fastest way to identify the top talkers

tshark -r file.pcap -q -z endpoints,tcp
# Every unique TCP endpoint (IP:port pair) with traffic volume

tshark -r file.pcap -q -z endpoints,eth
# Every unique MAC address seen, with traffic volume
```

### HTTP statistics

```bash
tshark -r file.pcap -q -z http,stat
# Summary of HTTP request and response counts grouped by status code
# Reveals error rates at a glance

tshark -r file.pcap -q -z http_req,tree
# Tree view of HTTP requests grouped by URI path
# Shows which endpoints were hit most frequently

tshark -r file.pcap -q -z http_srv,tree
# Tree view of HTTP activity grouped by server hostname
```

### DNS statistics

```bash
tshark -r file.pcap -q -z dns,tree
# Tree view of all DNS queries and responses
# Shows query names, types (A/AAAA/MX/etc.), and response codes
```

### Expert info

```bash
tshark -r file.pcap -q -z expert
# Prints all notes, warnings, and errors that TShark's expert system detected:
# retransmissions, out-of-order packets, duplicate ACKs, RST connections, etc.
# The fastest way to spot network problems without reading individual packets
```

### Flow graph

```bash
tshark -r file.pcap -q -z flow,tcp,network
# Prints a text-based sequence diagram of TCP flows
# Shows the order of SYN, SYN-ACK, data segments, and FIN for each connection
```

---

## Decryption

### TLS with a key log file

Chrome and Firefox export TLS session keys when the `SSLKEYLOGFILE` environment variable is set. TShark uses these keys to decrypt HTTPS traffic after the fact.

```bash
export SSLKEYLOGFILE=~/tls_keys.log
# Set this BEFORE launching the browser
# The browser appends a new line to this file for every TLS session it establishes
```

```bash
tshark -r capture.pcap \
  -o "tls.keylog_file:tls_keys.log" \
  -Y "http" \
  -T fields -e http.host -e http.request.uri
# Reads the capture, decrypts TLS using the key log,
# then extracts the decrypted HTTP host and URI from each request
```

### WPA / WPA2 Wi-Fi

```bash
tshark -r wifi.pcap \
  -o "wlan.enable_decryption:TRUE" \
  -o "wlan.wep_key1:wpa-pwd:MyPassword:MySSID" \
  -Y "http"
# Decrypts a WPA2-protected Wi-Fi capture using the network password and SSID
# Requires the complete 4-way EAPOL handshake to be present in the capture file
# Without the handshake, decryption is not possible
```

### IPsec

```bash
tshark -r ipsec.pcap \
  -o "esp.enable_decryption:TRUE" \
  -o "esp.sa:<SPI>,<proto>,<src>,<dst>,<enc_algo>,<enc_key>,<auth_algo>,<auth_key>"
# Decrypts IPsec ESP traffic given the Security Association parameters
# All SA parameters must match exactly what was negotiated during IKE
```

---

## Following Streams

Stream following reassembles all the raw bytes exchanged in a TCP or UDP conversation back into the original application-layer data, in the correct order.

### TCP stream

```bash
tshark -r file.pcap -q -z follow,tcp,ascii,0
# Reassembles TCP stream number 0 and prints the content as ASCII text
# Stream 0 is the first TCP conversation in the capture
# Data sent in each direction is shown separately

tshark -r file.pcap -q -z follow,tcp,hex,0
# Same reassembly, but output is raw bytes in hexadecimal
# Useful for binary protocols where ASCII output is unreadable

tshark -r file.pcap -q -z follow,tcp,ascii,0 -Y "http"
# Follows stream 0 but only considers packets that match the HTTP display filter
```

Find the stream index of a specific conversation first:

```bash
tshark -r file.pcap -T fields -e ip.src -e ip.dst -e tcp.stream | sort -u
# Lists all TCP streams with their source/destination IPs and their stream index number
# Use the index from the third column in the follow,tcp,ascii,<index> command above
```

### UDP stream

```bash
tshark -r file.pcap -q -z follow,udp,ascii,0
# Reassembles and displays the content of UDP stream 0
# Useful for DNS, TFTP, and other UDP-based protocols
```

---

## Exporting Objects

TShark can extract files that were transferred over the network directly from a capture file, without needing to follow and manually reassemble streams.

```bash
tshark -r file.pcap --export-objects http,/tmp/http_exports
# Extracts all files transferred over HTTP: images, HTML pages, scripts, downloads
# Each file is saved to /tmp/http_exports/ with an auto-generated filename

tshark -r file.pcap --export-objects smb,/tmp/smb_exports
# Extracts files transferred over SMB — Windows network file shares (older SMB1)

tshark -r file.pcap --export-objects smb2,/tmp/smb2_exports
# Extracts files from SMB2/SMB3 — the modern version used by Windows Vista and later

tshark -r file.pcap --export-objects tftp,/tmp/tftp_exports
# Extracts files from TFTP transfers — commonly used by network devices for firmware updates

tshark -r file.pcap --export-objects dicom,/tmp/dicom_exports
# Extracts DICOM medical imaging files transferred over the network
```

> [!tip]
> After exporting, run `file *` in the output directory to identify every file by its magic bytes, regardless of what extension TShark assigned. Malware and exfiltrated data are often disguised with wrong extensions.

---

## Common Use Cases

### Network troubleshooting

```bash
tshark -i eth0 -f "udp port 53" -Y "dns" \
  -T fields -e frame.time -e ip.src -e dns.qry.name -e dns.resp.addr
# Live monitor of DNS traffic
# Shows: timestamp | which host made the query | what domain was queried | what IP was returned
# Useful for diagnosing DNS resolution failures or spotting unusual lookups
```

```bash
tshark -i eth0 -Y "http.request" \
  -T fields -e frame.time -e ip.src -e http.host -e http.request.uri
# Live monitor of all outgoing HTTP requests
# Shows: timestamp | source IP | HTTP Host header | requested URI path
```

```bash
tshark -i eth0 -Y "tcp.flags.reset == 1" \
  -T fields -e frame.time -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport
# Shows every TCP RST packet in real time
# RSTs indicate abrupt terminations: refused connections, firewall drops, crashed services
```

```bash
tshark -r file.pcap -Y "tcp.analysis.retransmission || tcp.analysis.lost_segment"
# Finds all retransmitted packets and detected segment losses in a saved capture
# High counts indicate network congestion, packet loss, or an overloaded server
```

### Security monitoring

```bash
tshark -i eth0 -f "icmp" -q -z io,stat,1
# Monitors ICMP packet volume in 1-second intervals
# A sudden spike in ICMP packets is the primary indicator of an ICMP flood attack
```

```bash
tshark -i eth0 -Y "arp.duplicate-address-detected"
# Alerts on ARP packets where the same IP is claimed by two different MAC addresses
# This is the primary indicator of ARP spoofing / man-in-the-middle attacks on the local network
```

```bash
tshark -i eth0 -f "port 21 or port 23" -V | grep -iE "pass|user|login"
# Captures FTP (21) and Telnet (23) traffic, then greps for credential-related strings
# Both protocols transmit passwords in plaintext — anything matched here is a live credential
```

```bash
tshark -i eth0 -Y "dns && frame.len > 512"
# Flags DNS packets larger than 512 bytes
# Normal DNS queries are tiny; oversized DNS packets can indicate DNS tunneling —
# a technique used to exfiltrate data or maintain C2 channels by encoding data inside DNS
```

```bash
tshark -r file.pcap -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0" \
  -T fields -e ip.src -e ip.dst -e tcp.dstport | sort | uniq -c | sort -rn
# Counts how many unique destination ports each source IP sent SYN packets to
# A single IP hitting dozens or hundreds of ports in a short time is port scanning
```

```bash
tshark -r file.pcap -Y "http.authorization" \
  -T fields -e ip.src -e http.host -e http.authorization
# Extracts HTTP Basic Auth headers from a capture
# Values are base64-encoded but trivially decoded: echo 'VALUE' | base64 -d
```

### Bandwidth analysis

```bash
tshark -i eth0 -q -z io,stat,1
# Real-time throughput in 1-second intervals
# Shows packets/sec and bytes/sec — useful for baselining normal traffic and spotting anomalies
```

```bash
tshark -r file.pcap -q -z conv,ip | sort -k3 -rn | head -10
# Lists the top 10 IP pairs sorted by packet count
# Identifies the heaviest bandwidth consumers in a capture
```

### Automation and scripting

```bash
#!/bin/bash
# Captures 5 minutes of traffic on eth0,
# then extracts all HTTP requests to a CSV file for further analysis

tshark -i eth0 -a duration:300 -w /tmp/cap.pcap -q
# -q suppresses live output so the capture runs silently in the background

tshark -r /tmp/cap.pcap \
  -Y "http.request" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e http.host \
  -e http.request.uri \
  -E separator="," \
  -E header=y \
  > /tmp/http_log.csv
# Output: CSV file with columns — timestamp, source IP, HTTP host, URI path
# Ready to open in Excel, import into a database, or process with pandas
```

---

## Advanced Techniques

### Suppress ARP and broadcast noise

```bash
tshark -i eth0 -f "not arp and not broadcast"
# ARP and broadcast traffic is almost always irrelevant for application-layer analysis
# Excluding it at capture time reduces output clutter and the size of saved files
```

### Capture only new TCP connections

```bash
tshark -i eth0 -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0"
# Shows only the initial SYN packet of each new TCP connection
# Provides a high-level view of all services being contacted, without data stream noise
# Useful for mapping out what a host is trying to reach
```

### Extract all DNS queries (sorted and deduplicated)

```bash
tshark -r file.pcap -Y "dns.flags.response == 0" \
  -T fields -e frame.time -e ip.src -e dns.qry.name | sort -u
# Shows every unique DNS query in the capture: who asked, what they asked for, and when
```

### Count packets per protocol stack

```bash
tshark -r file.pcap -T fields -e frame.protocols | sort | uniq -c | sort -rn
# Counts packets per unique protocol stack (e.g., eth:ip:tcp:http)
# Gives a fast breakdown of traffic composition without running a full statistics pass
```

### Merge multiple PCAP files

```bash
mergecap -w merged.pcap file1.pcap file2.pcap file3.pcap
# Merges multiple capture files into one, sorted chronologically by timestamp
# mergecap is included with every Wireshark / TShark installation
```

### Lua scripting

```bash
tshark -r file.pcap -X lua_script:myscript.lua
# Loads a custom Lua dissector or tap script at runtime
# Allows adding new protocol decoders or custom statistics without recompiling TShark
```

---

## TShark vs tcpdump

Both tools capture network traffic using `libpcap` and read/write standard `.pcap` files. Their BPF capture filter syntax (`-f`) is identical. The core difference is the **depth of analysis** each tool performs on the captured packets.

| | TShark | tcpdump |
|---|---|---|
| **Engine** | libwireshark — full Wireshark dissection engine | libpcap only |
| **Packet output** | Decoded, human-readable protocol dissection | Basic one-line summary |
| **Protocol support** | 2000+ protocols fully decoded | Common protocols only (IP, TCP, DNS…) |
| **Capture filters** | BPF syntax via `-f` | BPF syntax |
| **Display / read filters** | Wireshark syntax via `-Y` — very powerful | Not available |
| **Field extraction** | Any field of any protocol via `-T fields -e` | Not available — requires manual `grep` / `awk` |
| **Built-in statistics** | Yes — conversations, endpoints, I/O, HTTP, DNS… | None |
| **Output formats** | text, JSON, PDML, EK, tabs | text, hex dump, raw pcap |
| **TLS decryption** | Yes — via SSLKEYLOGFILE | No |
| **WPA2 decryption** | Yes — password + SSID | No |
| **Stream reassembly** | Built-in via `-z follow,tcp` | Not supported |
| **Object export** | HTTP, SMB, TFTP, DICOM file extraction | Not supported |
| **Lua scripting** | Yes — custom dissectors and tap scripts | No |
| **Install size** | ~100 MB | ~1 MB (usually pre-installed) |
| **Performance** | Slower — deep protocol parsing has overhead | Very fast — minimal processing per packet |

### Same task, different tools

Capture HTTP traffic and display the requested URLs:

```bash
# tcpdump — saves raw bytes, you parse them yourself later
tcpdump -i eth0 -w out.pcap port 80

# TShark — decodes the protocol live and extracts exactly what you need
tshark -i eth0 -f "port 80" -Y "http.request" \
  -T fields -e ip.src -e http.host -e http.request.uri
# Prints a live feed of: source IP | destination hostname | requested path
```

### When to use which

**Use tcpdump when:**
- You need a quick capture on a remote server where Wireshark is not installed
- Minimal footprint and zero dependencies matter — containers, IoT, embedded systems
- You want to save a `.pcap` and open it later in Wireshark or TShark
- You need maximum capture performance with the lowest possible CPU overhead

**Use TShark when:**
- You want to analyse traffic directly on the command line without a GUI
- You need to extract specific fields into structured output (CSV, JSON) for scripting or a SIEM
- You need built-in statistics — conversations, bandwidth breakdown, protocol hierarchy
- You need to decrypt TLS, WPA2, or IPsec traffic
- You are building automated capture-and-analysis pipelines

> [!tip]
> A common production workflow combines both: use `tcpdump` for a fast, lightweight capture on the target machine (minimal overhead, no Wireshark needed), copy the `.pcap` to your workstation, then analyse it with TShark or Wireshark.

---

## Tips and Tricks

> [!tip]
> **Always use `-n` when performance matters.** By default, TShark resolves every IP to a hostname via DNS. On a live capture this adds significant latency and generates spurious DNS traffic. Add `-n` to disable all resolution and make output appear instantly.

> [!tip]
> **Use BPF and display filters together.** BPF (`-f`) reduces what is captured — saving disk space and CPU. The display filter (`-Y`) refines what is shown from the already-filtered data. Use both for maximum efficiency on high-traffic interfaces.

> [!tip]
> **Always use `-q` with `-z`.** When running statistics, `-q` silences the per-packet output so only the summary table is shown at the end. Without it, the statistics are buried after thousands of packet lines.

> [!tip]
> **`-V` and `grep` together are powerful.** Full verbose output is noisy on its own, but piped to `grep` it becomes a fast protocol-aware search tool:
> ```bash
> tshark -r file.pcap -V | grep -A5 "Domain Name:"
> # Shows every DNS domain name with 5 lines of context around each match
> ```

> [!tip]
> **Pipe field output directly to standard Unix tools.** TShark's `-T fields` output is line-oriented, making it easy to chain with `sort`, `uniq`, `cut`, `awk`, and `wc` for quick analysis without writing a script.

> [!important]
> - **Permissions on Linux:** capturing requires root or membership in the `wireshark` group. Run `sudo usermod -aG wireshark $USER` and re-login to avoid running TShark as root.
> - **BPF ≠ display filter syntax:** using Wireshark syntax with `-f` will produce a capture error. `-f` takes BPF; `-Y` takes Wireshark syntax.
> - **`-q` is required with `-z`:** without it, statistics are buried after thousands of packet lines.
> - **Key log file must exist before the browser starts:** set `SSLKEYLOGFILE` before launching Chrome or Firefox — keys for already-established sessions cannot be recovered retroactively.
> - **Ring buffer requires `-w`:** the `-b` options have no effect unless an output file is specified with `-w`.

---

## Quick Reference

```
LIVE CAPTURE
  tshark -i eth0                                    Capture on eth0, print to terminal
  tshark -i eth0 -c 100                             Stop after 100 packets
  tshark -i eth0 -a duration:30                     Stop after 30 seconds
  tshark -i eth0 -w out.pcap                        Save all packets to file (no terminal output)
  tshark -i eth0 -f "tcp port 80"                   BPF capture filter — HTTP only
  tshark -i eth0 -b filesize:100000 -b files:5 \
    -w /tmp/ring.pcap                               5-file rotating ring buffer, 100 MB each

READ FILE
  tshark -r file.pcap                               Read and print every packet
  tshark -r file.pcap -Y "http"                     Apply display filter on read
  tshark -r file.pcap -V                            Full protocol dissection for every packet
  tshark -r file.pcap -2 -R "http"                 Two-pass analysis with read filter

FIELD EXTRACTION
  tshark -r file.pcap -T fields \
    -e ip.src -e ip.dst -e tcp.dstport \
    -E header=y -E separator=","                   CSV output with header row

OUTPUT FORMAT
  tshark -r file.pcap -T json                       Full JSON output
  tshark -r file.pcap -T json | jq '.[].layers.ip' Extract IP layer from JSON

STATISTICS
  tshark -r file.pcap -q -z io,stat,1               I/O stats in 1-second intervals
  tshark -r file.pcap -q -z io,phs                  Protocol hierarchy tree
  tshark -r file.pcap -q -z conv,tcp                TCP conversations with byte counts
  tshark -r file.pcap -q -z endpoints,ip            Top IP endpoints by traffic volume
  tshark -r file.pcap -q -z expert                  All warnings, errors, and anomalies

STREAMS
  tshark -r file.pcap -q -z follow,tcp,ascii,0      Reassemble and print TCP stream 0

EXPORT OBJECTS
  tshark -r file.pcap --export-objects http,/tmp/   Extract all HTTP-transferred files

DECRYPTION
  tshark -r cap.pcap -o "tls.keylog_file:keys.log" Decrypt TLS with key log file
```

---

*Notes cover TShark 4.x on Linux. Most syntax is compatible with TShark 3.x and Windows.*
## 👥 Team

Maintained by **DUTH-SEC-TEAM** — Network Forensics group.
