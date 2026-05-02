| Field            | Value                                  |
| ---------------- | -------------------------------------- |
| **Category**     | `network-forensics / traffic-analysis` |
| **Platform**     | Linux                                  |
| **Language**     | tcpdump                                |
| **License**      | -                                      |
| **Version**      | 1.0                                    |
| **Authors**      | Panagiotis Michalitsios                |
| **Last updated** | 02/05/2026                             |
# 🔬 tcpdump — Σημειώσεις

> **tcpdump** = command-line packet analyzer (packet sniffer) για ανάλυση TCP/IP traffic σε network interfaces.  
> Χρησιμοποιείται για: troubleshoot network issues · ανάλυση traffic · εντοπισμό security threats.

---

## ⚙️ Βασικές Επιλογές (Flags)

### Capture

| Flag | Χρήση | Περιγραφή |
|------|-------|-----------|
| `-i` | `tcpdump -i any` | Interface — ποιο network interface να παρακολουθεί. `any` = όλα |
| `-c` | `tcpdump -c 10000` | Count — σταμάτα μετά από N packets (προστατεύει τον δίσκο) |
| `-w` | `tcpdump -w capture.pcap` | Write — αποθήκευση σε αρχείο για ανάλυση στο Wireshark |

### Εμφάνιση

| Flag | Χρήση | Περιγραφή |
|------|-------|-----------|
| `-n` | `tcpdump -n -i any` | Εμφάνισε IPs ως έχουν (χωρίς DNS reverse lookup) |
| `-A` | `tcpdump -A dest port 7777` | Εμφάνισε περιεχόμενο packet ως ASCII — μόνο HTTP, όχι HTTPS |
| `-e` | `tcpdump -e -i any port 443` | Εμφάνισε Ethernet info (MAC addresses) |
| `-p` | `tcpdump -p -i eth0` | Απενεργοποίησε promiscuous mode — μόνο traffic του δικού σου μηχανήματος |

---

## 🧪 Συνταγές (Recipes)

```bash
# Έρχονται packets στον server μου;
tcpdump -i any port 1337

# Τι DNS queries στέλνει ο υπολογιστής μου;
tcpdump -n -i any port 53

# Εμφάνισε μόνο ΑΠΟΤΥΧΗΜΕΝΑ DNS queries (RCODE=3, NXDOMAIN)
tcpdump 'udp[11]&0xf==3'

# Αποθήκευσε packets για ανάλυση στο Wireshark
tcpdump -w packets.pcap

# Ασφαλής καταγραφή HTTP requests (μέχρι 5000 packets)
tcpdump -A -c 5000 -w req.pcap dest port 8080

# Traffic μόνο από/προς συγκεκριμένο host
tcpdump host 8.8.8.8 -w dns.pcap
```

---

## 🔍 Ανατομία Packet

### UDP / DNS Packet

```
10:52:13.919782 IP 192.168.1.241.63019 > 192.168.1.1.53: 44000+ A? ask.metafilter.com. (36)
│               │  │                    │                  │       │
│               │  └─ Source IP:port    └─ Dest IP:port    │       └─ DNS query
│               └─ Protocol                                └─ DNS ID (44000+)
└─ Timestamp

10:52:13.928894 IP 192.168.1.1.53 > 192.168.1.241.63019: 44000 2/0/0 CNAME metafilter.com., A 54.186.13.33 (80)
                                                                 └─────┘
                                                                 2 answers / 0 authority / 0 additional
```

**Παρατήρηση:** 3 queries (10:52:03, 10:52:08, 10:52:13) αλλά μόνο 1 response → **packet loss** στο upstream link, ο client retry-αρε ανά ~5 δευτερόλεπτα.

### TCP Packet

```
11:36:26.353797 IP 192.168.1.241.45296 > 192.241.182.146.443: Flags [.], ack 2291349910, win 319, length 0
                                                               └────────┘
                                                               TCP Flag (. = ACK)
```

### TCP Flags — Λεζάντα

| Flag | Σύμβολο | Σημασία |
|------|---------|---------|
| SYN | `[S]` | Έναρξη σύνδεσης |
| ACK | `[.]` | Επιβεβαίωση (χωρίς δεδομένα) |
| PSH+ACK | `[P.]` | Αποστολή δεδομένων |
| FIN+ACK | `[F.]` | Κλείσιμο σύνδεσης |
| RST | `[R.]` | Βίαιη διακοπή σύνδεσης |

---

💡 Σημειώσεις

- Το `udp[11]&0xf==3` είναι **raw byte offset filter** — byte 11 του UDP payload = DNS flags field. Masking με `0xf` δίνει το RCODE. RCODE 3 = NXDOMAIN (domain δεν βρέθηκε).
- Πάντα χρησιμοποίησε `-n` όταν κάνεις debug DNS — αλλιώς το tcpdump κάνει κι αυτό DNS lookups και μπερδεύει τα αποτελέσματα.
- Για HTTP body inspection, το `ngrep` είναι πιο βολικό από το `tcpdump -A`.
- Τα `.pcap` αρχεία ανοίγουν με **Wireshark** για γραφική ανάλυση (connection duration, retransmits, RTT).

---

## 🏷️ Tags

#networking #tcpdump #packets #dns #tcp #linux #sysadmin
