> GUI-based network protocol analyzer for capturing and interactively analyzing live or recorded network traffic.

---

# Metadata

| Field            | Value                                  |
| ---------------- | -------------------------------------- |
| **Category**     | `network-forensics / traffic-analysis` |
| **Platform**     | Linux / Windows / macOS                |
| **Language**     | C                                      |
| **License**      | GPL-2.0 (GPLv2)                        |
| **Version**      | 4.0.17 (latest)                        |
| **Authors**      | @T-Konstantinos                        |
| **Last updated** | 05-05-2026                             |

---

# Description

Wireshark is the most widely used network protocol analyzer. It allows capturing and analyzing traffic in real time or from saved `.pcap` / `.pcapng` files. It operates from the Data Link layer (Layer 2) up to the Application layer (Layer 7) of the OSI model. Core capabilities include deep packet inspection, stream reassembly, protocol decoding, object export, and VoIP analysis.

---

# Installation

## Windows

Install via the browser: https://www.wireshark.org/download.html

## Unix

```bash
# Linux / Debian-based
sudo apt install wireshark

# macOS
brew install --cask wireshark
```

> **Note:** On Linux, membership in the `wireshark` group is required to capture without root: `sudo usermod -aG wireshark $USER`

---

# Analysis

## Display Filters

> Applied **after** capture. They select which packets are visible from the already-recorded traffic. Located in the filter bar at the top of the Wireshark window.

### Syntax

```shell
protocol.field  operator  value
```

---

### Modifiers

The `any` / `all` modifiers apply to fields that appear multiple times in a packet:

**`all`** — **all** instances of the field must match the value.

> [!example]
> 
> ```lua
> all ip.addr != 1.1.1.1
> ```

**`any`** — at least **one** instance of the field must match the value.

> [!example] At least one IP (src or dst) is different
> 
> ```lua
> any ip.addr != 1.1.1.1
> ```

> `all` and `any` have higher precedence than comparison Operators.

---

### Operators

#### Priority

| Priority    | Operator                               | Example                  |
| ----------- | -------------------------------------- | ------------------------ |
| 1 (highest) | `@` At operator                        | `@browser.comment`       |
| 2           | `#` Layer operator                     | `ip.addr#2`              |
| 3           | `[]` Slice operator                    | `eth.src[0:3]`           |
| 4           | `{}` Arithmetic grouping               | `{tcp.srcport + 3}`      |
| 5           | `*` `/` `%` Multiply / Divide / Modulo | `frame.len * 2`          |
| 6           | `+` `-` Addition / Subtraction         | `frame.len - 54`         |
| 7           | `&` Bitwise AND                        | `tcp.flags & 0x02`       |
| 8           | `in` Membership                        | `tcp.port in {80,443}`   |
| 9           | `==` `!=` `>` `<` `>=` `<=` Comparison | `frame.len > 1000`       |
| 10          | `contains` `matches` Search & Match    | `http contains "pass"`   |
| 11          | `all` `any` Modifiers                  | `all ip.addr != 1.1.1.1` |
| 12          | `!` `not` Logical NOT                  | `!arp`                   |
| 13          | `&&` `and` Logical AND                 | `ip && tcp`              |
| 14          | `^^` `xor` Logical XOR                 | `ip ^^ tcp`              |
| 15 (lowest) | `\|` `or` Logical OR                   | `dns or icmp`            |

#### Comparison Operators

| Operator                 | Symbol   | Example               |
| ------------------------ | -------- | --------------------- |
| Equal                    | `eq, ==` | `ip.addr == 10.0.0.1` |
| Not Equal                | `ne, !=` | `tcp.port != 443`     |
| Greater Than             | `gt, >`  | `frame.len > 1000`    |
| Less Than                | `lt, <`  | `ip.ttl < 10`         |
| Greater than or Equal to | `ge, >=` | `frame.len >= 1000`   |
| Less than or Equal to    | `le, <=` | `frame.len <= 1000`   |

#### Arithmetic Operators

Allow you to perform **mathematical operations inside a filter** — instead of comparing against fixed values, you compare fields against each other with a calculation.

| Operator | Operation      | Example                                |
| -------- | -------------- | -------------------------------------- |
| `+`      | Addition       | `udp.dstport >= udp.srcport + 1`       |
| `-`      | Subtraction    | `frame.len - 20 > 100`                 |
| `*`      | Multiplication | `tcp.dstport >= 4 * {tcp.srcport + 3}` |
| `/`      | Division       | `frame.len / 2 > 500`                  |
| `%`      | Modulo         | `frame.number % 2 == 0`                |

> [!warning] **Use `{}` for grouping, not `()`:**
> 
> ```c
> tcp.dstport >= 4 * {tcp.srcport + 3}
> ```

> [!warning] Subtraction requires a space before the `-`:
> 
> **Wrong**
> 
> ```c
> frame.len-54 > 1000
> ```
> 
> **Correct**
> 
> ```c
> frame.len - 54 > 1000
> frame.len -54 > 1000
> ```

#### Logical Operators

| Operator | Symbol | Example                                          |                     |
| -------- | ------ | ------------------------------------------------ | ------------------- |
| and      | `&&`   | `ip.src == 192.168.1.5 &&` <br>`tcp.port == 443` | (left-associative)  |
| or       | \|     | `dns or icmp`                                    | (left-associative)  |
| xor      | `^^`   | `dns ^^ icmp`                                    | (left-associative)  |
| not      | `!`    | `!(arp or dns)`                                  | (right-associative) |

#### Bitwise Operators

> [!info] What is bitwise AND? Compares bits between two values. Returns `1` only where **both** have `1`:
> 
> ```
>    0000 1111   (0x0F)
>  & 0000 0010   (0x02)
>  -----------
>    0000 0010   -> true, the bit is set
> ```

|Operator|Syntax|Meaning|
|---|---|---|
|Bitwise AND|`&`, `bitand`, `bitwise_and`|Checks whether specific bits are set|

> **Note:** Only AND is currently supported. Works on integer fields and slices.

##### Usage in Wireshark

###### TCP Flags

Each TCP flag is a bit in the `tcp.flags` byte:

|Flag|Hex|Filter|
|---|---|---|
|FIN|`0x01`|`tcp.flags & 0x01`|
|SYN|`0x02`|`tcp.flags & 0x02`|
|RST|`0x04`|`tcp.flags & 0x04`|
|PSH|`0x08`|`tcp.flags & 0x08`|
|ACK|`0x10`|`tcp.flags & 0x10`|
|URG|`0x20`|`tcp.flags & 0x20`|
|SYN+ACK|`0x12`|`tcp.flags & 0x12`|

**Examples:**

```lua
# SYN flood (SYN without ACK)
tcp.flags & 0x02 && !(tcp.flags & 0x10)

# RST flood
tcp.flags & 0x04

# Locally administered MAC
eth.addr[0] & 0x0f == 2
```

###### Ethernet Address Masking

|Check|Filter|What it finds|
|---|---|---|
|Multicast|`eth.addr[0] & 0x01`|MACs with multicast bit set|
|Unicast|`!(eth.addr[0] & 0x01)`|Normal unicast frames|
|Locally administered|`eth.addr[0] & 0x02`|Spoofed or VM MAC|
|Globally unique|`!(eth.addr[0] & 0x02)`|Normal hardware MACs|
|Locally administered unicast|`eth.addr[0] & 0x0f == 2`|Spoofed + unicast|
|Broadcast|`eth.dst == ff:ff:ff:ff:ff:ff`|Broadcast frames|

> **Forensics tip:** Locally administered MACs (`& 0x02`) in a production network are a red flag — usually VMs, Docker containers, or MAC spoofing.

---

#### Slice Operator `[]`

> Extracts specific bytes from a byte array or string field.

##### Syntax

|Syntax|Meaning|
|---|---|
|`[i:j]`|Offset i, length j|
|`[i-j]`|From offset i to offset j (inclusive)|
|`[i]`|Only the byte at offset i|
|`[:j]`|From the beginning, length j|
|`[i:]`|From offset i to the end|

> Zero-based: the first byte is `[0]`.

##### Negative Offsets

```lua
frame[-1]      # last byte
frame[-4:]     # last 4 bytes
```

##### Concatenation

```lua
# Combining slices with a comma
ftp[1,3-5,9:] == 01:03:04:05:09:0a:0b
```

##### Examples

```lua
eth.src[0:3] == 00:00:83          # Vendor OUI check
http.content_type[0:4] == "text"  # Content type
frame[100-199] contains "wireshark"
frame[-4:] == 0.1.2.3             # Last 4 bytes
```

---

#### Layer Operator `#`

Allows you to focus on a **specific layer** of the protocol stack when a protocol appears multiple times in the same packet.

##### When is it needed?

In tunneling protocols such as **GRE, VXLAN, IPsec** — a packet may contain **two IP headers**:

```
┌─────────────────────┐
│ Ethernet Header     │
├─────────────────────┤
│ IP Header #1 (outer)│  ← ip.src#1
├─────────────────────┤
│ GRE/VXLAN Header    │
├─────────────────────┤
│ IP Header #2 (inner)│  ← ip.src#2
├─────────────────────┤
│ Payload             │
└─────────────────────┘
```

##### Syntax

```bash
# Specific layer
ip.addr#1 == 192.168.1.1      # outer IP
ip.addr#2 == 192.168.30.40    # inner IP

# Layer range (with slice syntax)
tcp.port#[2-4]                # layers 2, 3, or 4
```

> **Note:** Layers are counted from `1`, not from `0` as with slices.

---

#### At Operator `@`

Compares the **raw, undecoded** content of a field instead of the decoded string.

By prefixing the field name with an at sign (`@`), the comparison is done against the raw packet data for the field.

##### When is it needed?

A character string must be decoded from a source encoding during dissection. If there are decoding errors, the resulting string will usually contain replacement characters:

```bash
# This does not work if there are encoding errors
browser.comment == "string is ?????"

# This compares the raw bytes
@browser.comment == 73:74:72:69:6e:67:20:69:73:20:aa:aa:aa:aa
```

##### How to read the raw bytes

```
73:74:72:69:6e:67:20:69:73:20:aa:aa:aa:aa
 s  t  r  i  n  g     i  s    ?  ?  ?  ?
```

The `aa:aa:aa:aa` bytes cannot be decoded as a normal string.

---

#### Membership Operator `in {}`

Checks whether a field belongs to a **set of values**. A cleaner approach than multiple `or` expressions.

##### Comparison

```bash
# Verbose approach
tcp.port == 80 or tcp.port == 443 or tcp.port == 8080

# Clean approach with membership
tcp.port in {80, 443, 8080}
```

##### Syntax

```bash
# Discrete values
tcp.port in {80, 443, 8080}
http.request.method in {"HEAD", "GET"}

# Ranges with ..
tcp.port in {443, 4430..4434}
ip.addr in {10.0.0.5..10.0.0.9, 192.168.1.1..192.168.1.9}
frame.time_delta in {10..10.5}

# Mixed discrete + ranges
tcp.port in {80, 443, 8000..8080}
```

##### Forensics Examples

```bash
# Suspicious ports (common C2 ports)
tcp.port in {4444, 5555, 6666, 7777}

# Scanning detection on well-known ports
tcp.dstport in {21, 22, 23, 80, 443, 3389}

# Specific subnet range
ip.dst in {192.168.1.1..192.168.1.254}

# HTTP method analysis
http.request.method in {"POST", "PUT", "DELETE"}
```

---

#### Search and Match Operators

|Operator|Usage|Example|
|---|---|---|
|`contains`|Checks whether the field **contains** a value (string)|`http contains "login"`|
|`matches`|Checks whether the field matches a **pattern** (PCRE2, case-insensitive)|`dns.qry.name matches "\.ru$"`|

> **contains** — does not work on numbers/IPs, only on strings/bytes. **matches** — uses PCRE2 regex, case-insensitive by default.

##### Contains — Simple Search

Searches whether the field **contains** a value (string):

```c
http contains "password"
http contains "login"
dns contains "evil"
```

> **Limitation:** Does not work on numbers or IPs — only on strings/bytes.

---

##### Matches — Regex Search

Searches whether the field matches a **pattern** (PCRE2, case-insensitive):

```c
# Domain ending in .ru
dns.qry.name matches "\.ru$"

# Suspicious subdomains (very long = possible DNS tunneling)
dns.qry.name matches "[a-z0-9]{20,}"

# Passwords in URLs
http.request.uri matches "pass(word)?="

# Case-sensitive when needed
http.user_agent matches "(?-i)Mozilla"
```

---

##### PCRE2 Regex Cheatsheet

###### Anchors — Position in String

|Element|Meaning|Example|Matches|
|---|---|---|---|
|`^`|Start of string|`^evil`|"evil.com", "evil-host"|
|`$`|End of string|`\.ru$`|"example.ru", "site.ru"|
|`^...$`|Exact match|`^admin$`|"admin" only|

---

###### Character Classes — Character Types

|Element|Meaning|Example|Matches|
|---|---|---|---|
|`.`|Any character (except newline)|`a.c`|abc, a9c, a-c|
|`\.`|Literal dot (escaped)|`\.com`|.com|
|`[abc]`|One of the listed characters|`[abc]`|a, b, c|
|`[^abc]`|Any character except these|`[^0-9]`|letters, symbols|
|`[a-z]`|Lowercase letters|`[a-z]+`|abc, xyz|
|`[A-Z]`|Uppercase letters|`[A-Z]+`|ABC, XYZ|
|`[0-9]`|Digits|`[0-9]+`|0 through 9|
|`[a-z0-9]`|Lowercase letters or digits|`[a-z0-9]{5}`|abc12|
|`\d`|Digit|`\d+`|0-9|
|`\w`|Letter, digit, or underscore|`\w+`|hello_1|
|`\s`|Whitespace|`\s+`|space, tab|

---

###### Quantifiers — Repetitions

|Element|Meaning|Example|Matches|
|---|---|---|---|
|`*`|0 or more times|`ab*c`|ac, abc, abbc|
|`+`|1 or more times|`ab+c`|abc, abbc|
|`?`|0 or 1 time (optional)|`pass(word)?`|pass, password|
|`{n}`|Exactly n times|`[0-9]{4}`|1234|
|`{n,}`|At least n times|`[a-z]{20,}`|20+ characters|
|`{n,m}`|Between n and m times|`[a-z]{3,5}`|abc, abcd, abcde|

---

###### Groups & Alternation

|Element|Meaning|Example|Matches|
|---|---|---|---|
|`()`|Group|`(abc)+`|abc, abcabc|
|`a or b`|OR — one of the two|`cat or dog`|cat, dog|
|`(a or b)`|OR inside a group|`(\.ru or .cn)$`|.ru, .cn|

---

###### Flags & Modes

|Element|Meaning|Example|Note|
|---|---|---|---|
|_(default)_|Case-insensitive|`matches "mozilla"`|matches Mozilla, MOZILLA|
|`(?-i)`|Case-sensitive mode|`(?-i)Mozilla`|only "Mozilla"|
|`(?i)`|Case-insensitive (explicit)|`(?i)mozilla`|matches all cases|

---

### Functions

Functions transform fields before the operator is applied.

#### String Functions

|Function|Description|
|---|---|
|`upper(field)`|Converts a string to uppercase|
|`lower(field)`|Converts a string to lowercase|
|`string(field)`|Converts a non-string field (e.g. a number) to a string|
|`vals(field)`|Converts an integer/boolean to its corresponding label|
|`dec(field)`|Unsigned integer → decimal string|
|`hex(field)`|Unsigned integer → hexadecimal string|

```
# Case-insensitive comparison
lower(http.server) contains "apache"

# Only odd-numbered packets
string(frame.number) matches "[13579]$"

# Packets with PFCP Request
vals(pfcp.msg_type) contains "Request"
```

#### Numeric Functions

|Function|Description|
|---|---|
|`len(field)`|Returns the length in bytes (string or bytes)|
|`count(field)`|Number of times the field appears in the frame|
|`float(field)`|Converts to single precision float|
|`double(field)`|Converts to double precision float|
|`max(f1,...,fn)`|Returns the maximum value from the arguments|
|`min(f1,...,fn)`|Returns the minimum value from the arguments|
|`abs(field)`|Absolute value of a numeric field|

```
# DNS queries with a long name (possible tunneling)
len(dns.qry.name) > 40

# Frames with many DNS answers
count(dns.a) > 5
```


---

## Capture Filters

> Applied **before** capture. Configured in the initial window before clicking Capture. Traffic that does not match is not captured at all.

### Syntax

The general format is:

```
[direction] [type] [value]  [logical_operator]  [direction] [type] [value]
```

> BPF — Berkeley Packet Filter



#### 1. Type — What you are looking for

|Type|Example|
|---|---|
|`host`|`host 192.168.1.1`|
|`net`|`net 192.168.1.0/24`|
|`port`|`port 80`|
|`portrange`|`portrange 80-443`|

#### 2. Direction — From/To where

|Direction|Example|
|---|---|
|`src`|`src host 10.0.0.1`|
|`dst`|`dst port 443`|
|_(none)_|both src and dst|

### Combined Examples

```bash
# TCP traffic only
tcp

# Specific host
host 192.168.1.1

# Inbound to port 80 only
dst port 80

# From a specific host AND port
src host 10.0.0.5 and dst port 443

# Entire subnet
net 192.168.1.0/24

# Everything except SSH
not port 22

# TCP or UDP on port 53
port 53 and (tcp or udp)
```


### Capture vs Display Filter Comparison

| |Capture Filter|Display Filter|
|---|---|---|
|**When**|Before capture|After capture|
|**Syntax**|BPF|Wireshark syntax|
|**Change**|Cannot change after|Can change freely|

> **Remember:** Capture Filters use **BPF syntax** — different from Display Filters. You do not write `ip.addr ==` here, you write `host`.

### Macros
%% not studied  %%
## Statistics

> _COMING SOON_

---

# Use Cases in Network Forensics

- **Credential Harvesting** — analyze plaintext protocols (Telnet, FTP, HTTP) to extract credentials
- **DNS Tunneling Detection** — identify unusually long DNS queries using `len()`
- **Data Exfiltration** — detect large transfers via Conversations & `frame.len`
- **Attack Signature Recognition** — SYN floods, port scanning, brute force via TCP flags
- **Object Extraction** — extract files via `File → Export Objects`
- **VoIP Analysis** — extract and replay calls via `Telephony → VoIP Calls`

---

# Limitations & Caveats

- Cannot decode encrypted traffic (SRTP, TLS) without the private key.
- Applications such as Discord, Teams, and WhatsApp use SRTP/DTLS — audio is not accessible.
- Requires root / `CAP_NET_RAW` for live capture on Linux.
- On very large PCAPs (GB range) performance degrades — prefer TShark for CLI analysis.

---

# References

- [Wireshark Official Documentation](https://www.wireshark.org/docs/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Wireshark Sample Captures](https://wiki.wireshark.org/SampleCaptures)
- [PCRE2 Pattern Syntax](https://www.pcre.org/current/doc/html/pcre2pattern.html)

---

# Notes

- Right-click on any field → **Apply as Filter** to automatically generate a filter.
- Save frequently used filters with the **+** button next to the filter bar.
- For CLI analysis use **[[TShark]]** — same syntax as Display Filters.