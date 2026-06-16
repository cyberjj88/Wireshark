# 🦈 Lab 02 — Wireshark & Network Traffic Analysis

## Watch me do the lab HERE
https://www.loom.com/share/bda5ed4ad2a14c2e9adbb22cc4c8e993

**Author:** Jair Smith
**Difficulty:** 🟡 Intermediate
**Estimated Time:** ⏱ 2–4 Hours (across multiple sessions)
**Estimated Cost:** 💲 $0 — Wireshark is permanently free
**Platform:** Local Machine or Azure VM

| Certification Alignment | Free Tools Required |
|---|---|
| CompTIA Network+ · Security+ · CySA+ | [Wireshark](https://wireshark.org) — free, open source, no account required |

| Career Relevance |
|---|
| Network Engineer · SOC Analyst · Cloud Security Engineer · Incident Responder |

---

## 📋 Overview

Networks carry every piece of data your organisation produces — emails, database queries, login credentials, file transfers, API calls. When something goes wrong — a service is unreachable, a user reports slow performance, a security alert fires — the network is almost always involved. **The only way to know what is actually happening on a network is to look at the packets.**

Wireshark is the industry-standard tool organisations use to do exactly that. It captures the raw data moving across a network interface and lets you inspect it at every layer — from the physical frame all the way up to the application payload. This lab builds the packet analysis skills that underpin network engineering, SOC analysis, and cloud security work.

---

## 🏗️ Architecture Diagram — How Wireshark Captures Traffic

```
┌──────────────────────────────────────────────────────────────────────┐
│                        THE INTERNET                                  │
│                                                                      │
│    google.com · example.com · DNS Servers · Web Servers             │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ Packets flow in both directions
                           ▼
┌──────────────────────────────────────────────────────────────────────┐
│                     YOUR LOCAL NETWORK                               │
│                                                                      │
│   Router / Gateway (e.g. 192.168.1.1)                               │
│         │                                                            │
│         │  Ethernet or Wi-Fi                                         │
│         ▼                                                            │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │              YOUR MACHINE (Capture Point)                   │   │
│   │                                                             │   │
│   │   Network Interface Card (NIC)                              │   │
│   │   ┌──────────────────────────────────────────────────────┐  │   │
│   │   │  PROMISCUOUS MODE — captures ALL packets on segment  │  │   │
│   │   └──────────────────────────────────────────────────────┘  │   │
│   │                        │                                    │   │
│   │                        ▼                                    │   │
│   │   ┌──────────────────────────────────────────────────────┐  │   │
│   │   │                  WIRESHARK                           │  │   │
│   │   │                                                      │  │   │
│   │   │  Packet List   ← Every frame captured in real time  │  │   │
│   │   │  Packet Detail ← Drill into each protocol layer     │  │   │
│   │   │  Packet Bytes  ← Raw hex and ASCII payload          │  │   │
│   │   │                                                      │  │   │
│   │   │  Display Filters  → dns · http · tcp · ip.addr==x   │  │   │
│   │   │  Follow TCP Stream → Full conversation view         │  │   │
│   │   │  Export → .pcapng  → Portfolio evidence             │  │   │
│   │   └──────────────────────────────────────────────────────┘  │   │
│   └─────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘

  PACKET LAYERS WIRESHARK DECODES:
  Layer 2 (Data Link) → Ethernet frames, MAC addresses
  Layer 3 (Network)   → IP addresses, routing
  Layer 4 (Transport) → TCP/UDP ports, handshakes, streams
  Layer 7 (App)       → DNS queries, HTTP requests, credentials
```

> **Key Concept:** Wireshark sits passively on your network interface and reads every packet that passes through. It does not send any traffic itself — it only listens. This is called **passive network monitoring**.

---

## 🎯 Objectives

By completing this lab, you will be able to:

- [x] Install and configure Wireshark on any operating system
- [x] Capture live network traffic on a real interface
- [x] Apply **display filters** to isolate traffic by protocol, IP, and port
- [x] Read and interpret a **TCP three-way handshake**
- [x] Identify **DNS queries and responses** at the packet level
- [x] Detect **cleartext credentials** in unencrypted HTTP traffic
- [x] Reconstruct a full network conversation using **Follow TCP Stream**
- [x] Save and export captures as portfolio evidence

---

## 💼 Why This Lab Matters

| Role | Direct Application |
|---|---|
| **Network Engineer** | Diagnose connectivity issues by seeing exactly where packets are dropped or delayed |
| **SOC Analyst** | Identify malicious traffic patterns, extract indicators of compromise from packet captures |
| **Cloud Security Engineer** | The mental model from Wireshark transfers directly to reading Azure Network Watcher and AWS VPC Flow Logs |
| **Help Desk** | Prove that a reported network issue is real and identify whether it is client-side or server-side |

---

## ✅ Prerequisites

- [ ] A computer running Windows, macOS, or Linux
- [ ] Admin/sudo rights to install software and capture packets
- [ ] A working internet connection
- [ ] A terminal (Command Prompt, Terminal, or Bash)

---

## 📖 Key Concepts — Read Before Starting

Understanding these concepts before touching Wireshark is what makes the exercises click. Read through once now — you will see each of these in action during the lab.

---

### 🔹 What is a Packet?
A packet is the fundamental unit of data on a network. When you load a webpage or send an email, the data does not travel as one whole piece — it is broken into hundreds or thousands of smaller packets. Each packet has a **header** (containing source IP, destination IP, and port number) and a **payload** (the actual data). Packets travel independently across the network and are reassembled at the destination. Wireshark captures and displays each individual packet.

---

### 🔹 What is a Network Protocol?
A protocol is a set of rules defining how data is formatted and transmitted. Different protocols handle different jobs:

| Protocol | Purpose | Default Port |
|---|---|---|
| **DNS** | Translates domain names to IP addresses | UDP 53 |
| **HTTP** | Transfers web content (unencrypted) | TCP 80 |
| **HTTPS** | HTTP with TLS encryption | TCP 443 |
| **TCP** | Reliable, ordered data delivery | — |
| **ICMP** | Ping and network diagnostics | — |

---

### 🔹 What is the TCP Three-Way Handshake?
Before two computers can exchange data over TCP, they perform a three-step connection setup:

```
Your Machine          Server
     │                  │
     │──── SYN ────────►│   "I want to connect."
     │                  │
     │◄─── SYN-ACK ─────│   "Request received. Connection accepted."
     │                  │
     │──── ACK ────────►│   "Confirmed. Ready to send data."
     │                  │
     │   [DATA FLOWS]   │
```

If you see **SYN but no SYN-ACK**, the server is unreachable or refusing the connection. If you see a **RST (Reset)** packet, the connection was forcibly closed. These two patterns are the most diagnostically valuable things to look for.

---

### 🔹 What is DNS?
DNS (Domain Name System) translates human-readable domain names like `google.com` into IP addresses like `142.250.80.46`. Every website visit, API call, and email triggers a DNS lookup first. In Wireshark you can see these queries and their responses in real time.

> 🔒 **Security relevance:** Unexpected DNS queries to unusual domains are often the **first sign of malware** calling home to a command and control (C2) server. SOC analysts hunt for this pattern daily.

---

### 🔹 HTTP vs HTTPS
| | HTTP | HTTPS |
|---|---|---|
| **Encryption** | None — plaintext | TLS encrypted |
| **Credentials visible in Wireshark?** | ✅ Yes — anyone on the network can read them | ❌ No — payload is encrypted |
| **Port** | 80 | 443 |

In Exercise C of this lab, you will see actual credentials in plaintext inside an HTTP capture. This demonstration is exactly why HTTPS became the standard for every site that handles sensitive data.

---

### 🔹 What is Promiscuous Mode?
Normally, a NIC only captures packets addressed to your machine. In **promiscuous mode**, it captures every packet on the network segment — including packets addressed to other machines. Wireshark enables promiscuous mode automatically when you start a capture.

> On modern **switched networks**, you will primarily see your own traffic plus broadcast traffic. On networks with **port mirroring** configured, you can see all traffic on the segment.

---

## 🔬 Step 1 — Install Wireshark

Go to [wireshark.org/download.html](https://wireshark.org/download.html) and download the installer for your OS. No account required, no trial period, no licence.

| OS | Download | Notes |
|---|---|---|
| **Windows** | Windows x64 Installer (.exe) | Accept all defaults. Install **Npcap** when prompted — required for packet capture |
| **macOS** | macOS Arm or Intel (.dmg) | If prompted about ChmodBPF — allow it. This gives Wireshark permission to access network interfaces |
| **Linux** | Package manager | See command below. Add yourself to the `wireshark` group to capture without root |

```bash
# Linux (Ubuntu/Debian)
sudo apt install wireshark

# Add yourself to the wireshark group (log out and back in after running this)
sudo usermod -aG wireshark $USER

# Verify installation
wireshark --version
```

---

## 🔬 Step 2 — Your First Capture

1. Open **Wireshark**.
2. On the welcome screen, you will see a list of network interfaces with wavy lines showing live traffic activity.
3. **Double-click your active interface** — Ethernet or Wi-Fi, pick the one with the most activity in the wave graph.
4. Wireshark starts capturing immediately — packets appear in the list in real time.
5. Open a browser and navigate to any website.
6. After 30 seconds, click the **red square Stop button** in the toolbar.

You now have a packet capture in memory containing every frame that passed through your interface during the capture window. The volume can be overwhelming — hundreds or thousands of packets in 30 seconds. This is exactly why display filters exist.

---

## 🔬 Step 3 — Essential Display Filters

Type filters into the **filter bar at the top** of the Wireshark window and press Enter. The packet list updates instantly to show only matching traffic.

> **Display filters vs Capture filters:**
> **Capture filters** are applied before capturing — they limit what gets recorded. **Display filters** are applied after capturing — they limit what you see without discarding anything. Always use display filters in this lab. They let you look at the same capture through multiple lenses without re-capturing. Apply `dns`, inspect it, remove it, apply `tcp` — the full capture is always underneath.

| Filter | What It Shows | When to Use |
|---|---|---|
| `dns` | All DNS queries and responses | Troubleshooting name resolution, identifying unusual domain lookups |
| `http` | Unencrypted HTTP traffic only | Finding cleartext data, debugging web apps without HTTPS |
| `tcp` | All TCP traffic | Starting point for any connectivity investigation |
| `tcp.flags.syn == 1` | TCP SYN packets — connection attempts | Seeing which hosts are trying to connect to what |
| `tcp.flags.reset == 1` | TCP RST packets — connection resets | Finding refused or forcibly closed connections |
| `icmp` | All ICMP traffic including ping | Verifying basic reachability between hosts |
| `ip.addr == 192.168.1.1` | All traffic to or from a specific IP | Isolating traffic for a single host in a busy capture |
| `ip.src == 10.0.0.5` | Traffic from a specific source IP only | Isolating outbound traffic from one host |
| `tcp.port == 443` | All HTTPS traffic | Identifying encrypted web traffic by port |
| `http.request` | HTTP GET and POST requests only | Finding web requests — spotting data exfiltration |

---

## 🔬 Step 4 — Guided Exercises

Work through each exercise in order. Each one builds on the previous and teaches a specific skill used in real-world network analysis.

---

### Exercise A — Capture a DNS Lookup

> **Before you start:** `nslookup` is a built-in command-line tool that manually performs a DNS lookup. You give it a domain name and it returns the IP address. It is useful because it lets you trigger a DNS query on demand — generating traffic that Wireshark can capture.
>
> Run `nslookup` on your **local machine in a separate terminal window** — NOT inside Wireshark. Wireshark is a capture and analysis tool only; it has no terminal.

**How to open your terminal:**
- **Windows:** Press `Win`, type `cmd`, press Enter → Command Prompt opens.
- **macOS:** Press `Cmd + Space`, type `Terminal`, press Enter.
- **Linux:** Press `Ctrl + Alt + T` or right-click the desktop → Open Terminal.

> **What is a DNS A Record?**
> An **A record** (Address record) maps a domain name to an IPv4 address. When you type `google.com` into a browser, your machine queries DNS for the A record for `google.com` — the response contains the IPv4 address to connect to. In Wireshark, A records appear as type `1` in the packet detail.

**Steps:**

1. In Wireshark, start a capture on your active interface — click the **blue shark fin icon** or double-click your interface name.
2. Open a separate terminal window on your local machine.
3. In the terminal, run:
   ```
   nslookup google.com
   ```
4. Confirm the terminal shows an IP address — the DNS lookup worked.
5. Return to Wireshark and click the **red square Stop button**.
6. In the filter bar, type `dns` and press Enter.
7. Find the **query packet** — look in the Info column for `Standard query A google.com`.
8. Find the **response packet** — look for `Standard query response A google.com`.
9. Click the response packet. In the **Packet Detail pane** (bottom half), expand **Domain Name System (response)** → expand **Answers** → confirm the A record IP matches what your terminal returned.

> **What you just saw:** Your machine sent a DNS query, the DNS server replied with an IP address, and your browser used that IP to make the actual connection. This invisible lookup happens before **every** website visit, API call, and email your machine sends.
>
> 🔒 **Security relevance:** Unexpected DNS queries to unusual domains in a capture are often the first indicator of malware calling home to a C2 server.

---

### Exercise B — Watch the TCP Three-Way Handshake

1. Start a capture on your active interface.
2. Open a browser and navigate to `http://example.com` — use HTTP (not HTTPS) to make the handshake easier to observe.
3. Stop the capture.
4. Run `nslookup example.com` in your terminal to get the server IP address.
5. Apply the filter: `tcp and ip.addr == [IP from nslookup]`
6. Locate three sequential packets:

| Packet | Flags | What It Means |
|---|---|---|
| **1st** | `SYN` | Your machine: *I want to connect. Here is my sequence number.* |
| **2nd** | `SYN, ACK` | The server: *Request received. Here is my sequence number. Connection accepted.* |
| **3rd** | `ACK` | Your machine: *Confirmed. Connection is open. Ready to send data.* |

> **Diagnostic patterns to know:**
> - **SYN with no SYN-ACK** → server is unreachable or refusing the connection
> - **RST (Reset) packet** → connection was forcibly closed by one side

---

### Exercise C — Spot Cleartext Credentials in HTTP

> ⚠️ **Educational exercise only.** Only capture on networks and systems you own or have explicit written permission to analyse. Never use this technique against systems you do not own.

1. Set up a test HTTP login form on your local machine, or use a dedicated test site that runs over HTTP (not HTTPS).
2. Start a capture.
3. Submit a login form with a **test username and password** — do not use real credentials.
4. Stop the capture.
5. Apply the filter: `http.request.method == POST`
6. Click the POST packet → in the Packet Detail pane, find the **HTML Form URL Encoded** layer.
7. Your test username and password will be visible in **plaintext**.

> **Why this matters:** Without TLS encryption, anyone on the network path between the client and server — your ISP, a coffee shop router, an attacker performing a man-in-the-middle attack — can read submitted credentials exactly as typed. Wireshark is how security teams prove this vulnerability exists and demonstrate it to stakeholders who resist implementing HTTPS.

---

### Exercise D — Follow a Full TCP Stream

1. Capture any HTTP traffic by navigating to an HTTP website.
2. Find any HTTP packet in the capture list.
3. **Right-click** the packet → **Follow** → **TCP Stream**.
4. Wireshark reassembles all packets from that connection into a readable conversation:
   - 🔴 **Red text** = your browser's request
   - 🔵 **Blue text** = the server's response

> **Why this matters:** Individual packets are fragments. The TCP stream view reconstructs the **complete conversation** — which is what incident responders need to understand what data was transferred, what commands were sent, and what the server returned. This is a core skill in forensic network analysis.

---

## 🔬 Step 5 — Save and Export Captures

Always save interesting captures. They are both evidence of your skills and the portfolio entries for this lab.

```bash
# Save a full capture
# File → Save As → .pcapng format (preferred over legacy .pcap)

# Export only packets matching your current display filter
# Apply your display filter first
# File → Export Specified Packets → Displayed

# Re-open a saved capture
# File → Open → select your .pcapng file

# Command-line capture with tshark (ships with Wireshark — useful for remote servers)
tshark -i eth0 -w capture.pcapng -c 1000
# -i   network interface to capture on
# -w   output file path
# -c   stop after this many packets
```

---

## ✅ Verification Checklist

| Skill | How to Verify |
|---|---|
| **DNS capture** | Apply `dns` filter → identify a query and response packet with matching transaction IDs |
| **TCP handshake** | Find three sequential packets with SYN, SYN-ACK, and ACK flags → explain what each means without notes |
| **Display filters** | Filter by IP address, port, and protocol from memory — you should not need to look these up |
| **Stream reconstruction** | Follow a TCP stream and read the full HTTP request and response as a conversation |
| **File management** | Save a capture, close Wireshark, reopen it, load the file — confirm all packets are present |

---

## 🔧 Troubleshooting

| Problem | Fix |
|---|---|
| No interfaces listed on Wireshark welcome screen | Npcap (Windows) or ChmodBPF (macOS) was not installed. Reinstall Wireshark and accept all prompts |
| "You don't have permission to capture on that device" (Linux) | Run `sudo usermod -aG wireshark $USER`, log out and back in, then retry |
| Capture is empty / no packets appearing | Confirm you selected the correct interface — pick the one with the active wave graph on the welcome screen |
| Can't find DNS packets after filtering | DNS may already be cached. Open a new terminal, run `nslookup newdomain.com` for a domain you haven't visited recently |
| POST packet not showing credentials | Confirm the site is HTTP (not HTTPS). HTTPS encrypts the payload — credentials will not be readable |
| TCP stream shows garbled text | The traffic is likely HTTPS/TLS encrypted. Use an HTTP site for Exercise D |
| `nslookup` command not found | Windows: use Command Prompt, not PowerShell ISE. Linux/macOS: install with `sudo apt install dnsutils` |

---

## 🔒 Security Notes

**Why packet analysis matters beyond troubleshooting:**

The same skill set used to diagnose a slow network is used to detect an attacker. Key security applications of what you built in this lab:

- **C2 Detection** — Malware beaconing to a command and control server generates DNS queries and TCP connections on a regular schedule. In a Wireshark capture or SIEM flow log, this looks like clockwork outbound connections to unusual domains.
- **Credential Theft Detection** — HTTP POST requests carrying credentials in plaintext are trivially captured on any network segment. Any organisation still running HTTP login forms is exposed to this.
- **Data Exfiltration** — Unusual volumes of outbound TCP traffic, especially to unknown external IPs, can indicate data being stolen. Following the TCP stream reveals what was sent.
- **Cloud transfer** — Azure Network Watcher, AWS VPC Flow Logs, and GCP Packet Mirroring are the cloud equivalents of Wireshark. The mental model you built here — packets, flows, protocols, ports — transfers directly to reading those logs.

---

## 💡 Key Concepts Reference

| Concept | Definition |
|---|---|
| **Packet** | A small unit of data with a header (routing info) and payload (content) that travels independently across the network |
| **Protocol** | A ruleset defining how data is formatted and transmitted for a specific purpose (DNS, HTTP, TCP, ICMP) |
| **TCP Three-Way Handshake** | SYN → SYN-ACK → ACK — the connection setup sequence required before data can flow over TCP |
| **DNS A Record** | Maps a domain name to an IPv4 address — type 1 in packet captures |
| **Promiscuous Mode** | NIC setting that captures all packets on the segment, not just those addressed to your machine |
| **Display Filter** | A post-capture filter applied in Wireshark that limits what you see without discarding captured data |
| **Capture Filter** | A pre-capture filter that limits what gets recorded — applied before starting a capture |
| **Follow TCP Stream** | A Wireshark feature that reassembles all packets from one connection into a readable conversation |
| **pcapng** | Packet Capture Next Generation — the preferred file format for saving Wireshark captures |
| **tshark** | Command-line version of Wireshark — useful for capturing on remote servers without a GUI |
| **RST (Reset)** | A TCP flag indicating a connection was forcibly closed — often a sign of a refused or broken connection |
| **C2 (Command and Control)** | Infrastructure used by attackers to send instructions to compromised machines — detectable via unusual DNS and TCP patterns |

---

## 📁 Repository Structure

```
lab-02-wireshark-network-analysis/
│
├── README.md                        # This file — full lab documentation
└── captures/                        # Save your .pcapng files here
    ├── exercise-a-dns-lookup.pcapng
    ├── exercise-b-tcp-handshake.pcapng
    ├── exercise-c-http-credentials.pcapng
    └── exercise-d-tcp-stream.pcapng
```

> 💼 **Portfolio tip:** Upload all four capture files to this repo alongside the README. In interviews, opening a pcapng file and walking through what you found demonstrates hands-on packet analysis skills more convincingly than any certification alone.

---

## 🔗 References

- [Wireshark Official Documentation](https://www.wireshark.org/docs/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Wireshark Sample Captures (practice files)](https://wiki.wireshark.org/SampleCaptures)
- [TCP/IP Illustrated — W. Richard Stevens](https://www.informit.com/store/tcp-ip-illustrated-volume-1-the-protocols-9780321336316)
- [Azure Network Watcher](https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-overview)
- [CompTIA CySA+ Exam Objectives](https://www.comptia.org/certifications/cybersecurity-analyst)

---

*Lab authored by **Jair Smith** · Network Analysis Series · Wireshark Lab 02*
