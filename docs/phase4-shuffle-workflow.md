# Phase 4: Shuffle SOAR Workflow

## Overview

This phase covers building the Shuffle automation workflow: receive alert from Splunk → extract IOCs → enrich with VirusTotal → classify severity → create case in TheHive.

---

## 4.1 Workflow Overview

```
[Webhook Trigger]
      ↓
[Extract Fields: src_ip, user, rule_name]
      ↓
[VirusTotal API – Lookup src_ip]
      ↓
[Check malicious score]
      ↓
[Set severity: Low / Medium / High]
      ↓
[Create Case in TheHive]
      ↓
[Create Observable: src_ip]
```

---

## 4.2 Setup VirusTotal API

1. Tạo tài khoản tại https://www.virustotal.com
2. Lấy API Key: **Profile → API Key**
![](../screenshots/phase4/image1.png)

---

## 4.3 Setup TheHive App trong Shuffle

1. Shuffle → **Apps → TheHive**
2. Authenticate:
   - URL: `http://127.0.0.1:9000`
   - API Key: Taken from TheHive


---

## 4.4 Lấy TheHive API Key

Login TheHive → **Admin → Users → admin → API Key → Create → copy**

```
API Key: WFF3fP0RhjcRZW5NG1F16TMyREFZ1xRR
```
![](../screenshots/phase4/image2.png)

---

## 4.5 Build Workflow – Step by Step

### Node 1: Webhook Trigger

- Type: `Webhook`
- Automatically receive payload from Splunk

---

### Node 2: Extract Fields

- Type: `Shuffle Tools → Repeat back to me`
- Input:

```
src_ip: $exec.result.src_ip
user: $exec.result.user
rule_name: $exec.search_name
```
![](../screenshots/phase4/image3.png)

---

### Node 3: VirusTotal IP Lookup

- App: `VirusTotal`
- Action: `Get IP report`
- Apikey: 86b879e3c6a024141e880a99db1e5dc38cc928088c563b34a756bdfb59dd3d2e
- Input:
  - `ip`: `1.1.1.1`
![](../screenshots/phase4/image4.png)

---

### Node 4: Check Score & Set Severity

- Type: `Shuffle Tools → Condition`
- Condition:

```
IF $virustotal.data.attributes.last_analysis_stats.malicious > 5
  → severity = High
ELSE IF > 2
  → severity = Medium
ELSE
  → severity = Low
```

---

### Node 5: Create Case in TheHive

- App: `TheHive`
- Action: `Create case`
- Input:

```json
{
  "title": "Splunk Alert: $exec.webhook_1.search_name",
  "description": "Source IP: $exec.webhook_1.result.src_ip\nUser: $exec.webhook_1.result.user\nVT Malicious Score: $exec.virustotal_v3_1.body.data.attributes.last_analysis_stats.malicious",
  "severity": 2,
  "tags": ["splunk", "automated", "soar", "CyberShield"],
  "tlp": 2,
  "status": "New",
  "type": "external"
}
```
![](../screenshots/phase4/image7.png)
> Severity mapping: 1=Low, 2=Medium, 3=High

---

### Node 6: Add Observable to Case

- App: `TheHive`
- Action: `Create observable`
- Input:

```json
{
  "caseId": "$thehive_1.body._id",
  "dataType": "ip",
  "data": "$exec.result.src_ip",
  "tlp": 2,
}
```
![](../screenshots/phase4/image5.png)

---

## 4.6 Test Workflow

1. Mở Shuffle → Workflow → **Run**
2. Send test payload:

```bash
curl -X POST http://10.0.0.5:3001/api/v1/hooks/webhook_<id> \
  -H "Content-Type: application/json" \
  -d '{
    "result": {
      "src_ip": "1.1.1.1",
      "user": "administrator",
      "count": "10",
      "host": "WIN-AGENT"
    },
    "search_name": "Brute Force Detection"
  }'
```
![](../screenshots/phase4/image8.png)


3. Kiểm tra TheHive → case mới được tạo tự động.

![](../screenshots/phase4/image9.png)

---

## 4.7 Export Workflow

Shuffle → Workflow → **Export** → lưu file `shuffle-workflow.json` vào thư mục `configs/`.
