# dogdock-agent

[English](#english) | [ภาษาไทย](#ภาษาไทย-thai)

---

## English

Deployment bundle for **DogDock Agent Monitor** — a host-agent security
detection pipeline (process/port/cron/auth-log monitoring, MITRE ATT&CK
mapping, Slack alerting). This repo only contains install artifacts
(`docker-compose.yml`, Kubernetes manifest, env template); the actual
source code for each service lives in DogDock's private repos and is
published here only as pre-built container images.

### Components

| Service | Image | Port | Role |
|---|---|---|---|
| `agent` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent` | 3009 | Next.js dashboard (frontend) |
| `agent-api` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-api` | 8093 | REST/WebSocket API backing the dashboard |
| `agent-webhook` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-webhook` | 8094 | Ingests reports from host agents, buffers to NATS |
| `agent-worker` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-worker` | — | Consumes NATS, runs detection rules, persists to Mongo |

Plus `mongo:7` (storage) and `nats:2-alpine` (message bus), both pulled
from Docker Hub directly.

Images are tagged per branch by CI (`main` = latest from the main branch).
Pin `IMAGE_TAG` (Compose) / the image tag in `k8s/dogdock-agent-deploy.yaml`
to a specific release for anything beyond a trial.

### Quick start — Docker Compose

```bash
cp .env.example .env
# edit .env: set MONGO_ROOT_PASSWORD and NEXTAUTH_SECRET to real random values
#   openssl rand -base64 48

docker compose up -d
```

Open `http://localhost:3009` — since no admin account exists yet, you'll
see a one-time "create admin account" form instead of a login form. Point
your host agents at `http://<this-host>:8094/api/agent/logs`.

### Quick start — Kubernetes (`kubectl apply`, no Helm)

```bash
# 1. Edit k8s/dogdock-agent-deploy.yaml first: replace the placeholder
#    MONGO_ROOT_PASSWORD and NEXTAUTH_SECRET in the Secret with real
#    random values (openssl rand -base64 48).

kubectl apply -f k8s/dogdock-agent-deploy.yaml

# 2. Check pods
kubectl get pods -n dogdock-agent

# 3. Reach the dashboard
kubectl port-forward -n dogdock-agent svc/agent 3009:3009

# Point host agents at agent-webhook, e.g. via port-forward:
kubectl port-forward -n dogdock-agent svc/agent-webhook 8094:8094
```

A commented-out `Ingress` block is included at the bottom of the manifest
— uncomment it and set your own host/ingressClassName/TLS secret to expose
this outside the cluster instead of using port-forward.

A more configurable Helm chart (adjustable replicas/resources/ingress TLS,
bring-your-own-Secret support) also exists in DogDock's private source
repos if you need it — this repo is the plain-manifest, no-Helm equivalent.

### Environment variables

See `.env.example` for the full list. The required ones (fail closed if
unset) are `MONGO_ROOT_PASSWORD` and `NEXTAUTH_SECRET`. Everything else
(GCS dual-write, AWS EventBridge ingestion, buffer tuning) is optional.

### Known limitations

- **Frontend URLs are baked in at image build time.** The published
  `agent` image is built pointing at DogDock's own SaaS URLs
  (`agent-api.dogdock.dev` / `console.dogdock.dev` / `agent.dogdock.dev`),
  not this deployment — that's a Next.js constraint (`NEXT_PUBLIC_*` vars
  can't be overridden at container runtime). For a fully isolated install,
  you'll need to rebuild the `agent` image yourself from source with
  `--build-arg` overrides pointed at your own hostnames.
- No ClickHouse/Redis by default — `agent-api`'s optional analytics routes
  and caching are simply disabled (logs a warning, keeps running).
- No automatic TLS/reverse proxy — put this behind a real Ingress/proxy
  with certificates before using it beyond a local trial.
- AWS CloudTrail ingestion (`/api/cloudevents/aws`) needs its own setup on
  your AWS side (EventBridge API Destination pointed at `agent-webhook`).
- No self-serve licensing flow yet; contact DogDock for entitlement setup.

---

## ภาษาไทย (Thai)

ชุดไฟล์สำหรับติดตั้ง **DogDock Agent Monitor** — ระบบตรวจจับความปลอดภัยระดับ host
(เฝ้าดู process/port/cron/auth-log, จับคู่กับ MITRE ATT&CK, แจ้งเตือนผ่าน Slack)
รีโปนี้มีแค่ไฟล์สำหรับติดตั้ง (`docker-compose.yml`, Kubernetes manifest, env
template) — source code จริงของแต่ละ service อยู่ใน private repo ของ DogDock
และเผยแพร่มาที่นี่ในรูปแบบ container image สำเร็จรูปเท่านั้น

### ส่วนประกอบ

| Service | Image | Port | หน้าที่ |
|---|---|---|---|
| `agent` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent` | 3009 | Dashboard หน้าบ้าน (Next.js) |
| `agent-api` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-api` | 8093 | REST/WebSocket API ของ dashboard |
| `agent-webhook` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-webhook` | 8094 | รับรายงานจาก host agent แล้ว buffer เข้า NATS |
| `agent-worker` | `asia-southeast3-docker.pkg.dev/dogdock/dogdock-public/dogdock-platform-agent-worker` | — | อ่านจาก NATS, รัน detection rules, บันทึกลง Mongo |

รวมถึง `mongo:7` (เก็บข้อมูล) และ `nats:2-alpine` (message bus) ที่ดึงจาก
Docker Hub โดยตรง

Image ถูก tag ตามแต่ละ branch โดย CI (`main` = ล่าสุดจาก branch main) — ถ้าจะ
ใช้งานจริงเกินกว่าทดลอง ควร pin `IMAGE_TAG` (สำหรับ Compose) หรือ tag ใน
`k8s/dogdock-agent-deploy.yaml` ให้เป็น release เวอร์ชันที่แน่นอน

### เริ่มต้นเร็ว — Docker Compose

```bash
cp .env.example .env
# แก้ .env: ใส่ MONGO_ROOT_PASSWORD และ NEXTAUTH_SECRET เป็นค่าสุ่มจริง
#   openssl rand -base64 48

docker compose up -d
```

เปิด `http://localhost:3009` — เนื่องจากยังไม่มี admin account จะเจอฟอร์ม
"สร้าง admin account" ครั้งแรกแทนฟอร์ม login ปกติ ตั้งค่า host agent ให้ชี้มาที่
`http://<host นี้>:8094/api/agent/logs`

### เริ่มต้นเร็ว — Kubernetes (`kubectl apply` ไม่ใช้ Helm)

```bash
# 1. แก้ k8s/dogdock-agent-deploy.yaml ก่อน: เปลี่ยน placeholder ของ
#    MONGO_ROOT_PASSWORD และ NEXTAUTH_SECRET ใน Secret ให้เป็นค่าสุ่มจริง
#    (openssl rand -base64 48)

kubectl apply -f k8s/dogdock-agent-deploy.yaml

# 2. เช็ค pod
kubectl get pods -n dogdock-agent

# 3. เข้าถึง dashboard
kubectl port-forward -n dogdock-agent svc/agent 3009:3009

# ตั้งค่า host agent ให้ชี้มาที่ agent-webhook เช่นผ่าน port-forward:
kubectl port-forward -n dogdock-agent svc/agent-webhook 8094:8094
```

ท้ายไฟล์ manifest มีตัวอย่าง `Ingress` แบบ comment ไว้ให้ — เปิดใช้และใส่
host/ingressClassName/TLS secret ของตัวเองได้ถ้าต้องการเปิดออกนอกคลัสเตอร์
แทนการใช้ port-forward

ถ้าต้องการความยืดหยุ่นกว่านี้ (ปรับ replicas/resources/ingress TLS,
bring-your-own-Secret) DogDock มี Helm chart ที่ยืดหยุ่นกว่าอยู่ใน private
source repo เช่นกัน — รีโปนี้คือเวอร์ชัน manifest ธรรมดาที่ไม่ใช้ Helm

### ตัวแปรสภาพแวดล้อม

ดูรายการทั้งหมดใน `.env.example` ตัวที่จำเป็น (ไม่ตั้งค่าแล้วจะพังทันที) คือ
`MONGO_ROOT_PASSWORD` และ `NEXTAUTH_SECRET` ที่เหลือเป็นออปชัน (GCS dual-write,
AWS EventBridge, ปรับ buffer)

### ข้อจำกัดที่ทราบอยู่แล้ว

- **URL ของหน้าบ้านถูกฝังไว้ตอน build image** — image `agent` ที่เผยแพร่ build
  มาให้ชี้ไปที่ SaaS จริงของ DogDock (`agent-api.dogdock.dev` /
  `console.dogdock.dev` / `agent.dogdock.dev`) ไม่ใช่ deployment นี้ — เป็นข้อจำกัด
  ของ Next.js (`NEXT_PUBLIC_*` override ตอน runtime ไม่ได้) ถ้าต้องการติดตั้งแบบ
  แยกขาดจริงๆ ต้อง build image `agent` เองจาก source พร้อม `--build-arg` ชี้ไปที่
  hostname ของตัวเอง
- ไม่มี ClickHouse/Redis โดยดีฟอลต์ — route analytics และ caching ของ `agent-api`
  ถูกปิดไว้เฉยๆ (แค่ log warning ไม่พัง)
- ไม่มี TLS/reverse proxy อัตโนมัติ — ควรวางไว้หลัง Ingress/proxy จริงที่มี
  certificate ก่อนใช้งานเกินกว่าทดลอง
- การรับ AWS CloudTrail (`/api/cloudevents/aws`) ต้องตั้งค่าฝั่ง AWS เอง
  (EventBridge API Destination ชี้มาที่ `agent-webhook`)
- ยังไม่มีระบบซื้อไลเซนส์แบบ self-serve — ติดต่อ DogDock เพื่อขอ entitlement
