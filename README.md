# DogDock Agent Monitor

[English](#english) | [ภาษาไทย](#ภาษาไทย-thai)

---

## English

Deployment bundle for **DogDock Agent Monitor** — a host-agent security detection pipeline (process/port/cron/auth-log monitoring, MITRE ATT&CK mapping, Slack alerting). 

This repository contains deployment artifacts ([docker-compose.yml](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-agent/docker-compose.yml), Kubernetes manifests, and env templates). The actual source code for each service resides in DogDock's private repositories and is published here as pre-built container images.

### Components

| Service | Image | Default Port | Role |
|---|---|---|---|
| `agent` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent` | 3009 | Next.js dashboard (frontend) |
| `agent-api` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-api` | 8093 | REST/WebSocket API backing the dashboard |
| `agent-webhook` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-webhook` | 8094 | Ingests reports from host agents, buffers to NATS |
| `agent-worker` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-worker` | — | Consumes NATS, runs detection rules, persists to MongoDB |

Additionally, standard images `mongo:7` (storage) and `nats:2-alpine` (message bus) are pulled directly from Docker Hub.

---

### 🚀 Quick Start — Docker Compose

1. Copy the environment variables template:
   ```bash
   cp .env.example .env
   ```
2. Edit `.env` and set `MONGO_ROOT_PASSWORD` and `NEXTAUTH_SECRET` to secure, random values:
   ```bash
   # Generate a random base64 string
   openssl rand -base64 48
   ```
3. Start the services:
   ```bash
   docker compose up -d
   ```
4. Access the dashboard at `http://localhost:3009`. 
   
> [!NOTE]
> Since no admin account exists initially, you will be prompted with a one-time admin setup form. Register your administrator credentials to proceed.

---

### ☸️ Quick Start — Kubernetes (Manifest-based)

1. Open [dogdock-agent-deploy.yaml](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-agent/k8s/dogdock-agent-deploy.yaml).
2. Edit the placeholders for `MONGO_ROOT_PASSWORD` and `NEXTAUTH_SECRET` in the `Secret` resource with base64-encoded values.
3. Deploy to your cluster:
   ```bash
   kubectl apply -f https://github.com/dogdock/dogdock-agent/blob/dbf3d77a57c2aa0b23b4230547733b2f630f1014/k8s/dogdock-agent-deploy.yaml
   ```
4. Verify the pods are running:
   ```bash
   kubectl get pods -n dogdock-agent
   ```
5. Forward the dashboard and webhook ports to access them locally:
   ```bash
   # Access dashboard
   kubectl port-forward -n dogdock-agent svc/agent 3009:3009
   
   # Access webhook (to send agent logs)
   kubectl port-forward -n dogdock-agent svc/agent-webhook 8094:8094
   ```

---

### 🛡️ Installing Host Collector Agent

To monitor a host VM/server, you must install the [dogdock-agent-collector](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-agent-collector) binary on it:

1. **Register Host**: On the DogDock Dashboard (under Agents), click **"Register New Agent"** to generate an `Agent ID` and `Agent Token`.
2. **Build Agent**: Build the Go binary on the target machine:
   ```bash
   go build -o dogdock-agent-collector .
   ```
3. **Install & Run**: Run the installer script with root privileges to register it as a systemd service:
   ```bash
   sudo ./deploy/install.sh <AGENT_ID> <AGENT_TOKEN> <WEBHOOK_URL>
   ```
4. **Verify**:
   ```bash
   systemctl status dogdock-agent
   journalctl -u dogdock-agent -f
   ```

---

### ⚠️ Known Limitations

* **Frontend URLs are baked in at build time**: The published `agent` image is pre-configured to point to DogDock's SaaS URLs (`agent-api.dogdock.dev`). For completely isolated or custom-domain self-hosted setups, rebuild the frontend image from source with build-argument overrides (see [dogdock-platform-agent-selfhosted/README.md](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-platform-agent-selfhosted/README.md)).
* **No automatic TLS/Reverse Proxy**: Put this behind an Ingress Controller or reverse proxy (Caddy, Nginx) before deploying beyond a local trial.
* **Privileged Execution**: The host collector agent needs root access to query `/var/log/auth.log` and active crontabs.

---

## ภาษาไทย (Thai)

ชุดไฟล์สำหรับติดตั้ง **DogDock Agent Monitor** — ระบบวิเคราะห์และตรวจจับความมั่นคงปลอดภัยระดับโฮสต์ (เฝ้าดูและตรวจสอบ Process, Port, Cron, และ Auth-log ร่วมกับการเทียบเคียงพฤติกรรมกับ **MITRE ATT&CK Framework** และแจ้งเตือนผ่าน Slack)

รีโปนี้รวบรวมไฟล์อาร์ติแฟกต์สำหรับรันระบบหลังบ้าน ([docker-compose.yml](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-agent/docker-compose.yml), Kubernetes manifests, และ env templates) สำหรับซอร์สโค้ดหลักของแต่ละบริการได้รับการเผยแพร่ในรูปแบบ Container Image สำเร็จรูปบนการลงทะเบียนสาธารณะของเรา

### ส่วนประกอบของระบบ

| บริการ (Service) | คอนเทนเนอร์อิมเมจ (Image) | พอร์ตหลัก | หน้าที่ |
|---|---|---|---|
| `agent` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent` | 3009 | Dashboard หน้าบ้าน (Next.js) |
| `agent-api` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-api` | 8093 | REST/WebSocket API สำหรับดึงข้อมูลหน้าจอ |
| `agent-webhook` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-webhook` | 8094 | จุดรับ Log Telemetry จากโฮสต์ต่าง ๆ เข้าสู่ NATS |
| `agent-worker` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-worker` | — | ดึงข้อมูลจากคิวมาวิเคราะห์พฤติกรรมความปลอดภัยและบันทึกลง MongoDB |

นอกจากนี้ จะมีการดึงบริการเสริมระบบจัดการคิวอย่าง `nats:2-alpine` และระบบจัดเก็บฐานข้อมูลอย่าง `mongo:7` จาก Docker Hub โดยตรง

---

### 🚀 เริ่มต้นด่วนด้วย Docker Compose

1. คัดลอกเทมเพลตตัวแปรสภาพแวดล้อม:
   ```bash
   cp .env.example .env
   ```
2. แก้ไขไฟล์ `.env` และเปลี่ยนค่า `MONGO_ROOT_PASSWORD` และ `NEXTAUTH_SECRET` ให้เป็นค่าสุ่มระดับสูงเพื่อความปลอดภัย:
   ```bash
   # สุ่มรหัสผ่านระดับสูง
   openssl rand -base64 48
   ```
3. สั่งรันชุดบริการทั้งหมดในเบื้องหลัง:
   ```bash
   docker compose up -d
   ```
4. เปิดหน้าเว็บแดชบอร์ดที่พอร์ต `http://localhost:3009`

> [!NOTE]
> เนื่องจากการรันระบบครั้งแรกจะไม่มีบัญชีผู้ดูแลระบบ คุณจะเข้าสู่หน้าจอกำหนดค่าบัญชีผู้ใช้ขั้นแรกอัตโนมัติ ให้ทำการลงทะเบียนข้อมูลบัญชี Admin เพื่อเริ่มใช้งาน

---

### ☸️ เริ่มต้นด่วนด้วย Kubernetes (Manifest-based)

1. เปิดไฟล์ [dogdock-agent-deploy.yaml](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-agent/k8s/dogdock-agent-deploy.yaml)
2. แก้ไขส่วนค่าของ `MONGO_ROOT_PASSWORD` และ `NEXTAUTH_SECRET` ในชนิดข้อมูล `Secret` ให้เป็นค่าที่ผ่านการเข้ารหัส base64 เรียบร้อยแล้ว
3. สั่งติดตั้งบริการลงบนคลัสเตอร์:
   ```bash
   kubectl apply -f https://github.com/dogdock/dogdock-agent/blob/dbf3d77a57c2aa0b23b4230547733b2f630f1014/k8s/dogdock-agent-deploy.yaml
   ```
4. ตรวจสอบสถานะ Pod ภายใน namespace:
   ```bash
   kubectl get pods -n dogdock-agent
   ```
5. เปิดใช้งานพอร์ตชั่วคราวเพื่อเข้าทดสอบการเชื่อมต่อ:
   ```bash
   # หน้าจอหลัก Dashboard
   kubectl port-forward -n dogdock-agent svc/agent 3009:3009
   
   # จุดรับข้อมูล Webhook
   kubectl port-forward -n dogdock-agent svc/agent-webhook 8094:8094
   ```

---

### 🛡️ การติดตั้งตัวสแกนบนเครื่องปลายทาง (Host Collector Agent)

สำหรับการสแกนข้อมูลระบบความปลอดภัยของ VM/Server ปลายทาง คุณจำเป็นต้องติดตั้งเอเจนต์ดึงข้อมูล [dogdock-agent-collector](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-agent-collector):

1. **ลงทะเบียนโฮสต์**: บนแดชบอร์ดระบบ (ในหน้าจอ Agents) กดปุ่ม **"Register New Agent"** เพื่อสร้างและคัดลอก `Agent ID` และ `Agent Token`
2. **ทำการคอมไพล์โปรแกรม**: คอมไพล์โปรแกรม Go บนเครื่องปลายทาง:
   ```bash
   go build -o dogdock-agent-collector .
   ```
3. **ติดตั้งการทำงาน**: รันคำสั่งสคริปต์ติดตั้งในสิทธิ์ root เพื่อรันเอเจนต์เป็นระบบบริการถาวรเบื้องหลัง (Systemd Service):
   ```bash
   sudo ./deploy/install.sh <AGENT_ID> <AGENT_TOKEN> <WEBHOOK_URL>
   ```
4. **ตรวจสอบการเชื่อมต่อ**:
   ```bash
   systemctl status dogdock-agent
   journalctl -u dogdock-agent -f
   ```

---

### ⚠️ ข้อจำกัดสำคัญที่ต้องทราบ

* **URL ของหน้าบ้านถูกกำหนดตายตัวไว้ตอน Build**: อิมเมจ `agent` สำเร็จรูปจะถูกคอมไพล์ให้ชี้บริการเชื่อมโยงไปยัง SaaS หลักของ DogDock (`agent-api.dogdock.dev`) หากต้องการรันแบบระบบปิดภายในองค์กรอย่างสมบูรณ์แบบ ให้ทำการบิวด์อิมเมจจากซอร์สโค้ดหน้าบ้านด้วยการระบุ Build Args เพิ่มเติม (ศึกษารายละเอียดที่ [dogdock-platform-agent-selfhosted/README.md](file:///Users/prisanp/Desktop/dogdock/dogdock-develop-team/code/dogdock-agent-monitor/dogdock-platform-agent-selfhosted/README.md))
* **ไม่มีบริการ HTTPS/TLS ในตัว**: ควรวางโครงสร้างระบบหลังบ้านนี้ไว้ข้างหลัง Ingress Controller หรือ Nginx Proxy ที่ลงทะเบียนใบรับรอง TLS เพื่อความปลอดภัยของข้อมูล
* **การรันในสิทธิ์ระดับสูง**: ตัวเอเจนต์ปลายทางทำงานจำเป็นต้องรันในสิทธิ์ `root` เพื่อใช้อ่านข้อมูลจากไดเรกทอรีระบบระดับสูง เช่น `/var/log/auth.log` และตรวจสอบตารางคำสั่ง cron ของผู้ใช้อื่น ๆ
