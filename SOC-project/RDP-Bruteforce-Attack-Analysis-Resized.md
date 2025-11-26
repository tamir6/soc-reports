# RDP Brute Force Attack Analysis Report

## 📌 Overview
This report presents the analysis of a simulated **RDP brute force attack**, investigated using Splunk.

- The attack logs were artificially generated using ChatGPT for training purposes.
- The logs were ingested into Splunk to simulate a real SOC investigation workflow.
- The goal: identify suspicious behavior, detect brute-force patterns, analyze IPS/SIEM correlation, and map findings to **MITRE ATT&CK T1110 – Brute Force**  

---

## 🧩 Step 1 — Identifying Suspicious IP Addresses
After filtering relevant RDP (port **3389**) traffic in Splunk, three IP addresses appeared suspicious:

<img src="https://github.com/user-attachments/assets/dcbf4da8-27ec-4c3d-ba4a-291fdcb7b551" width="300"/>
<br/>
<img src="https://github.com/user-attachments/assets/955c6c8b-006d-4099-ad8b-8fa1464ed7f8" width="300"/>

```
203.0.113.50
10.0.0.70
10.0.0.50
```

Based on event volume, **203.0.113.50** showed the largest spike and was identified as the primary suspect.

---

## ❌ Step 1.1 — Eliminating False Positives (Non-Suspicious IPs)

### 🔹 10.0.0.50  
- The activity observed from this IP was a **legitimate login** to the user `HELPDESK`.  
- No rapid attempts, no failures, and the timeframe matched normal operational use.

### 🔹 10.0.0.70  
- This IP performed **two login attempts** targeting `it-admin`.  
- Both attempts were legitimate, without repetition, brute-force behavior, or anomalies.

<img src="https://github.com/user-attachments/assets/61eda07f-d940-4b64-be12-33ccbfa149d2" width="300"/>
<br/>
<img src="https://github.com/user-attachments/assets/630732a8-a1a3-4545-94eb-cae4a9149d4f" width="300"/>

### ✔ Conclusion  
Both **10.0.0.50** and **10.0.0.70** exhibited normal authentication behavior, with no rapid repeated login attempts.  
Therefore, **they were excluded as suspects**, leaving **203.0.113.50** as the only malicious actor.

---

## 🔍 Deep-Dive Into the Suspicious IP (203.0.113.50)

<img src="https://github.com/user-attachments/assets/854d3b45-cc7d-410a-a682-d16ecaa82b7c" width="300"/>

Further filtering showed:

- **21 total events** associated with this IP  
- **16 login attempts** within **32 seconds**

### 🕒 Attack Timeline
| Event | Timestamp |
|-------|-----------|
| **First login attempt** | 25/11/2025 10:05:01 |
| **Last recorded attempt** | 25/11/2025 10:05:33 |

<img width="900" height="125" alt="image" src="https://github.com/user-attachments/assets/4be72cf6-911e-4c36-98ba-569943d6eaca" />


<img width="900" height="294" alt="image" src="https://github.com/user-attachments/assets/28323761-1a93-436a-84af-692bb8d8ef49" />



## 🚨 IPS Alerts & Behavior

### Alert: `rdp_bruteforce_threshold`
<img src="https://github.com/user-attachments/assets/3ae207a3-da9b-4f5d-8234-b4c612d10c22" width="300"/>

### Alert: `rdp_bruteforce_blocked_ip`
<img src="https://github.com/user-attachments/assets/b3467932-bd5f-4b9c-9309-6ed4cc427af5" width="300"/>

**Conclusion:**  
The attacker IP (`203.0.113.50`) was **automatically blacklisted** by the IPS after exceeding the brute-force threshold.

---

## 🛡 Additional Investigation — Lateral Activity?
All events originated from **RDP traffic only**.  
No evidence of lateral movement or non-RDP probing was found.

---

## 📌 Attack Pattern Summary
- Traffic detected on port **3389**
- Multiple rapid login attempts (**10 attempts / 30 seconds**)  
- Attempts originating from a single IP
   
Introducing a new law for warning

<img src="https://github.com/user-attachments/assets/1f5e913b-9781-4509-8119-1f147221d2b4" width="300"/>

---

# 🛡 MITRE ATT&CK Mappings — T1110 Brute Force

## 🔧 Mitigations

| ID | Mitigation | Description |
|----|------------|-------------|
| **M1036** | Account Use Policies | Set account lockout policies after failed login attempts; block risky authentication origins. |
| **M1032** | Multi-factor Authentication | Enable MFA on all external-facing services. |
| **M1027** | Password Policies | Follow NIST password guidelines. |
| **M1018** | User Account Management | Reset accounts known to be in breached credential lists. |

---

## 🔍 Detection Strategies

| ID | Name | Analytic ID | Description |
|----|------|-------------|-------------|
| **DET0463** | Brute Force Authentication Failures with Multi-Platform Correlation | AN1275 | High volume of failed logons followed by a success. |
| | | AN1276 | Multiple failures followed by success from same user/IP. |
| | | AN1277 | Password spraying/brute force across user pool. |
| | | AN1278 | Multiple failed authentications in unified logs. |
| | | AN1279 | Excessive login attempts followed by SaaS success. |

Source: https://attack.mitre.org/techniques/T1110/

---

## 📸 Splunk Evidence
A screenshot of the Splunk query identifying these events was added to the Alerts dashboard.
