| Field            | Value                                  |
| ---------------- | -------------------------------------- |
| **Category**     | `network-forensics / traffic-analysis` |
| **Platform**     | Linux                                  |
| **Language**     | tcpdump                                |
| **License**      | -                                      |
| **Version**      | 1.0                                    |
| **Authors**      | Panagiotis Michalitsios                |
| **Last updated** | 02/05/2026                             |
# 🔬 tcpdump — Πλήρης Οδηγός (Από Μηδέν)

> Αυτός ο οδηγός είναι δομημένος ιεραρχικά — από την αρχή ως το προχωρημένο επίπεδο.  
> Διάβασέ τον γραμμικά αν μαθαίνεις, ή χρησιμοποίησέ τον ως reference αν ξέρεις ήδη τα βασικά.

---

## 📚 Πίνακας Περιεχομένων

1. [[#🧠 Τι Είναι το tcpdump]]
2. [[#🌐 Βασικές Έννοιες Networks (Prerequisite)]]
3. [[#🚀 Πρώτα Βήματα]]
4. [[#⚙️ Flags — Επιλογές Εντολής]]
5. [[#🔍 Ανατομία Packet — Πώς Διαβάζεις την Έξοδο]]
6. [[#🧵 Φίλτρα (BPF Expressions)]]
7. [[#📦 Raw Byte Offset Filters]]
8. [[#📊 Output Format Options]]
9. [[#💾 Αποθήκευση & Ανάγνωση Αρχείων]]
10. [[#🩺 Troubleshooting Workflows]]
11. [[#🔐 Ασφάλεια & Monitoring]]
12. [[#⚡ Χρήσιμα One-liners]]
13. [[#📋 tcpdump vs Wireshark]]
14. [[#⚠️ Παγίδες & Σημαντικές Παρατηρήσεις]]

---

## 🧠 Τι Είναι το tcpdump

Το **tcpdump** είναι ένα command-line εργαλείο που "ακούει" τα packets που διέρχονται από ένα network interface του υπολογιστή σου — σαν στηθοσκόπιο για το δίκτυο.

**Χρησιμοποιείται για:**
- Troubleshoot γιατί κάτι δεν συνδέεται
- Να δεις τι traffic φεύγει/έρχεται
- Εντοπισμό security threats
- Επιβεβαίωση ότι ένας server λαμβάνει/στέλνει σωστά

```bash
# Η πιο απλή χρήση — δες ό,τι περνά από το δίκτυο
sudo tcpdump
```

> ⚠️ Χρειάζεται πάντα `sudo` για να ανοίξει network interfaces.

---

## 🌐 Βασικές Έννοιες Networks (Prerequisite)

Πριν χρησιμοποιήσεις το tcpdump, πρέπει να ξέρεις αυτά:

### Τι είναι ένα Packet

Κάθε επικοινωνία στο δίκτυο σπάει σε μικρά κομμάτια δεδομένων που λέγονται **packets**. Κάθε packet έχει:
- **Header** — πού πάει, από πού έρχεται, τι είδους είναι
- **Payload** — τα πραγματικά δεδομένα

### Πρωτόκολλα

| Πρωτόκολλο | Χρήση | Τύπος |
|------------|-------|-------|
| **TCP** | Web, SSH, email — αξιόπιστη επικοινωνία | Connection-based |
| **UDP** | DNS, video streaming — γρήγορη επικοινωνία | Connectionless |
| **ICMP** | Ping, error messages | Διαγνωστικό |
| **ARP** | "Ποιος έχει αυτή την IP;" — Layer 2 | Local network |

### IP και Port

- **IP address** = η διεύθυνση του υπολογιστή (π.χ. `192.168.1.5`)
- **Port** = η "πόρτα" της εφαρμογής (π.χ. `:80` = HTTP, `:443` = HTTPS, `:53` = DNS)
- Ένα packet πηγαίνει από `IP:port` → `IP:port`

### Network Interface_

Ο υπολογιστής σου έχει πολλά "αυτιά" για το δίκτυο:
- `eth0` / `ens3` — καλωδιακή σύνδεση
- `wlan0` / `en0` — WiFi
- `lo` — loopback (ο υπολογιστής μιλά με τον εαυτό του)
- `any` — όλα μαζί

---

## 🚀 Πρώτα Βήματα

### Βήμα 1: Δες τα interfaces σου

```bash
# Λίστα με όλα τα network interfaces
tcpdump -D
```

Παράδειγμα output:
```
1.eth0
2.wlan0
3.lo (Loopback)
4.any (Pseudo-device that captures on all interfaces)
```

### Βήμα 2: Πρώτο capture

```bash
# Capture σε όλα τα interfaces, δείξε μόνο 10 packets
sudo tcpdump -i any -c 10
```

### Βήμα 3: Κατανόησε την έξοδο

```
10:52:03.992138 IP 192.168.1.241.63019 > 192.168.1.1.53: UDP, length 36
^               ^  ^                    ^                  ^
Timestamp       Πρ Source IP:port       Dest IP:port       Πληροφορίες
```

---

## ⚙️ Flags — Επιλογές Εντολής

### Capture

| Flag | Παράδειγμα | Τι κάνει |
|------|-----------|----------|
| `-i` | `tcpdump -i any` | **Interface** — ποιο interface να ακούει. `any` = όλα |
| `-c` | `tcpdump -c 1000` | **Count** — σταμάτα μετά από N packets |
| `-w` | `tcpdump -w out.pcap` | **Write** — αποθήκευση σε αρχείο .pcap |
| `-r` | `tcpdump -r out.pcap` | **Read** — ανάγνωση από αρχείο |

### Εμφάνιση

| Flag | Παράδειγμα | Τι κάνει |
|------|-----------|----------|
| `-n` | `tcpdump -n` | Εμφάνισε raw IPs — χωρίς DNS reverse lookup |
| `-A` | `tcpdump -A port 80` | Εμφάνισε περιεχόμενο ως ASCII (μόνο HTTP) |
| `-X` | `tcpdump -X` | Εμφάνισε περιεχόμενο ως HEX |
| `-XX` | `tcpdump -XX` | HEX + ASCII μαζί |
| `-e` | `tcpdump -e` | Εμφάνισε MAC addresses (Ethernet layer) |
| `-p` | `tcpdump -p` | Promiscuous mode OFF — μόνο το δικό σου traffic |
| `-v` | `tcpdump -v` | Verbose — περισσότερες λεπτομέρειες |
| `-vv` | `tcpdump -vv` | Ακόμα περισσότερες |
| `-vvv` | `tcpdump -vvv` | Μέγιστες λεπτομέρειες (TTL, checksum κλπ.) |
| `-t` | `tcpdump -t` | Χωρίς timestamps |
| `-tttt` | `tcpdump -tttt` | Timestamps σε absolute format |

> 💡 **Tip:** Σχεδόν πάντα χρησιμοποίησε `-n`. Χωρίς αυτό, το tcpdump κάνει DNS lookups για κάθε IP — αργό και μπερδεμένο.

---

## 🔍 Ανατομία Packet — Πώς Διαβάζεις την Έξοδο

### UDP / DNS Packet

```
10:52:13.919782 IP 192.168.1.241.63019 > 192.168.1.1.53: 44000+ A? ask.metafilter.com. (36)
│               │  │                    │                  │       │                    │
│               │  └─ Source IP:port    └─ Dest IP:port    │       └─ DNS query (τύπος A)
│               └─ Πρωτόκολλο (IP)                         └─ DNS Transaction ID
└─ Timestamp (ώρα:λεπτά:δευτερόλεπτα.microseconds)

# DNS Response:
10:52:13.928894 IP 192.168.1.1.53 > 192.168.1.241.63019: 44000 2/0/0 CNAME metafilter.com., A 54.186.13.33 (80)
                                                                 └─────┘ └──────────────────────────────────┘
                                                                 2 answers / 0 authority / 0 additional     Τα αποτελέσματα
```

**Ανάλυση του DNS query:**
- `44000+` = DNS ID (το `+` σημαίνει recursion desired)
- `A?` = ερώτηση για A record (IPv4 address)
- `(36)` = μέγεθος packet σε bytes

> 📌 **Παράδειγμα από πραγματικό πρόβλημα:**  
> 3 queries στις 10:52:03, 10:52:08, 10:52:13 αλλά μόνο 1 response στην τρίτη.  
> **Διάγνωση:** Packet loss στο upstream link. Ο client retry-αρε ανά ~5 δευτερόλεπτα μέχρι να πετύχει.

### TCP Packet

```
11:36:26.353797 IP 192.168.1.241.45296 > 192.241.182.146.443: Flags [.], ack 2291349910, win 319, length 0
                                                               └──────┘   └──────────┘   └──────┘  └──────┘
                                                               TCP Flag   Ack   number     Window    Μέγεθος
                                                               (. = ACK)                size      payload
```

### TCP Flags — Λεζάντα

| Σύμβολο | Όνομα | Σημασία | Πότε το βλέπεις |
|---------|-------|---------|-----------------|
| `[S]` | SYN | Έναρξη σύνδεσης | Client → Server (1ο βήμα handshake) |
| `[S.]` | SYN-ACK | Αποδοχή σύνδεσης | Server → Client (2ο βήμα) |
| `[.]` | ACK | Επιβεβαίωση (χωρίς δεδομένα) | Παντού |
| `[P.]` | PSH+ACK | Αποστολή δεδομένων | Κατά τη μεταφορά data |
| `[F.]` | FIN+ACK | Κανονικό κλείσιμο | Τέλος σύνδεσης |
| `[R.]` | RST | Βίαιη διακοπή | Σφάλμα ή firewall block |
| `[R]` | RST | Απόρριψη | Port κλειστό ή crash |

### TCP 3-Way Handshake (πώς ανοίγει μια σύνδεση)

```
Client                          Server
  │                               │
  │──── [S] SYN ─────────────────►│   "Θέλω να συνδεθώ"
  │                               │
  │◄─── [S.] SYN-ACK ────────────│   "Εντάξει, συνδέσου"
  │                               │
  │──── [.] ACK ─────────────────►│   "Ωραία, ξεκινάμε"
  │                               │
  │    [ΣΥΝΔΕΣΗ ΕΔΡΑΣΘΗΚΕ]       │
```

---

## 🧵 Φίλτρα (BPF Expressions)

Το tcpdump χρησιμοποιεί **Berkeley Packet Filter (BPF)** — γράφεις το φίλτρο μετά τις επιλογές.

```bash
sudo tcpdump [επιλογές] [φίλτρο]
#                        ^
#                        Εδώ γράφεις τι θες να δεις
```

### Φίλτρα ανά Host

```bash
# Traffic από/προς συγκεκριμένο host
tcpdump host 192.168.1.1

# Μόνο traffic ΑΠΟ (source)
tcpdump src host 192.168.1.1

# Μόνο traffic ΠΡΟΣ (destination)
tcpdump dst host 192.168.1.1
```

### Φίλτρα ανά Port

```bash
# Συγκεκριμένο port (και source και destination)
tcpdump port 80

# Μόνο source port
tcpdump src port 1234

# Μόνο destination port
tcpdump dst port 443
```

### Φίλτρα ανά Πρωτόκολλο

```bash
tcpdump tcp      # Μόνο TCP
tcpdump udp      # Μόνο UDP
tcpdump icmp     # Μόνο ICMP (ping)
tcpdump arp      # Μόνο ARP
```

### Φίλτρα ανά Subnet

```bash
# Όλο το subnet
tcpdump net 192.168.1.0/24

# Traffic μεταξύ δύο subnets
tcpdump src net 10.0.0.0/8 and dst net 172.16.0.0/12

# Μόνο broadcast
tcpdump broadcast
```

### Συνδυασμός Φίλτρων

Χρησιμοποίησε `and`, `or`, `not` για να συνδυάσεις:

```bash
# HTTP traffic ΑΠΟ συγκεκριμένο host
tcpdump host 10.0.0.5 and port 80

# DNS ή HTTP
tcpdump port 53 or port 80

# Όλα ΕΚΤΟΣ από SSH
tcpdump not port 22

# TCP στο 443 από συγκεκριμένο subnet
tcpdump tcp and dst port 443 and src net 192.168.1.0/24
```

### Φίλτρα ανά TCP Flag

```bash
# Νέες συνδέσεις (SYN)
tcpdump 'tcp[tcpflags] & tcp-syn != 0'

# Μόνο καθαρό SYN (χωρίς ACK) = πρώτο βήμα handshake
tcpdump 'tcp[tcpflags] == tcp-syn'

# Βίαιες διακοπές (RST)
tcpdump 'tcp[tcpflags] & tcp-rst != 0'

# Κανονικό κλείσιμο (FIN)
tcpdump 'tcp[tcpflags] & tcp-fin != 0'

# SYN-ACK (απάντηση server)
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
```

> 💡 Βλέπεις πολλά `tcp-syn` χωρίς `tcp-syn|tcp-ack`; → πιθανό **SYN flood attack**.

---

## 📦 Raw Byte Offset Filters

Πρόσβαση σε συγκεκριμένα bytes μέσα στο packet — για προχωρημένο φιλτράρισμα.

```
πρωτόκολλο[offset:μέγεθος]  operator  τιμή
     ^          ^      ^
     tcp/udp    byte   πόσα bytes
     icmp/ip    index  να διαβάσεις
```

```bash
# DNS RCODE = 3 → NXDOMAIN (domain δεν υπάρχει)
# byte 11 του UDP payload = DNS flags, τα τελευταία 4 bits = RCODE
tcpdump 'udp[11]&0xf==3'

# DNS RCODE = 2 → SERVFAIL (server error)
tcpdump 'udp[11]&0xf==2'

# Οποιοδήποτε DNS error (RCODE != 0)
tcpdump 'udp[11]&0xf!=0'

# HTTP GET requests (0x47455420 = ASCII "GET ")
tcpdump 'tcp[20:4] == 0x47455420'

# HTTP POST requests
tcpdump 'tcp[20:4] == 0x504f5354'

# ICMP echo request (ping που στέλνεις)
tcpdump 'icmp[icmptype] == icmp-echo'

# ICMP echo reply (ping που λαμβάνεις)
tcpdump 'icmp[icmptype] == icmp-echoreply'

# Μεγάλα IP packets (> 1400 bytes)
tcpdump 'ip[2:2] > 1400'
```

**DNS RCODE τιμές:**

| RCODE | Σημασία |
|-------|---------|
| 0 | NOERROR — επιτυχία |
| 1 | FORMERR — λάθος format query |
| 2 | SERVFAIL — server απέτυχε |
| 3 | NXDOMAIN — domain δεν υπάρχει |
| 5 | REFUSED — server αρνήθηκε |

---

## 📊 Output Format Options

```bash
# Κανονική έξοδος (default)
tcpdump -i any port 80

# Verbose — TTL, TOS, IP ID, fragment info
tcpdump -v port 80

# Πολύ verbose
tcpdump -vv port 80

# Εξαιρετικά verbose (checksum validation κλπ.)
tcpdump -vvv port 80

# ASCII περιεχόμενο (για HTTP)
tcpdump -A port 80

# HEX μόνο
tcpdump -X port 80

# HEX + ASCII (το πιο πλήρες)
tcpdump -XX port 80
```

### Παράδειγμα `-XX` output

```
11:42:05.123456 IP 192.168.1.5.54321 > 93.184.216.34.80: Flags [S]
        0x0000:  4500 003c 1234 4000 4006 f3c8 c0a8 0105  E..<.4@.@.......
        0x0010:  5db8 d822 d431 0050 0000 0000 0000 0000  ]..".1.P........
        └────┘   └──────────────────────────────────────┘ └──────────────┘
        Offset   HEX bytes                                ASCII (. = non-printable)
```

### Timestamps

```bash
tcpdump -t      # Χωρίς timestamp
tcpdump -tt     # Unix epoch timestamp
tcpdump -ttt    # Delta από προηγούμενο packet
tcpdump -tttt   # Πλήρης ημερομηνία + ώρα
```

---

## 💾 Αποθήκευση & Ανάγνωση Αρχείων

### Αποθήκευση

```bash
# Απλή αποθήκευση
tcpdump -w capture.pcap

# Με όριο packets (ασφαλές για production)
tcpdump -c 10000 -w capture.pcap

# Rotating files — νέο αρχείο κάθε 10 λεπτά (600 sec)
tcpdump -w capture-%Y%m%d-%H%M%S.pcap -G 600

# Rotating + όριο μεγέθους 10MB ανά αρχείο
tcpdump -w capture-%Y%m%d-%H%M%S.pcap -G 600 -C 10

# Capture συγκεκριμένου host και αποθήκευση
tcpdump host 8.8.8.8 -w dns.pcap
```

### Ανάγνωση

```bash
# Διάβασε αρχείο
tcpdump -r capture.pcap

# Διάβασε με φίλτρο
tcpdump -r capture.pcap port 80

# Διάβασε verbose
tcpdump -r capture.pcap -vv
```

> 💡 **Best practice:** Πάντα `-w` για αποθήκευση → ανοίξε στο **Wireshark** για βαθιά ανάλυση.

---

## 🩺 Troubleshooting Workflows

### Workflow 1: Ο Server Δεν Ανταποκρίνεται

```bash
# Βήμα 1: Φτάνουν καθόλου packets στον server;
sudo tcpdump -n -i any dst port 8080
# Αν δεν βλέπεις τίποτα → firewall ή routing πρόβλημα

# Βήμα 2: Στέλνει απαντήσεις ο server;
sudo tcpdump -n -i any src port 8080
# Αν δεν βλέπεις τίποτα → ο server δεν ακούει ή κρεμάει

# Βήμα 3: Υπάρχουν TCP RST; (απόρριψη)
sudo tcpdump -n 'tcp[tcpflags] & tcp-rst != 0 and port 8080'
# RST από server → port κλειστό ή crash
# RST από client → timeout στον client
```

### Workflow 2: Αργό Network

```bash
# Βήμα 1: Αποθήκευσε traffic για ανάλυση
sudo tcpdump -n -w slow.pcap
# Άνοιξε στο Wireshark → Statistics → TCP Stream Graphs → Time Sequence
# Ψάξε για "flat lines" (= ο sender περιμένει ACK = packet loss)

# Βήμα 2: Υπάρχουν ICMP unreachable; (= κάτι block-άρει)
sudo tcpdump -n 'icmp and icmp[icmptype] == icmp-unreach'
```

### Workflow 3: DNS Δεν Δουλεύει

```bash
# Βήμα 1: Στέλνω queries; (βλέπω outgoing requests;)
sudo tcpdump -n -i any port 53

# Βήμα 2: Παίρνω απαντήσεις; (βλέπω responses;)
sudo tcpdump -n 'udp src port 53'

# Βήμα 3: Τι λάθη παίρνω;
sudo tcpdump -n 'udp[11]&0xf!=0'   # οποιοδήποτε error
sudo tcpdump -n 'udp[11]&0xf==2'   # SERVFAIL — DNS server problem
sudo tcpdump -n 'udp[11]&0xf==3'   # NXDOMAIN — domain δεν υπάρχει
```

---

## 🔐 Ασφάλεια & Monitoring

```bash
# Εντοπισμός port scan — πολλά SYN σε διαφορετικά ports από ίδια IP
sudo tcpdump -n 'tcp[tcpflags] == tcp-syn'

# Ύποπτα μεγάλα DNS queries — πιθανό DNS tunneling/exfiltration
sudo tcpdump -n 'udp port 53 and udp[10:2] > 100'

# ARP scanning — ποιος ψάχνει το network; (network discovery)
sudo tcpdump -n arp

# Cleartext credentials — HTTP, FTP, Telnet
sudo tcpdump -A -n 'port 21 or port 23 or port 80'

# Traffic από ύποπτο IP
sudo tcpdump -n src host 203.0.113.99

# Πολύ μεγάλα packets — πιθανό data exfiltration
sudo tcpdump -n 'ip[2:2] > 1400'
```

---

## ⚡ Χρήσιμα One-liners

```bash
# Top 10 IPs που στέλνουν traffic
sudo tcpdump -n -c 1000 | awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn | head -10

# Μέτρησε connections ανά destination port
sudo tcpdump -n -c 5000 'tcp[tcpflags] == tcp-syn' | awk '{print $7}' | cut -d: -f1 | sort | uniq -c | sort -rn

# Live παρακολούθηση HTTP Host headers
sudo tcpdump -A -n 'tcp dst port 80' | grep 'Host:'

# Δες πόσα bytes στέλνει κάθε IP
sudo tcpdump -n -q -c 10000 | awk '{print $3, $NF}' | sort | uniq -c | sort -rn
```

---

## 📋 tcpdump vs Wireshark

| | tcpdump | Wireshark |
|---|---------|-----------|
| Interface | Command-line | GUI |
| Real-time capture | ✅ | ✅ |
| Scripting / automation | ✅ | ❌ |
| Remote capture (SSH) | ✅ | Με plugin |
| Deep protocol analysis | Βασική | Προχωρημένη |
| TCP stream graphs | ❌ | ✅ |
| Follow TCP stream | ❌ | ✅ |
| Φίλτρα | BPF | BPF + display filters |
| Καλύτερο για | Γρήγορο debugging on-server | Λεπτομερή ανάλυση offline |

> 💡 **Η τέλεια ροή εργασίας:**  
> `sudo tcpdump -w capture.pcap` → άνοιξε στο **Wireshark** τοπικά

---

## ⚠️ Παγίδες & Σημαντικές Παρατηρήσεις

- **`sudo` πάντα** — το tcpdump χρειάζεται root για να ανοίξει network interfaces.
- **`-c` σε production** — χωρίς `-c`, ένα capture μπορεί να γεμίσει τον δίσκο σε λίγα λεπτά σε busy server.
- **`.pcap` = ευαίσθητα δεδομένα** — τα αρχεία μπορεί να περιέχουν passwords, tokens, cookies. Χειρίσου τα με προσοχή.
- **`-A` = μόνο plaintext** — σε HTTPS/SSH δεν βλέπεις τίποτα χρήσιμο. Χρησιμοποίησε `-w` + Wireshark.
- **`-i any` σε macOS δεν δουλεύει** — χρησιμοποίησε το συγκεκριμένο interface: `-i en0` (WiFi) ή `-i en1` (Ethernet).
- **`-n` σχεδόν πάντα** — χωρίς `-n`, το tcpdump κάνει reverse DNS για κάθε IP → αργό και μπερδεμένο όταν debuggάρεις DNS.
- **Single quotes για σύνθετα φίλτρα** — `tcpdump 'tcp[tcpflags] & tcp-syn != 0'` — τα single quotes προστατεύουν τους ειδικούς χαρακτήρες από το shell.

---

## 🏷️ Tags

#networking #tcpdump #packets #bpf #dns #tcp #udp #icmp #security #sysadmin #wireshark #troubleshooting #linux
