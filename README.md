# 🛡️ Zeek & Suricata IDS Lab – Detecting a Compromised Spam Host

## 📌 Project Overview

This lab demonstrates detection of a compromised internal host performing mass outbound spam activity using:

- **Zeek** – Behavioral network analysis  
- **Suricata** – Signature-based intrusion detection  

The objective was to analyze a malicious PCAP file and identify abnormal SMTP behavior consistent with spam bot activity using layered IDS methodology.

---

## 🧰 Tools & Environment

- Ubuntu 22.04 (VirtualBox)
- Zeek 8.x
- Suricata 6.x
- tcpdump
- jq (JSON parsing)

---

## 📂 Dataset

**PCAP analyzed:**

```
sf19us-MTA-lab-16.pcap
```

**Scenario:**  
Internal host suspected of malicious outbound email activity.

---

# 🔍 Investigation Methodology

## 1️⃣ Behavioral Detection – Zeek

Zeek was used to extract structured network metadata:

```bash
zeek -r sf19us-MTA-lab-16.pcap
```

Primary logs analyzed:

- conn.log  
- dns.log  
- smtp.log  
- ssl.log  

---

# 🔎 Key Findings (Zeek Analysis)

## 🔹 Single Infected Internal Host

```bash
cat conn.log | zeek-cut id.orig_h | sort | uniq
```

**Result**

```
10.5.6.105
```

Only one internal system generated abnormal outbound traffic.

---

## 🔹 Abnormal SMTP Volume

```bash
cat conn.log | zeek-cut id.resp_p | sort | uniq -c | sort -nr
```

**Result**

```
230 587
212 25
47  465
```

This indicates high-volume outbound email activity across multiple SMTP ports.

---

## 🔹 279 Unique SMTP Destinations

```bash
cat conn.log | zeek-cut id.resp_h id.resp_p | grep -E "25|587|465" | sort | uniq | wc -l
```

**Result**

```
279
```

A normal workstation typically connects to 1–2 mail servers.  
This host contacted **279 unique external mail servers**, indicating automated spam behavior.

---

## 🔹 Top SMTP Targets

```bash
cat conn.log | zeek-cut id.resp_h id.resp_p | grep -E "25|587|465" | sort | uniq -c | sort -nr | head -20
```

Example output:

```
50  74.208.5.13  25
21  74.208.5.13  587
9   196.25.211.150 25
6   212.227.15.188 587
5   64.233.171.109 587
```

This confirms repeated outbound delivery attempts to global mail infrastructure providers.

---

# 🛡️ Signature Detection – Suricata

Suricata was executed against the same PCAP:

```bash
sudo suricata -r sf19us-MTA-lab-16.pcap \
  -S /var/lib/suricata/rules/suricata.rules \
  -l suri-logs
```

---

## 🔎 Alert Extraction

```bash
cat suri-logs/eve.json | jq '.alert.signature' | sort | uniq -c
```

**Result**

```
202  "SURICATA Applayer Detect protocol only one direction"
8    "SURICATA SMTP invalid reply"
```

---

## 🚨 Alert Interpretation

Suricata detected:

- One-direction SMTP sessions  
- Invalid SMTP replies  
- Abnormal mail protocol behavior  

These findings align with Zeek’s behavioral detection of automated spam activity.

---

# 🧠 Final Assessment

Internal host:

```
10.5.6.105
```

Exhibited:

- Mass outbound SMTP connections  
- 279 unique mail server targets  
- Encrypted SMTP submissions (587 / 465)  
- SMTP protocol anomalies  
- Automated mail behavior consistent with spam bot activity  

### 🚨 Incident Classification:
Compromised host participating in outbound spam distribution.

---

# 🧩 Detection Correlation Summary

| Tool      | Detection Type        | Findings |
|-----------|----------------------|----------|
| Zeek      | Behavioral analysis  | Abnormal SMTP volume & distribution |
| Suricata  | Signature detection  | SMTP protocol anomalies |

This demonstrates layered IDS methodology and cross-tool validation.

---

# 🚀 Skills Demonstrated

- PCAP network traffic analysis  
- Zeek log parsing and filtering  
- SMTP anomaly detection  
- Suricata rule-based detection  
- Behavioral vs signature correlation  
- SOC investigative methodology  
- Incident-style reporting  

---

# 📈 Future Improvements

- Custom Suricata rule development for SMTP abuse detection  
- Threat intelligence enrichment of external IP addresses  
- ELK stack integration for log visualization  
- Deployment in live network monitoring environment  
