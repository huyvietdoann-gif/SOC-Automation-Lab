# Phase 5: TheHive Case Management & Investigation

## Overview

This phase covers managing security incidents in TheHive — investigating cases created automatically by Shuffle, assigning tasks, adding observables, and closing the incident lifecycle.

---

## 5.1 TheHive Concepts

| Term | Description |
|---|---|
| **Case** | Một incident đang được điều tra |
| **Observable** | IOC liên quan: IP, domain, file hash, email |
| **Task** | Công việc cần làm trong quá trình xử lý |
| **Alert** | Thông báo chưa được triage (có thể promote lên Case) |
| **TLP** | Traffic Light Protocol – mức độ chia sẻ thông tin |

---

## 5.2 Kiểm tra Case Được Tạo Tự Động

1. Login TheHive: `http://10.0.0.5:9000`
2. Vào **Cases** → thấy case được Shuffle tạo tự động
3. Click vào case → xem chi tiết:
   - Title: `Splunk Alert: Brute Force Detection`
   - Observable: `src_ip`
   - Description: thông tin từ Splunk

---

## 5.3 Thêm Tasks Vào Case

Mở case → **Tasks tab → Add Task**:

| Task | Assignee | Description |
|---|---|---|
| Verify source IP | Analyst | Check IP reputation, geolocation |
| Check affected user | Analyst | Verify if account was compromised |
| Isolate host (if needed) | Analyst | Block or isolate affected endpoint |
| Document findings | Analyst | Write investigation notes |
| Close case | Analyst | Mark resolution |

---

## 5.4 Thêm Observable Thủ Công

Trong case → **Observables tab → Add Observable**:

```
Type: ip
Value: <src_ip>
TLP: Amber
Tags: splunk, brute-force
```

Các observable type hữu ích:
- `ip` – địa chỉ IP
- `domain` – tên miền
- `hash` – file hash (MD5/SHA256)
- `user-agent` – HTTP User-Agent
- `filename` – tên file độc hại

---

## 5.5 Analyze Observable với Cortex (Optional)

> Cortex là analyzer engine tích hợp với TheHive — có thể chạy analyzer tự động trên observable.

Nếu đã cài Cortex:
1. Click observable → **Run analyzers**
2. Chọn: `VirusTotal_GetReport`, `AbuseIPDB`, `MaxMind_GeoIP`
3. Xem kết quả ngay trong TheHive

---

## 5.6 Case Investigation Workflow

```
[Case Created by Shuffle]
        ↓
[Analyst reviews case details]
        ↓
[Verify src_ip → VirusTotal / AbuseIPDB]
        ↓
[Check Windows Security logs for affected user]
        ↓
[Determine: True Positive or False Positive?]
        ↓
    [True Positive]              [False Positive]
        ↓                              ↓
[Escalate / Isolate host]        [Close case]
[Block IP on firewall]           [Add FP tag]
[Reset compromised account]
        ↓
[Document timeline in case notes]
        ↓
[Close case – Resolution: TruePositive]
```

---

## 5.7 Close Case

Sau khi điều tra xong:

1. Mở case → **Close case**
2. Chọn resolution:
   - `TruePositive` – thực sự bị tấn công
   - `FalsePositive` – cảnh báo nhầm
   - `Indeterminate` – không xác định được
3. Điền summary notes

---

## 5.8 Verify End-to-End Flow

Checklist xác nhận toàn bộ pipeline hoạt động:

- [ ] Splunk nhận log từ Windows & Linux agents
- [ ] Splunk alert rule trigger khi detect brute force
- [ ] Webhook gửi payload sang Shuffle
- [ ] Shuffle enrich IP với VirusTotal
- [ ] Shuffle tạo case trong TheHive tự động
- [ ] Case có đầy đủ: title, description, observable, severity
- [ ] Analyst có thể assign task và investigate trong TheHive
- [ ] Case được đóng với resolution đúng
