# Malicious Network Traffic Analysis – HTTP POST C2 Behavior

## Overview
This project documents a hands-on network traffic investigation performed on a real packet capture (PCAP) file containing both legitimate system traffic and malicious outbound communication. The goal was to identify suspicious and malicious behavior using packet-level evidence only, without relying on endpoint logs, SIEM alerts, or malware reverse engineering.

The analysis focuses on detecting abnormal outbound HTTP POST activity consistent with command-and-control (C2) communication.

---

## Capture Source (What This PCAP Is)
- **File:** `2025-06-13-traffic-analysis-exercise.pcap`
- **Source:** Public traffic analysis exercise dataset
- **Purpose of Capture:** Simulated enterprise network traffic containing a mix of:
  - Normal Windows system activity
  - Legitimate web traffic
  - Embedded malicious communication

This capture is designed to replicate a realistic investigation scenario where malicious traffic is hidden inside normal background noise rather than isolated or obvious.

---

## Tools Used
- Wireshark
- TCP stream reconstruction
- Endpoint and conversation statistics
- Manual protocol inspection (HTTP)

---

## Step-by-Step Investigation Process

### Step 1: Scope the Traffic
- Loaded the PCAP into Wireshark.
- Applied high-level filters to focus on application-layer traffic:

http || tls

- Identified internal host **10.6.13.133** as the primary source of outbound connections.

---

### Step 2: Establish a Legitimate Traffic Baseline
Before flagging anything as malicious, I identified known-good behavior.

Observed legitimate traffic including:
- Windows connectivity checks (`connecttest.txt`)
- Windows Update traffic:
- Domains such as `ctldl.windowsupdate.com`
- Predictable URI paths (`/msdownload/update/v3/...`)
- Valid Microsoft User-Agent strings
- Normal `GET` requests with consistent request patterns

This step was critical to avoid false positives and to understand how normal system traffic appeared in the same capture.

---

### Step 3: Identify Abnormal HTTP Behavior
After establishing a baseline, I focused on outbound HTTP POST traffic.

Key observations:
- Repeated HTTP POST requests originating from **10.6.13.133**
- Requests sent to external IP addresses not associated with known services
- POST requests occurred frequently and consistently over time

Applied focused filters such as:
Content-Type: application/octet-stream

This indicates raw binary data transfer rather than form submissions, API usage, or user-driven web activity.

---

### Step 5: Analyze Payload Characteristics
- Followed multiple TCP streams associated with the POST requests.
- Payloads appeared as large binary blobs with no readable structure.
- No HTML, JSON, or application-layer formatting was present.

This behavior is consistent with automated machine-to-machine communication, such as C2 beaconing or data exfiltration.

---

### Step 6: Analyze Server Responses
- Observed consistent `HTTP 200 OK` responses.
- No `403 Forbidden` or access-denied responses were present.

This suggests the remote server was intentionally accepting the traffic, aligning with expected behavior of active command-and-control infrastructure rather than blocked or misconfigured endpoints.

---

## What Makes This Traffic Malicious

The conclusion is based on **behavioral evidence**, not payload content:

- Persistent automated HTTP POST traffic
- Randomized URI paths
- Binary payload transmission
- No browser characteristics
- No legitimate application ownership
- Successful server responses indicating active communication

These indicators do not align with normal user activity or standard system processes.

---

## Final Assessment
The traffic analyzed in this capture demonstrates characteristics consistent with malicious outbound command-and-control communication. While no malware family attribution was attempted, the network evidence alone is sufficient to justify incident escalation in a real security operations environment.

---

## How This Would Be Used in a Real SOC
In a real-world scenario, this analysis would support:
- Host isolation and containment
- Endpoint investigation using EDR tools
- Network blocking of external infrastructure
- Creation of SIEM detections for repeated POST beaconing behavior

---

## Next Steps
Potential follow-up actions include:
- Correlating findings with endpoint telemetry
- Investigating process-level network ownership
- Building behavioral detections in SIEM platforms
- Performing memory or disk forensics on the affected host

---

## Disclaimer
This project is for educational and defensive security purposes only. No exploitation or offensive activity was performed.
