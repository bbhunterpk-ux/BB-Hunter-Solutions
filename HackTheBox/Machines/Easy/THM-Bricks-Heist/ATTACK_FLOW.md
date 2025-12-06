# TryHackMe Bricks Heist - Attack Flow Diagrams

## 🎯 Complete Attack Chain

```
┌─────────────────────────────────────────────────────────────────┐
│                    BRICKS HEIST ATTACK FLOW                      │
└─────────────────────────────────────────────────────────────────┘

Phase 1: RECONNAISSANCE
═══════════════════════
┌──────────────┐
│ Add to hosts │
│ bricks.thm   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Nmap Scan   │
│  Ports: 22,  │
│  80, 443     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Web Enum     │
│ WordPress +  │
│ Bricks Theme │
└──────┬───────┘
       │
       ▼

Phase 2: VULNERABILITY IDENTIFICATION
═══════════════════════════════════════
┌──────────────┐
│ Research CVE │
│ CVE-2024-    │
│ 25600        │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Clone Exploit│
│ from GitHub  │
│ K3ysTr0K3R   │
└──────┬───────┘
       │
       ▼

Phase 3: EXPLOITATION
═══════════════════════
┌──────────────┐
│ Execute RCE  │
│ Exploit      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Verify RCE   │
│ whoami       │
│ → apache     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Get Reverse  │
│ Shell        │
│ nc -lvnp 1337│
└──────┬───────┘
       │
       ▼

Phase 4: POST-EXPLOITATION
═══════════════════════════
┌──────────────┐
│ Stabilize    │
│ Shell        │
└──────┬───────┘
       │
       ├─────────────────────┬─────────────────────┐
       │                     │                     │
       ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Find Hidden  │    │ Enumerate    │    │ Analyze      │
│ Flag File    │    │ Services     │    │ Miner        │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                    │                    │
       ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Q1: Flag     │    │ Q2: Process  │    │ Q4: Log File │
│ Content      │    │ Q3: Service  │    │ Q5: Wallet   │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                                │
                                                ▼
Phase 5: THREAT INTELLIGENCE
═══════════════════════════════
                                        ┌──────────────┐
                                        │ Research     │
                                        │ Wallet on    │
                                        │ Blockchain   │
                                        └──────┬───────┘
                                               │
                                               ▼
                                        ┌──────────────┐
                                        │ Q6: Threat   │
                                        │ Group        │
                                        └──────────────┘
```

---

## 🔍 Detailed Enumeration Flow

```
WEB ENUMERATION WORKFLOW
════════════════════════

Start
  │
  ▼
┌─────────────────────────────────┐
│ curl -I https://bricks.thm      │
│ Check HTTP headers              │
└────────────┬────────────────────┘
             │
             ▼
      ┌──────────────┐
      │ WordPress?   │
      └──────┬───────┘
             │ YES
             ▼
┌─────────────────────────────────┐
│ View Page Source                │
│ Look for:                       │
│ - meta generator                │
│ - theme paths                   │
│ - plugin paths                  │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ Identify Theme: Bricks          │
│ Version: 1.9.x (from assets)   │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ Research Vulnerabilities        │
│ searchsploit bricks             │
│ Google: "bricks theme rce"      │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ Found: CVE-2024-25600           │
│ Unauthenticated RCE             │
│ Endpoint: /wp-json/bricks/v1/   │
│           render_element        │
└─────────────────────────────────┘
```

---

## 🎭 Service Analysis Workflow

```
IDENTIFYING THE CRYPTO MINER
════════════════════════════

┌─────────────────────────────────┐
│ systemctl list-units            │
│ --type=service --state=running  │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ Suspicious Services Found:      │
│ ✓ badr.service                  │
│ ✓ ubuntu.service                │
└────────────┬────────────────────┘
             │
             ├──────────────────────┬──────────────────────┐
             │                      │                      │
             ▼                      ▼                      │
┌──────────────────────┐  ┌──────────────────────┐       │
│ systemctl status     │  │ systemctl status     │       │
│ badr.service         │  │ ubuntu.service       │       │
└──────────┬───────────┘  └──────────┬───────────┘       │
           │                         │                    │
           ▼                         ▼                    │
┌──────────────────────┐  ┌──────────────────────┐       │
│ Process: badr        │  │ Process:             │       │
│ Files deleted after  │  │ nm-inet-dialog       │       │
│ 10 seconds (DECOY!)  │  │ (ACTUAL MINER!)      │       │
└──────────────────────┘  └──────────┬───────────┘       │
                                     │                    │
                                     ▼                    │
                          ┌──────────────────────┐       │
                          │ ps aux | grep        │       │
                          │ nm-inet              │       │
                          └──────────┬───────────┘       │
                                     │                    │
                                     ▼                    │
                          ┌──────────────────────┐       │
                          │ Find config/log:     │       │
                          │ /usr/lib/            │       │
                          │ NetworkManager/      │       │
                          │ inet.conf            │       │
                          └──────────────────────┘       │
                                                          │
                                                          ▼
                                              ┌──────────────────────┐
                                              │ ANSWERS:             │
                                              │ Q2: nm-inet-dialog   │
                                              │ Q3: ubuntu.service   │
                                              │ Q4: inet.conf        │
                                              └──────────────────────┘
```

---

## 🔓 Wallet Extraction Process

```
DECODING THE WALLET ADDRESS
════════════════════════════

┌─────────────────────────────────────────────────────────┐
│ cat /usr/lib/NetworkManager/inet.conf                   │
│ grep "ID:"                                              │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│ Hex String Found:                                       │
│ 5757314e65474e5962484a4f656d787457544e424e5746...      │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│ Step 1: Hex Decode                                      │
│ echo "5757..." | xxd -r -p | base64 -d                 │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│ Base64 String:                                          │
│ YmMxcXlrNzlmY3A5aGQ1a3JlcHJjZTg5dGtoNHdydGw4...      │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│ Step 2: Base64 Decode                                   │
│ echo "YmMx..." | base64 -d                              │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│ Bitcoin Wallet Address:                                 │
│ bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa            │
└─────────────────────────────────────────────────────────┘
```

---

## 🕵️ Threat Intelligence Workflow

```
IDENTIFYING THE THREAT GROUP
═════════════════════════════

┌─────────────────────────────────┐
│ Wallet Address:                 │
│ bc1qyk79fcp9hd5kreprce89tkh4... │
└────────────┬────────────────────┘
             │
             ├──────────────────────┬──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐
│ Blockchain Explorer  │  │ Google Search        │  │ Threat Intel DB      │
│ blockchain.com       │  │ "wallet + threat"    │  │ MITRE ATT&CK         │
│ blockchair.com       │  │ "wallet + malware"   │  │ AlienVault OTX       │
└──────────┬───────────┘  └──────────┬───────────┘  └──────────┬───────────┘
           │                         │                         │
           └─────────────────────────┴─────────────────────────┘
                                     │
                                     ▼
                          ┌──────────────────────┐
                          │ Cross-reference:     │
                          │ - Transaction history│
                          │ - Known campaigns    │
                          │ - IOCs               │
                          └──────────┬───────────┘
                                     │
                                     ▼
                          ┌──────────────────────┐
                          │ Threat Group         │
                          │ Identified           │
                          └──────────────────────┘
```

---

## 📊 Timeline of Attack

```
ATTACK TIMELINE
═══════════════

T+0:00  │ Initial Reconnaissance
        │ ├─ Add target to /etc/hosts
        │ ├─ Nmap scan
        │ └─ Web enumeration
        │
T+0:15  │ Vulnerability Research
        │ ├─ Identify WordPress + Bricks
        │ ├─ Research CVE-2024-25600
        │ └─ Clone exploit from GitHub
        │
T+0:30  │ Exploitation
        │ ├─ Execute RCE exploit
        │ ├─ Verify command execution
        │ └─ Obtain reverse shell
        │
T+0:45  │ Post-Exploitation
        │ ├─ Stabilize shell
        │ ├─ Enumerate web directory
        │ └─ Find hidden flag file
        │     └─ Answer Q1 ✓
        │
T+1:00  │ Service Enumeration
        │ ├─ List running services
        │ ├─ Investigate badr.service (decoy)
        │ ├─ Investigate ubuntu.service (miner)
        │ └─ Identify nm-inet-dialog process
        │     ├─ Answer Q2 ✓
        │     └─ Answer Q3 ✓
        │
T+1:15  │ Malware Analysis
        │ ├─ Locate configuration files
        │ ├─ Find inet.conf log file
        │ └─ Extract encoded wallet
        │     └─ Answer Q4 ✓
        │
T+1:30  │ Data Decoding
        │ ├─ Hex decode
        │ ├─ Base64 decode
        │ └─ Extract wallet address
        │     └─ Answer Q5 ✓
        │
T+1:45  │ Threat Intelligence
        │ ├─ Research wallet on blockchain
        │ ├─ Cross-reference with threat DBs
        │ └─ Identify threat group
        │     └─ Answer Q6 ✓
        │
T+2:00  │ Challenge Complete! 🎉
```

---

## 🎯 Decision Tree

```
EXPLOITATION DECISION TREE
══════════════════════════

                    ┌─────────────┐
                    │ Start Enum  │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Web Server? │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │             │
                   YES           NO
                    │             │
                    ▼             ▼
            ┌─────────────┐  ┌─────────────┐
            │ WordPress?  │  │ Try other   │
            └──────┬──────┘  │ services    │
                   │         └─────────────┘
            ┌──────┴──────┐
            │             │
           YES           NO
            │             │
            ▼             ▼
    ┌─────────────┐  ┌─────────────┐
    │ Bricks      │  │ Check other │
    │ Theme?      │  │ themes/     │
    └──────┬──────┘  │ plugins     │
           │         └─────────────┘
          YES
           │
           ▼
    ┌─────────────┐
    │ Version     │
    │ ≤ 1.9.6?    │
    └──────┬──────┘
           │
          YES
           │
           ▼
    ┌─────────────┐
    │ Exploit     │
    │ CVE-2024-   │
    │ 25600       │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │ RCE Success!│
    └─────────────┘
```

---

## 🔄 Iterative Enumeration Process

```
POST-EXPLOITATION ENUMERATION LOOP
═══════════════════════════════════

    ┌──────────────────────────────────────┐
    │                                      │
    │  ┌────────────────────────────┐     │
    │  │ 1. Basic System Info       │     │
    │  │    - whoami, id, hostname  │     │
    │  └────────────┬───────────────┘     │
    │               │                      │
    │               ▼                      │
    │  ┌────────────────────────────┐     │
    │  │ 2. Network Info            │     │
    │  │    - ip a, netstat         │     │
    │  └────────────┬───────────────┘     │
    │               │                      │
    │               ▼                      │
    │  ┌────────────────────────────┐     │
    │  │ 3. Running Processes       │     │
    │  │    - ps aux                │     │
    │  └────────────┬───────────────┘     │
    │               │                      │
    │               ▼                      │
    │  ┌────────────────────────────┐     │
    │  │ 4. Services                │     │
    │  │    - systemctl list-units  │     │
    │  └────────────┬───────────────┘     │
    │               │                      │
    │               ▼                      │
    │  ┌────────────────────────────┐     │
    │  │ 5. Suspicious Findings?    │     │
    │  └────────────┬───────────────┘     │
    │               │                      │
    │        ┌──────┴──────┐              │
    │        │             │              │
    │       YES           NO              │
    │        │             │              │
    │        ▼             └──────────────┘
    │  ┌────────────────────────────┐
    │  │ 6. Deep Dive Analysis      │
    │  │    - Check configs         │
    │  │    - Analyze binaries      │
    │  │    - Extract artifacts     │
    │  └────────────────────────────┘
    │
    └─► Continue until all questions answered
```

---

**Created by:** Amazon Q  
**Challenge:** TryHackMe - Bricks Heist  
**Date:** December 2025
