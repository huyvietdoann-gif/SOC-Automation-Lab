# Phase 1: Infrastructure Setup

## Overview

This phase covers deploying Splunk Enterprise, Shuffle, and TheHive on a single Ubuntu server, and preparing all virtual machines in the lab environment.

---

## 1.1 VM Specifications

| VM | OS | IP | RAM | CPU | Disk |
|---|---|---|---|---|---|
| SOC Server | Ubuntu 22.04 LTS | 10.0.0.5 | 8GB | 4 core | 80GB |
| Windows Agent | Windows 10 | 10.0.0.7 | 4GB | 2 core | 50GB |
| Linux Agent | Ubuntu 20.04 | 10.0.0.4 | 2GB | 2 core | 30GB |
| Kali Attacker | Kali Linux | 10.0.0.x | 4GB | 2 core | 40GB |

---

## 1.2 Install Splunk Enterprise
> Performed on: **Ubuntu – 10.0.0.5**

### Step 1 – Download Splunk

```bash
wget -O splunk-9.2.1-linux-2.6-amd64.deb \
  "https://download.splunk.com/products/splunk/releases/9.2.1/linux/splunk-9.2.1-linux-2.6-amd64.deb"
```

> ⚠️ Lấy link download mới nhất tại: https://www.splunk.com/en_us/download/splunk-enterprise.html (yêu cầu tạo tài khoản free)

### Step 2 – Install

```bash
sudo dpkg -i splunk-9.2.1-linux-2.6-amd64.deb
sudo /opt/splunk/bin/splunk start --accept-license
```

Tạo admin credentials khi được hỏi lần đầu.

### Step 3 – Enable autostart

```bash
sudo /opt/splunk/bin/splunk enable boot-start
```

### Step 4 – Verify

```bash
sudo systemctl status Splunkd
```

Access dashboard: `http://10.0.0.5:8000`

---

## 1.3 Install Docker (for Shuffle + TheHive)
> Shuffle và TheHive sẽ chạy qua Docker Compose

```bash
# Install Docker
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

---

## 1.4 Install Shuffle

```bash
# Clone Shuffle repo
git clone https://github.com/Shuffle/Shuffle
cd Shuffle

# Start Shuffle
docker-compose up -d
```

Access Shuffle: `http://10.0.0.5:3001`

Default credentials: tạo tài khoản mới lần đầu đăng nhập.

### Verify containers running

```bash
docker ps | grep shuffle
```

---

## 1.5 Install TheHive 5

```bash
mkdir thehive && cd thehive
```

Tạo file `docker-compose.yml`:

```yaml
version: "3"
services:
  thehive:
    image: strangebee/thehive:5.2
    ports:
      - "9000:9000"
    environment:
      - JVM_OPTS=-Xms512m -Xmx512m
    volumes:
      - thehive-data:/opt/thp/thehive/db
      - thehive-index:/opt/thp/thehive/index
      - thehive-attachments:/opt/thp/thehive/attachments

volumes:
  thehive-data:
  thehive-index:
  thehive-attachments:
```

```bash
docker-compose up -d
```

Access TheHive: `http://10.0.0.5:9000`

Default credentials: `admin@thehive.local` / `secret`

---

## 1.6 Summary – Services & Ports

| Service | Port | URL |
|---|---|---|
| Splunk Web | 8000 | http://10.0.0.5:8000 |
| Splunk HEC | 8088 | http://10.0.0.5:8088 |
| Splunk Receiver | 9997 | tcp://10.0.0.5:9997 |
| Shuffle | 3001 | http://10.0.0.5:3001 |
| TheHive | 9000 | http://10.0.0.5:9000 |
