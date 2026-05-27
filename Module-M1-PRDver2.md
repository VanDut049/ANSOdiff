# Module M1 PRD — Presale & Estimation


| Trường                            | Giá trị                                                                            |
| --------------------------------- | ---------------------------------------------------------------------------------- |
| **Module**                        | M1 — Presale & Estimation                                                          |
| **Phiên bản**                     | 2.0                                                                                |
| **Ngày tạo**                      | 02/05/2026                                                                         |
| **Ngày cập nhật**                 | 21/05/2026                                                                         |
| **Trạng thái**                    | Draft                                                                              |
| **Owner**                         | [TBD]                                                                              |
| **Liên quan**                     | PRD v1.3 §6.1, §10 (Milestone 1); ADR-001 (Markdown-First); ADR-002 (Workspace-VM) |
| **Milestone**                     | Milestone 1 — M1 Presale Live (6 tuần sau Sprint 0)                                |
| **Module ship được dùng độc lập** | Có — không cần M2-M7                                                               |


---

## 1. Tổng quan module

### 1.1. Mục đích

M1 là module **đầu tiên** trong vòng đời SDLC của ANSO, đảm nhiệm giai đoạn presale: từ **brief khách hàng** đến **output package có thể giao cho client**. Đây cũng là module **ship đầu tiên** trong ANSO Phase A (sau Sprint 0).

> **Triết lý:** M1 không thay thế Sales/Presale, mà **biến brief thô thành estimate có cấu trúc, technical spec, WBS, prototype và deliverables package — có audit trail, có khả năng tái sử dụng**.

### 1.2. Vấn đề cụ thể đang giải quyết

Outsourcing companies hiện tại có pain rõ rệt ở giai đoạn presale:


| Pain                           | Hiện trạng                          | Hệ quả                                                        |
| ------------------------------ | ----------------------------------- | ------------------------------------------------------------- |
| Estimate lâu                   | 3-5 ngày cho brief 5-10 trang       | Lỡ deal vì client chờ; BA/senior bị ngắt việc khác            |
| Phụ thuộc senior cá nhân       | Chỉ 2-3 senior estimate được        | Bottleneck; junior không có cơ hội học                        |
| Không tận dụng dữ liệu lịch sử | Mỗi estimate làm lại từ đầu         | Sai số lớn (50-100%); estimate không cải thiện theo thời gian |
| Format không đồng nhất         | Mỗi người một template              | Client review khó; brand inconsistent                         |
| Thiếu confidence info          | Estimate là 1 số duy nhất           | Client không biết rủi ro; sales khó negotiate                 |
| Khó iterate                    | Brief đổi → estimate làm lại từ đầu | Không support agile presale                                   |
| Output cho client thiếu sức nặng | Chỉ có file estimate đơn giản     | Client khó hình dung sản phẩm; thiếu technical credibility    |


### 1.3. Outcome target sau khi M1 live


| Metric                                  | Trước M1              | Sau M1 (3 tháng)             |
| --------------------------------------- | --------------------- | ---------------------------- |
| Thời gian từ brief → output package     | 3-5 ngày              | 4-8 giờ                      |
| Số người làm được estimate              | 2-3 senior (BA/PM cũ) | Mọi Sales/Presale (qua tool) |
| Sai số estimate                         | 50-100%               | ≤ 20% (target KPI)           |
| Số deal có thể serve song song          | 5-10/tháng            | 30-50/tháng                  |
| Confidence score communicate với client | Không có              | 100% deal có                 |
| Output package chất lượng cho client    | Không có              | Prototype + Deliverables list

---

## 2. Personas & User Flows

### 2.1. Primary Personas

**P1 — Sales/Presale ("Tuấn")**

- Vai trò: nhận brief, làm estimate, tạo output package, negotiate với client, manage project ở phase presale, advance sang M2.
- Tần suất: 3-5 brief/tuần, theo deal pipeline.
- Pain: estimate tốn thời gian (3-5 ngày), phụ thuộc PM senior cũ, format lại liên tục, khó defend confidence trước client.
- Mục tiêu với M1: tự độc lập làm 80% estimate, chỉ escalate case phức tạp; confidence score + rationale rõ để dùng trong sales conversation; advance project sang M2 (BA team) khi output package sẵn sàng.

### 2.2. Secondary Personas

- **BA** (downstream "Lan"): nhận estimate đã chốt làm input cho M2.
- **Client** (qua output package): nhận Prototype + Deliverables list, không tương tác trực tiếp với M1.
- **Dev** (downstream "Khoa"): review tech stack đề xuất khi feature có complexity cao.

> **Note về visibility:** BA, QA, Dev đã được assign vào project ngay từ phase M1 (qua `Sales/Presale` add member) có **read-only access** vào artifact của M1 (brief, feature-tree, estimate, technical, wbs, prototype, output-package) để có context — ANSO-PRD §7.6.6.

### 2.3. Core User Flow — Happy Path

```
┌─────────────────────────────────────────────────────────────┐
│ Step 1: Tạo Project trong ANSO Control Plane                │
│ Actor: Sales/Presale                                        │
│ Optional: Assign BA/QA/Dev members (incremental — có thể    │
│           add sau, không bắt buộc đầy đủ từ đầu)            │
│ Output: Workspace VM provisioned, repo trống tạo trên GitHub│
│         current_phase: M1                                   │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 2: Upload brief + tài liệu khách hàng                  │
│ Actor: Presale                                              │
│ Input: PDF, DOCX, CSV, free-text, image                     │
│ Output: anso-docs/01-presale/brief.md                       │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 3: Trigger Feature Extraction                          │
│ Actor: Presale (1 click)                                    │
│ ANSO Action: Brief Parser Agent extract features            │
│ Output: anso-docs/01-presale/feature-tree.md (draft)        │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 4: Review & Edit Feature Tree                          │
│ Actor: Sales/Presale                                        │
│ ANSO Action: BA Copilot suggest missing features            │
│ Output: anso-docs/01-presale/feature-tree.md (refined)      │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 5: Trigger Estimate + Technical Spec Generation        │
│ Actor: Sales/Presale                                        │
│ ANSO Action:                                                │
│   Estimate Engine — 3 methods chạy song song:               │
│     - Analogy-based (RAG từ historical projects)            │
│     - Bottom-up (per-feature complexity × productivity)     │
│     - Three-point (optimistic/realistic/pessimistic)        │
│   Technical Spec Generator — suggest tech stack từ brief    │
│ Output: anso-docs/01-presale/estimate.md                    │
│         anso-docs/01-presale/technical.md                   │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 6: Trigger WBS + Prototype Generation                  │
│ Actor: Sales/Presale (1 click)                              │
│ ANSO Action:                                                │
│   WBS Generator — từ feature-tree + estimate                │
│   Prototype Generator — wireframe từ feature tree           │
│ Output: anso-docs/01-presale/wbs.md                         │
│         anso-docs/01-presale/prototype/ [HTML + PNG]        │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 7: Review & Edit All Artifacts                         │
│ Actor: Sales/Presale                                        │
│ Action: Review brief, technical, feature-tree,              │
│         estimate, wbs, prototype                            │
│ Output: All artifacts refined, ready for output package     │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 8: Generate Output Package & Advance to M2             │
│ Actor: Sales/Presale click "Next → M2"                      │
│ ANSO Action:                                                │
│   - Compile output-package.md (deliverables list)           │
│   - Push all artifacts lên GitHub repo                      │
│   - project current_phase: M1 → M2                          │
│   - Input cho M2 (BA team unlock)                           │
│   - Members BA/QA/Dev assigned nhận notification            │
└─────────────────────────────────────────────────────────────┘
```

### 2.4. Alternative Flows

#### 2.4.1. Iteration loop khi brief đổi

```
Brief v1 → Estimate v1 → Send to client
                ↓
        Client feedback / change scope
                ↓
        Update brief.md (v2)
                ↓
        Re-trigger từ Step 3
                ↓
        ANSO diff: feature-tree v1 vs v2 → highlight changes
        ANSO recompute estimate, wbs, show delta
                ↓
        Estimate v2, WBS v2 (có git history với v1)
```

Quan trọng: **không** bắt đầu lại từ đầu — ANSO biết artifact đã có, chỉ update phần thay đổi.

#### 2.4.2. Multi-stakeholder review

```
Estimate draft → Comment workflow
                ↓
        Sales/Presale: comment on price strategy & timeline
        BA (assigned, if any): comment on requirement clarity
        Dev (assigned senior, if any): comment on tech stack feasibility & complexity
                ↓
        Sales/Presale resolve comments, update estimate
                ↓
        Artifacts refined → output package
```

Comments lưu trong git history (qua PR-like flow trên ANSO web UI) hoặc trong front matter `comments[]` array.

---

### 2.5. Project Lifecycle Controls

M1 là module **đầu tiên** trong vòng đời project (khi Sales/Presale là creator). Các rule lifecycle áp dụng cho project ở phase M1:

**Creator quyền hạn:**

- Sales/Presale (creator) có quyền **cancel project** bất kỳ lúc nào khi `current_phase == M1`.
- Sales/Presale (creator) có quyền **add/remove/swap member** (BA, QA, Dev) bất kỳ lúc nào.
- Sales/Presale (creator) có quyền **edit RW** trên toàn bộ artifact M1 (brief, technical, feature-tree, estimate, wbs, prototype, output-package).
- Sau khi click "Next → M2": project chuyển phase, Sales/Presales không được cancel project, BA team unlock access M2.

**Member visibility tại M1:**

- BA, QA, Dev được assign sẽ thấy project trong list của họ NGAY, kể cả khi project đang ở phase M1.
- Member click vào project → **read-only access full** vào artifact M1 (xem brief, estimate, feature-tree, technical, wbs, prototype) — purpose: cho member chuẩn bị context và capacity, không edit được ở phase M1.
- Member chưa được assign **KHÔNG** thấy project.

**Multiple members per role:** project có thể có N BA, N QA, N Dev (vd 3 Dev) — tất cả đều có read-only access.

Cross-reference: chi tiết permission matrix tại **ANSO-PRD §7.6**.

---

## 3. Artifacts & Schema

> Mọi artifact tuân theo ADR-001 §4.3 (front matter + markdown body). YAML front matter chạy ngầm để AI internal check — không render ra FE. File export mặc định là `.md`. Phần này định nghĩa schema cụ thể cho từng artifact của M1.

### 3.1. Brief (`anso-docs/01-presale/brief.md`)

**Vai trò:** Single source of truth về yêu cầu khách hàng — tổng hợp từ mọi nguồn input.

#### 3.1.1. Schema

```yaml
---
id: BRIEF-001
type: brief
title: ACME Corp — ERP Module HR
status: draft               # draft | review | approved
created_at: 2026-04-26T10:00:00Z
updated_at: 2026-04-26T15:30:00Z
created_by: tuan.presale
updated_by: tuan.presale
schema_version: 1

# Brief-specific
# YAML này do AI auto-fill từ brief upload, presales review/correct sau
client:
  name: ACME Corp
  industry: manufacturing
  size: enterprise            # startup | sme | enterprise
  contact: nguyen.vp@acme.com
  timezone: Asia/Ho_Chi_Minh

project_meta:
  type: new_build             # new_build | enhancement | migration | maintenance
  domain: erp_hr
  expected_start: 2026-06-01
  expected_end: 2026-12-31
  budget_range_usd: [50000, 100000]
  must_have_languages: [vi, en]

input_sources:
  - type: pdf
    path: brief-attachments/RFP-ACME-HR.pdf
    pages: 12
    received_at: 2026-04-26T09:30:00Z
  - type: docx
    path: brief-attachments/requirements.docx
    received_at: 2026-04-26T09:35:00Z
  - type: csv
    path: brief-attachments/employee-data-sample.csv
    received_at: 2026-04-26T09:40:00Z
  - type: image
    path: brief-attachments/system-diagram.png
    received_at: 2026-04-26T10:00:00Z
  - type: free_text
    content_inline: true        # nội dung trong body markdown phía dưới

constraints:
  technology_constraints: ["must integrate with SAP", "Azure AD SSO"]
  compliance: ["GDPR-equivalent", "Vietnam labor law"]
  team_constraints: ["client team has React experience"]

stakeholders:
  - name: Nguyen VP HR
    role: sponsor
    decision_authority: high
  - name: Tran IT Manager
    role: technical_lead
    decision_authority: medium

ambiguities:                 # ANSO tự detect và list những điểm chưa rõ
  - "Số lượng employee cần migrate chưa được nêu"
  - "Yêu cầu mobile app có nhưng chưa rõ scope"
  - "Tích hợp SAP version nào"
---

# Brief: ACME Corp — ERP Module HR

## Bối cảnh khách hàng
ACME Corp là tập đoàn sản xuất với 5,000 nhân viên...

## Mục tiêu dự án
Xây dựng module HR thay thế hệ thống cũ trên SAP B1...

## Yêu cầu nghiệp vụ chính
1. Quản lý hồ sơ nhân viên
2. Chấm công ca làm việc
3. Tính lương theo công thức phức tạp (overtime, KPI bonus)
4. ...

## Tích hợp
- SAP ERP hiện hữu
- Azure AD SSO
- ...

## Reference & attachments
- RFP đính kèm: brief-attachments/RFP-ACME-HR.pdf
- Requirements doc: brief-attachments/requirements.docx
- Sample data: brief-attachments/employee-data-sample.csv
- System diagram: brief-attachments/system-diagram.png
```

#### 3.1.2. Quy tắc

- **YAML front matter** được AI auto-fill khi parse brief upload; presales chỉ review và correct. YAML chạy ngầm, không render ra FE.
- Front matter `input_sources` track mọi nguồn → audit trail và RAG retrieval reference.
- `ambiguities` được ANSO sinh tự động khi parse brief, presale có thể manually thêm.
- Brief có thể có nhiều version qua git — `status: approved` chỉ khi presale chốt input.
- Attachments lớn (PDF, DOCX) lưu trong `brief-attachments/`, không inline vì sẽ làm markdown nặng.

### 3.2. Technical (`anso-docs/01-presale/technical.md`)

**Vai trò:** Tech stack và architecture overview — AI auto-suggest từ brief + constraints, presales review.

#### 3.2.1. Schema

```yaml
---
id: TECH-001
type: technical_spec
title: ACME ERP HR — Technical Specification
status: draft               # draft | review | approved
created_at: 2026-04-26T11:30:00Z
updated_at: 2026-04-26T11:30:00Z
created_by: technical-spec-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001

# YAML chạy ngầm — AI dùng để validate tech consistency, không render ra FE
stack:
  frontend:
    - layer: Framework
      choice: React 18 / Next.js 14
      purpose: Web application
      rationale: Team familiarity, SSR support
  backend:
    - layer: Runtime
      choice: Node.js 20+ / NestJS
      purpose: REST API server
      rationale: TypeScript-first, scalable
    - layer: Framework
      choice: NestJS / Express
      purpose: REST API framework
      rationale: Structured, maintainable
  database:
    - layer: Primary DB
      choice: PostgreSQL 16
      purpose: Main data store
      rationale: ACID, mature, JSON support
  infrastructure:
    - layer: Hosting
      choice: AWS EC2 / GCP Cloud Run
      purpose: API server, signaling server
      rationale: Auto-scale, high reliability
    - layer: CI/CD
      choice: GitHub Actions
      purpose: Automated build/test/deploy
      rationale: Direct GitHub integration, free tier sufficient
    - layer: Containerization
      choice: Docker + Docker Compose
      purpose: Backend packaging
      rationale: Reproducible, easy deploy
  security:
    - layer: Authentication
      choice: Firebase Auth / JWT + OAuth 2.0
      purpose: User auth, SSO
      rationale: Built-in Microsoft/Google login integration
    - layer: API Security
      choice: Helmet + CORS + Rate limiter
      purpose: DDoS/XSS/CSRF protection
      rationale: Industry standard
  dev_tools:
    - VSCode + ESLint + Prettier
    - Postman
    - Git + GitHub
    - pgAdmin

architecture_overview: |
  3-tier architecture: React/Next.js frontend → NestJS REST API → PostgreSQL.
  SSO via Azure AD (OAuth 2.0). CI/CD via GitHub Actions. Hosted on AWS EC2.
  SAP integration via REST/RFC adapter. All services containerized via Docker.

constraints_applied:
  - "SAP integration: REST/RFC adapter layer"
  - "Azure AD SSO: OAuth 2.0 + Firebase Auth"
  - "GDPR-equivalent: AES encryption for PII, data anonymization"
---

# Technical Specification: ACME ERP HR

## 1. Mobile Application

| # | Layer | Framework / Language | Purpose | Rationale |
|---|-------|---------------------|---------|-----------|
| 1 | Framework | React 18 / Next.js 14 | Cross-platform web app | Reduced dev cost, near-native performance |
| 2 | State Management | Redux Toolkit / Zustand | App state (auth, data cache) | Scalable, testable, async-friendly |

## 2. Backend & API

| # | Layer | Solution | Purpose | Notes |
|---|-------|---------|---------|-------|
| 1 | Runtime | Node.js 20+ | API server, auth, logging | High performance, rich ecosystem |
| 2 | Framework | NestJS / Express | REST API framework | Structured, TypeScript-first |
| 3 | Database | PostgreSQL 16 | Users, HR records, audit logs | ACID, mature, full-text search |
| 4 | Authentication | Firebase Auth / JWT + OAuth 2.0 | User auth, Azure AD SSO | Built-in MS/Google login integration |
| 5 | ORM | Prisma / TypeORM | PostgreSQL ORM | Type-safe, migration management |

## 3. Infrastructure & DevOps

| # | Layer | Platform | Purpose | Config |
|---|-------|---------|---------|--------|
| 1 | Hosting | AWS EC2 / GCP Cloud Run | Backend API server | Auto-scale, high reliability |
| 2 | CI/CD | GitHub Actions | Automated build/test/deploy | Direct GitHub integration |
| 3 | Containerization | Docker + Docker Compose | Backend packaging | Reproducible deploy |
| 4 | Monitoring | Firebase Crashlytics + Sentry | Crash/error tracking | Real-time crash reports |

## 4. Security & Compliance

| # | Aspect | Solution | Purpose |
|---|--------|---------|---------|
| 1 | Authentication & Authorization | JWT Auth | Secure user access |
| 2 | Password & Token Security | bcrypt + HTTPS (TLS) | Data encryption |
| 3 | API Security | Helmet + CORS + Rate limiter | DDoS/XSS/CSRF protection |
| 4 | Data Privacy | AES encryption for PII | User data protection |

## 5. Development Tools

| # | Tool | Purpose |
|---|------|---------|
| 1 | VSCode + ESLint + Prettier | Coding, linting, formatting |
| 2 | Postman | API testing |
| 3 | Git + GitHub | Source control |
| 4 | pgAdmin | Database management |

## Architecture Overview

[Client Browser / Mobile]
        ↓ HTTPS
[Next.js Frontend — React 18]
        ↓ REST API
[NestJS API Server — Node.js 20+]
        ↓
[PostgreSQL 16]    [Firebase Auth]    [SAP RFC Adapter]
        ↓
[GitHub Actions CI/CD → AWS EC2]
```

#### 3.2.2. Quy tắc

- AI auto-generate từ `brief.md` constraints + `feature_tree.md` domain — presales review và override từng layer.
- Mỗi lựa chọn tech có `rationale` rõ ràng — dùng trong sales conversation khi client hỏi "tại sao chọn X?".
- YAML `stack` chạy ngầm để AI validate consistency (vd nếu brief có SAP constraint mà tech stack không có adapter → flag).
- Body markdown là human-readable table — đây là phần presales show cho client/Dev review.

### 3.3. Feature Tree (`anso-docs/01-presale/feature-tree.md`)

**Vai trò:** Cây tính năng có cấu trúc, là input cho estimation và WBS.

#### 3.3.1. Schema

```yaml
---
id: FT-001
type: feature_tree
title: ACME ERP HR — Feature Breakdown
status: review              # draft | review | approved
created_at: 2026-04-26T11:00:00Z
updated_at: 2026-04-26T13:00:00Z
created_by: brief-parser-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
total_features: 47
total_features_by_complexity:
  S: 18                     # < 1 man-day
  M: 15                     # 1-3 man-day
  L: 10                     # 3-7 man-day
  XL: 4                     # > 7 man-day

derived_from:
  - input_source: RFP-ACME-HR.pdf
    confidence: 0.85
  - input_source: requirements.docx
    confidence: 0.80
  - input_source: system-diagram.png
    confidence: 0.72

ambiguity_flags: [F-023, F-031]   # features có ambiguity, cần presale clarify
---

# Feature Tree: ACME ERP HR

## F-001: Authentication & Authorization [L]
- F-001.1: Email/password login [S]
- F-001.2: SSO Azure AD integration [M]
- F-001.3: 2FA via OTP [M]
- F-001.4: Role-based access control (5 roles) [L]
- F-001.5: Session management [S]

## F-002: Employee Profile Management [XL]
- F-002.1: Personal info CRUD [M]
- F-002.2: Document upload (CV, contracts) [M]
- F-002.3: Organization chart visualization [L]
- F-002.4: Employee history tracking [M]
- F-002.5: Search & filter [M]

## F-003: Time Attendance [XL]
...
```

#### 3.3.2. Complexity rubric

ANSO đánh complexity dựa rubric:


| Code    | Mô tả                                         | Man-day estimate (single dev, mid-level) |
| ------- | --------------------------------------------- | ---------------------------------------- |
| **XS**  | Trivial change/setup                          | < 0.5                                    |
| **S**   | Single endpoint/screen, ít edge case          | 0.5 - 1                                  |
| **M**   | Multiple endpoint/screen, moderate logic      | 1 - 3                                    |
| **L**   | Complex business logic, multiple integrations | 3 - 7                                    |
| **XL**  | Đa module, nhiều unknowns                     | 7 - 15                                   |
| **XXL** | Cần break thêm                                | > 15 (warning)                           |


#### 3.3.3. Quy tắc

- Mỗi feature ID format: `F-NNN` hoặc `F-NNN.M` (sub-feature).
- Complexity gắn ở `[X]` cuối tên feature.
- ANSO **không gán XXL tự động** — flag để presale break down thêm.
- Tree có thể có 2-3 cấp depth tối đa; sâu hơn → cân nhắc thành module riêng.

### 3.4. Estimate (`anso-docs/01-presale/estimate.md`)

**Vai trò:** Estimate chi tiết theo nhiều phương pháp, có confidence score.

#### 3.4.1. Schema

```yaml
---
id: EST-001
type: estimate
title: ACME ERP HR — Estimate v1
status: draft               # draft | review | approved | superseded
version: 1                  # mỗi version mới tăng số
created_at: 2026-04-26T14:00:00Z
updated_at: 2026-04-26T14:30:00Z
created_by: estimate-engine
updated_by: estimate-engine
schema_version: 1

brief_ref: BRIEF-001
feature_tree_ref: FT-001

methods_used: [analogy, bottom_up, three_point]

summary:
  total_man_days: 187
  total_man_days_by_role:
    ba: 23           # gồm PM/Delivery Lead cũ
    qa: 24
    dev: 140         # gồm backend + frontend + devops (consolidated)
  estimated_calendar_weeks: 16   # với team 4 người
  estimated_team_size: 4
  total_cost_usd_range: [62000, 78000]
  confidence_score: 0.78         # 0-1
  confidence_label: medium-high  # low | low-medium | medium | medium-high | high

three_point:
  optimistic_md: 152
  realistic_md: 187
  pessimistic_md: 245
  pert_estimate_md: 192          # (O + 4×R + P) / 6
  implicit_buffer_md: 58         # pessimistic - realistic = implicit buffer

historical_comparison:
  similar_projects:
    - id: PROJ-2024-018
      similarity_score: 0.82
      domain: erp_hr
      actual_man_days: 195
      actual_calendar_weeks: 18
      deviation_from_estimate: +12%
    - id: PROJ-2024-031
      similarity_score: 0.71
      domain: erp_general
      actual_man_days: 220
      actual_calendar_weeks: 20
      deviation_from_estimate: +25%
    - id: PROJ-2025-007
      similarity_score: 0.68
      domain: erp_hr
      actual_man_days: 168
      actual_calendar_weeks: 14
      deviation_from_estimate: -3%

sensitivity_analysis:
  if_remove_feature: F-008 (Mobile app)
    impact_man_days: -32
    impact_confidence: +0.05
  if_add_feature: "Multi-language support"
    impact_man_days: +18
    impact_confidence: -0.03
---

# Estimate v1: ACME ERP HR

## Tổng quan

| Metric | Giá trị |
|--------|---------|
| Total effort | 187 man-day |
| Calendar timeline | 16 tuần với team 4 người |
| Cost range | $62,000 - $78,000 |
| Confidence | Medium-High (0.78) |
| Implicit buffer | 58 man-day (pessimistic - realistic) |

## Detailed breakdown by feature

### F-001: Authentication & Authorization (M - 12 md)
| Sub | Complexity | BA | BE | FE | QA | DevOps | Total |
|-----|-----------|----|----|----|----|--------|-------|
| F-001.1 | S | 0.5 | 1 | 0.5 | 0.5 | - | 2.5 |
| F-001.2 | M | 1 | 2 | 1 | 1 | 0.5 | 5.5 |
| ... | ... | ... | ... | ... | ... | ... | ... |

### F-002: Employee Profile (XL — 24 md)
...

## Method comparison

| Method | Estimate (md) | Notes |
|--------|---------------|-------|
| Analogy-based | 195 | Dựa trên 3 project ERP HR tương tự |
| Bottom-up | 178 | Per-feature × productivity rate |
| Three-point PERT | 192 | (O + 4R + P) / 6 |
| **Final (weighted)** | **187** | 0.4×Analogy + 0.4×Bottom-up + 0.2×PERT |

## Confidence rationale

- Brief có ambiguity ở 2 features (F-023, F-031) → -0.10
- Domain ERP HR có 3 historical projects similarity > 0.7 → +0.15
- Tech stack đề xuất phù hợp với team capability → +0.05
- Client chưa rõ scope mobile app → -0.05
- ...

## Assumptions
1. Team có 4 người: 1 BA (gồm PM/Delivery Lead cũ), 2 Dev (full-stack, có 1 senior cover DevOps tasks), 1 QA.
2. Productivity rate trung bình: 5 man-day/tuần/người.
3. Buffer implicit từ three-point: pessimistic - realistic = 58 md.
4. Không bao gồm: data migration từ SAP cũ, training end-user.
5. ...
```

#### 3.4.2. 3 Estimation Methods

**Method 1 — Analogy-based (RAG)**

```
1. Embed feature_tree.md → vector
2. Search Qdrant trong historical projects (cross-tenant với consent)
3. Top-K similar projects (default K=5, similarity > 0.6)
4. Weight by similarity, lấy weighted average actual_man_days
5. Adjust theo project meta (team size, stack)
```

Ưu: dùng dữ liệu thật. Nhược: cần dữ liệu lịch sử (cold start vấn đề).

**Method 2 — Bottom-up**

```
For each feature in feature_tree:
    md_per_feature = complexity_baseline[feature.complexity] × stack_multiplier
    For each role:
        role_md = md_per_feature × role_distribution[feature.category]
    sum into totals
```

Baseline:

- XS: 0.3 md, S: 0.8 md, M: 2 md, L: 5 md, XL: 11 md
- Stack multiplier: NestJS 1.0, Spring 1.2, Django 0.95, custom 1.5
- Role distribution: vary theo feature category

Ưu: deterministic, explainable. Nhược: không reflect domain-specific nuance.

**Method 3 — Three-point (PERT)**

```
For each feature:
    optimistic = bottom_up × 0.75
    realistic = bottom_up × 1.0
    pessimistic = bottom_up × 1.4 + ambiguity_penalty
    pert = (O + 4R + P) / 6
    implicit_buffer = pessimistic - realistic
```

Ưu: capture uncertainty. Nhược: vẫn dựa bottom-up baseline.

**Final estimate:** weighted average của 3 methods. Default weights:

- Analogy: 0.4 (nếu có ≥ 3 similar projects)
- Bottom-up: 0.4
- Three-point: 0.2

Nếu không có dữ liệu lịch sử → weights: Bottom-up 0.6, Three-point 0.4.

#### 3.4.3. Confidence Score Formula

```
confidence = base_confidence
           + ambiguity_factor (-0.05 mỗi ambiguity)
           + historical_data_factor (+0.15 nếu có ≥ 3 similar projects)
           + stack_familiarity_factor (+0.05 nếu stack quen)
           + domain_familiarity_factor (+0.10 nếu domain ANSO biết)
           - scope_uncertainty_factor (-0.10 nếu budget range > 50%)
           - ...

Clamp to [0, 1]
```


| Confidence range | Label       | Hành động đề xuất                                              |
| ---------------- | ----------- | -------------------------------------------------------------- |
| ≥ 0.85           | High        | Có thể generate output package trực tiếp                      |
| 0.70 - 0.85      | Medium-High | Recommend BA review trước khi gửi client                      |
| 0.50 - 0.70      | Medium      | Bắt buộc BA review, có thể cần more discovery                 |
| 0.30 - 0.50      | Low-Medium  | **Cảnh báo:** cần discovery phase trước khi commit            |
| < 0.30           | Low         | **Cảnh báo rủi ro cao:** recommend không advance, list ambiguity rõ ràng |


> **Thay đổi v2:** Confidence < 0.50 không còn hard-block output package generation. Thay bằng **warning rõ ràng** với danh sách ambiguity — presales tự quyết định có proceed không.

#### 3.4.4. Versioning

- Mỗi estimate có `version` field, tăng dần.
- Estimate cũ giữ trong git history, `status: superseded`.
- File estimate_.md luôn là version mới nhất; version cũ xem qua `git log` hoặc UI history.

### 3.5. WBS (`anso-docs/01-presale/wbs.md`)

**Vai trò:** Work Breakdown Structure high-level theo phase — đủ để client thấy cấu trúc dự án, không cần chi tiết task-level. AI auto-generate từ feature-tree + estimate.

#### 3.5.1. Schema

```yaml
---
id: WBS-001
type: wbs
title: ACME ERP HR — Work Breakdown Structure
status: draft               # draft | review | approved
created_at: 2026-04-26T15:00:00Z
updated_at: 2026-04-26T15:00:00Z
created_by: wbs-generator-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
estimate_ref: EST-001
feature_tree_ref: FT-001

total_effort_md: 187
total_calendar_weeks: 16

phases:
  - id: PH-0
    name: "Phase 0: Setup & Architecture"
    weeks: "Week 1-2"
    effort_md: 12
    milestone: "Kickoff & design complete"
    modules: ["Infrastructure setup", "DB schema design", "UI wireframe"]

  - id: PH-1
    name: "Phase 1: Core Development"
    weeks: "Week 2-10"
    effort_md: 120
    milestone: "Phase 1 delivery"
    modules: ["Authentication", "Employee Profile", "Time Attendance"]

  - id: PH-2
    name: "Phase 2: Integration & Advanced Features"
    weeks: "Week 10-14"
    effort_md: 40
    milestone: "Phase 2 delivery"
    modules: ["SAP integration", "Payroll engine", "Reports & Analytics"]

  - id: PH-3
    name: "Phase 3: Testing & Launch"
    weeks: "Week 14-16"
    effort_md: 15
    milestone: "Final acceptance"
    modules: ["UAT", "Bug fix & polish", "Deployment", "Handover"]
---

# Work Breakdown Structure: ACME ERP HR

## Schedule Overview

| Phase | Timing | Milestone | Notes |
|-------|--------|-----------|-------|
| Phase 0: Setup & Architecture | Week 1-2 | Kickoff & design | Scope confirm, DB design, architecture, UI wireframe |
| Phase 1: Core Development | Week 2-10 | Phase 1 delivery | Auth, Employee Profile, Time Attendance |
| Phase 2: Integration & Advanced | Week 10-14 | Phase 2 delivery | SAP integration, Payroll, Reports |
| Phase 3: Testing & Launch | Week 14-16 | Final acceptance | UAT, bug fix, deploy, handover |

## Detailed WBS

| No | Code | Module / Group | Task | Complexity | Effort (MD) | Role |
|----|------|---------------|------|------------|-------------|------|
| 1 | 001 | Phase 0: Setup | Infrastructure & Architecture | — | 12 | DevOps + Tech Lead |
| 2 | 001.1 | | DB schema design | Simple | 4 | Tech Lead |
| 3 | 001.2 | | Core API server setup (NestJS, Auth) | Medium | 6 | BE Engineer |
| 4 | 001.3 | | CI/CD setup (GitHub Actions), Docker | Simple | 2 | DevOps |
| 5 | 002 | Phase 1: Core | Authentication & Authorization | — | 14 | Full-stack |
| 6 | 002.1 | | Email/password login | Simple | 2 | Full-stack |
| 7 | 002.2 | | SSO Azure AD integration | Medium | 5 | BE + FE |
| 8 | 002.3 | | 2FA via OTP | Simple | 2 | BE + FE |
| 9 | 002.4 | | RBAC (5 roles) | Complex | 5 | BE Engineer |
| 10 | 003 | | Employee Profile Management | — | 24 | Full-stack |
| ... | ... | ... | ... | ... | ... | ... |
| N | 00N | Phase 3: Launch | UAT & Deployment | — | 15 | QA + DevOps |

## Effort Summary by Role

| Role | Effort (MD) | Man-months (1mm = 22 days) |
|------|------------|---------------------------|
| BA (incl. PM/Delivery Lead) | 23 | 1.05 |
| Dev (BE + FE + DevOps consolidated) | 140 | 6.36 |
| QA | 24 | 1.09 |
| **Total** | **187** | **8.50** |
```

#### 3.5.2. Quy tắc

- WBS ở M1 là **high-level phase breakdown** — không đi sâu đến sub-task level (đó là việc của M2/BA).
- AI generate dựa trên feature_tree modules → group thành phases logic.
- Mỗi line item có complexity (Very Simple / Simple / Medium / Complex / Very Complex) tham chiếu từ `CRM_Estimation_v2.xlsx` guideline.
- Total effort WBS phải khớp với `estimate.md` total_man_days.

### 3.6. Prototype (`anso-docs/01-presale/prototype/`)

**Vai trò:** Wireframe để client hình dung sản phẩm. AI auto-generate từ feature tree — đủ để show sườn màn hình chính, không cần pixel-perfect.

#### 3.6.1. Schema

```yaml
---
id: PROTO-001
type: prototype
title: ACME ERP HR — Prototype
status: draft               # draft | review | approved
created_at: 2026-04-26T16:00:00Z
updated_at: 2026-04-26T16:00:00Z
created_by: prototype-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
feature_tree_ref: FT-001

tool: ai_generated            # ai_generated | figma_upload | image_upload
output_format: "HTML & PNG"   # html_wireframe | png

screens:
  - id: SCR-001
    name: Login / Authentication
    features: [F-001.1, F-001.2]
    path: prototype/SCR-001-login.md
  - id: SCR-002
    name: Employee Profile Management
    features: [F-002.1, F-002.2, F-002.3]
    path: prototype/SCR-002-employee-profile.md
  - id: SCR-003
    name: Time Attendance Dashboard
    features: [F-003.1, F-003.2]
    path: prototype/SCR-003-attendance.md
  - id: SCR-004
    name: Organization Chart
    features: [F-002.3]
    path: prototype/SCR-004-org-chart.md
---

# Prototype: ACME ERP HR

## Danh sách màn hình

| Screen | Features | Status |
|--------|----------|--------|
| SCR-001: Login / Authentication | F-001.1, F-001.2 | Draft |
| SCR-002: Employee Profile | F-002.1 → F-002.5 | Draft |
| SCR-003: Time Attendance | F-003.1, F-003.2 | Draft |
| SCR-004: Organization Chart | F-002.3 | Draft |

## SCR-001: Login / Authentication

## SCR-002: Employee Profile
...
```

#### 3.6.2. Quy tắc

- AI tự generate wireframe ASCII layout hoặc [TBD format] từ feature list của từng màn hình.
- Output format (`html_wireframe` vs `png`)
- Có thể upload ảnh mockup từ client.
- Prototype chỉ cần show màn hình **chính** (không cần edge case, error state).

### 3.7. Output Package (`anso-docs/01-presale/output-package.md`)

**Vai trò:** Deliverables package tổng hợp toàn bộ artifact M1 — đây là thứ presales giao cho client. Bao gồm code (starter/infra templates), guideline docs, toàn bộ push lên GitHub.

#### 3.7.1. Schema

```yaml
---
id: OUTPUT-001
type: output_package
title: ACME ERP HR — Output Package
status: draft               # draft | review | delivered
created_at: 2026-04-26T17:00:00Z
updated_at: 2026-04-26T17:00:00Z
created_by: output-package-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
estimate_ref: EST-001
wbs_ref: WBS-001
prototype_ref: PROTO-001
technical_ref: TECH-001

github_repo: "https://github.com/acme-outsourcing/acme-erp-hr"
storage: github

output_format: "zip"   # html_wireframe | png | md | source code

deliverables:
  - no: 1
    category: "Design & Architecture"
    item: "System Architecture Document"
    detail: "Architecture design, data flow, API spec, tech stack rationale"
    path: anso-docs/01-presale/technical.md
    format: .md
    storage: GitHub Repository

  - no: 2
    category: "Design & Architecture"
    item: "Prototype / Wireframe"
    detail: "Main screens wireframe for all core modules"
    path: anso-docs/01-presale/prototype/
    format: "PNG | HTML"
    storage: GitHub Repository

  - no: 3
    category: "Estimation"
    item: "Estimate Document"
    detail: "Feature-level effort breakdown, confidence score, method comparison"
    path: anso-docs/01-presale/estimate.md
    format: .md
    storage: GitHub Repository

  - no: 4
    category: "Planning"
    item: "Work Breakdown Structure"
    detail: "Phase-level WBS, effort by role, milestone timeline"
    path: anso-docs/01-presale/wbs.md
    format: .md
    storage: GitHub Repository

  - no: 5
    category: "Documentation"
    item: "Deployment Guide"
    detail: "Server setup, config, cloud hosting, CI/CD pipeline guide"
    path: anso-docs/01-presale/deploy-guide.md
    format: .md
    storage: GitHub Repository

  - no: 6
    category: "Documentation"
    item: "API Documentation"
    detail: "All endpoints Swagger/OpenAPI spec"
    path: anso-docs/01-presale/api-spec.yaml
    format: .yaml
    storage: GitHub Repository
---

# Output Package: ACME ERP HR

## Deliverables

| No | Category | Deliverable | Detail | Storage | Format |
|----|----------|-------------|--------|---------|--------|
| 1 | Design & Architecture | System Architecture Document | Architecture design, data flow, API spec, tech stack rationale | GitHub Repository | .md |
| 2 | Design & Architecture | Prototype / Wireframe | Main screens wireframe for all core modules | GitHub Repository | [TBD] |
| 3 | Estimation | Estimate Document | Feature-level effort breakdown, confidence score, method comparison | GitHub Repository | .md |
| 4 | Planning | Work Breakdown Structure | Phase-level WBS, effort by role, milestone timeline | GitHub Repository | .md |
| 5 | Documentation | Deployment Guide | Server setup, config, cloud hosting, CI/CD pipeline guide | GitHub Repository | .md |
| 6 | Documentation | API Documentation | All endpoints Swagger/OpenAPI spec | GitHub Repository | .yaml |

## GitHub Repository
- Repo: `https://github.com/acme-outsourcing/acme-erp-hr` (Private)
- Presales advance to M2 → BA team nhận notification + repo access

## Next Steps
1. Review output package với client
2. Confirm scope, timeline, budget
3. Sign contract → ANSO advance to M2
```

#### 3.7.2. Quy tắc

- Output Package là **client-facing deliverable** — đây là thứ presales show/giao cho client thay cho Proposal document cũ.
- `Output format` zip
- Toàn bộ artifacts push lên GitHub repo trước khi presales click "Next → M2".
- Presales **confirm** trước khi push (explicit permission action).

---

## 4. Functional Requirements

### 4.1. Brief Ingestion (F1.1)

**Inputs supported:**


| Format    | Method                 | Notes                                         |
| --------- | ---------------------- | --------------------------------------------- |
| PDF       | Upload via UI hoặc CLI | OCR fallback nếu scan                         |
| DOCX      | Upload                 | Convert sang markdown                         |
| CSV       | Upload                 | Parse structured data, append vào brief       |
| Free text | Paste vào textarea     | Direct save vào brief.md                      |
| Image     | Upload                 | OCR + LLM describe (PNG, JPG, WEBP)           |


**Acceptance Criteria:**

- AC-1.1.1: Hỗ trợ tối thiểu 5 file đính kèm/brief.
- AC-1.1.2: File size ≤ 20MB (per file), tổng ≤ 100MB.
- AC-1.1.3: Brief.md tự generate trong < 3 phút sau khi upload đầy đủ.

### 4.2. Feature Extraction (F1.2)

**Brief Parser Agent flow:**

```
1. Read brief.md + all attachments (resolved content)
2. LLM prompt: "Extract feature tree from this requirement"
3. Validate schema (front matter)
4. Detect ambiguity → flag
5. Suggest missing common features (auth, audit log, ...)
6. Write feature-tree.md
```

**Acceptance Criteria:**

- AC-1.2.1: Cho brief 5-10 trang → output feature tree với ≥ 80% feature được extract đúng (so với manual).
- AC-1.2.2: Mỗi feature có complexity gắn.
- AC-1.2.3: Ambiguity flag chính xác ≥ 70% (precision).
- AC-1.2.4: Tree có 2-3 cấp depth, không nested quá sâu.
- AC-1.2.5: Edit thủ công feature-tree.md → không bị overwrite khi re-trigger nếu file đã có content (chỉ append/suggest, không destroy).

### 4.3. Estimation Engine (F1.3)

**Đã chi tiết ở §3.4.2.**

**Acceptance Criteria:**

- AC-1.3.1: 3 methods chạy song song, total time < 2 phút cho 50 features.
- AC-1.3.2: Final estimate = weighted average với rationale rõ ràng.
- AC-1.3.3: Confidence score formula deterministic — cùng input cho cùng output.
- AC-1.3.4: Method comparison table luôn hiển thị (transparency).
- AC-1.3.5: Estimate có sensitivity analysis cho ≥ 3 features quan trọng nhất.
- AC-1.3.6: Confidence < 0.50 → hiển thị **warning** rõ ràng kèm danh sách ambiguity; không hard-block.

### 4.4. Technical Spec Generator (F1.4)

**Technical Spec Agent flow:**

```
1. Read brief.md (constraints, domain, stack preferences)
2. Read feature-tree.md (feature categories → infer tech needs)
3. LLM: suggest tech stack per layer (frontend, backend, DB, infra, security)
4. Validate consistency: constraints từ brief vs stack đề xuất
5. Generate rationale per choice
6. Write technical.md
```

**Acceptance Criteria:**

- AC-1.4.1: Tech stack đề xuất không conflict với constraints trong brief (vd brief nói "must use Azure" → không propose AWS).
- AC-1.4.2: Mỗi lựa chọn có rationale ≥ 1 câu.
- AC-1.4.3: Presales có thể override từng layer — không bị overwrite khi re-generate (chỉ update layer không bị pin).
- AC-1.4.4: Security layer luôn được generate (không optional).

### 4.5. WBS Generator (F1.5)

**WBS Generator flow:**

```
1. Read feature-tree.md (features, complexity)
2. Read estimate.md (total effort, role breakdown)
3. Group features thành phases logic (setup → core → integration → QA/launch)
4. Assign effort per phase dựa trên feature complexity distribution
5. Validate: sum(phase effort) == estimate.total_man_days
6. Write wbs.md
```

**Acceptance Criteria:**

- AC-1.5.1: Tổng effort trong WBS khớp với estimate total (±5%).
- AC-1.5.2: Mỗi phase có milestone, timing (week range), và modules rõ ràng.
- AC-1.5.3: WBS có ≥ 3 phases; không quá 6 phases cho một project presales.
- AC-1.5.4: Complexity level theo guideline `CRM_Estimation_v2.xlsx`: Very Simple / Simple / Medium / Complex / Very Complex.
- AC-1.5.5: Khi estimate update → WBS re-generate giữ phase structure, chỉ update effort numbers.

### 4.6. Prototype Generator (F1.6)

**Prototype Agent flow:**

```
1. Read feature-tree.md → identify main screens (top-level modules)
2. For each screen: map features → UI components
3. Generate wireframe layout (format: [TBD])
4. Write prototype/SCR-NNN.md per screen + index prototype.md
```

**Acceptance Criteria:**

- AC-1.6.1: Mỗi top-level feature module → ít nhất 1 screen wireframe.
- AC-1.6.2: Screen có đủ: tên màn hình, feature IDs liên quan, layout cơ bản.
- AC-1.6.3: Presales có thể replace AI-generated screen bằng upload (Figma link hoặc image).
- AC-1.6.4: Output format **[TBD — pending confirm]**.

### 4.7. Output Package (F1.7)

**Output Package flow:**

```
1. Check all artifacts ready: brief, technical, feature-tree, estimate, wbs, prototype
2. Compile output-package.md (deliverables list + links)
3. Presales confirm → push all artifacts to GitHub repo
4. Trigger: project current_phase M1 → M2
5. Notify BA/QA/Dev assigned members
```

**Acceptance Criteria:**

- AC-1.7.1: Output-package.md tự compile danh sách đầy đủ artifacts đã có.
- AC-1.7.2: Presales phải **explicitly confirm** trước khi push lên GitHub (không auto-push).
- AC-1.7.3: Push thành công → project advance M1 → M2 trong < 30 giây.
- AC-1.7.4: Delivery channel **[TBD — pending confirm]**.

### 4.8. Versioning & Iteration (F1.8)

**Acceptance Criteria:**

- AC-1.8.1: Mỗi artifact thay đổi → git commit auto với message rõ ("brief updated by tuan", "estimate v2 generated").
- AC-1.8.2: UI hiển thị version history với diff view.
- AC-1.8.3: Compare 2 versions estimate → hiển thị delta theo feature, total.
- AC-1.8.4: Rollback estimate sang version cũ qua UI 1 click.

### 4.9. Audit Trail (F1.9)

**Mọi action của ANSO/agent ghi log:**

```jsonl
{"ts":"2026-04-26T11:00:00Z","actor":"brief-parser-agent","action":"extract_features","input":"brief.md@abc123","output":"feature-tree.md@def456","model":"claude-opus-4","tokens_in":12500,"tokens_out":3200,"cost_usd":0.42,"duration_s":45}
{"ts":"2026-04-26T14:00:00Z","actor":"estimate-engine","action":"compute_estimate","method":"analogy","retrieved_projects":["PROJ-2024-018","PROJ-2024-031","PROJ-2025-007"],"output":"estimate.md@ghi789"}
{"ts":"2026-04-26T15:00:00Z","actor":"wbs-generator-agent","action":"generate_wbs","input":"feature-tree.md@def456,estimate.md@ghi789","output":"wbs.md@jkl012"}
{"ts":"2026-04-26T16:00:00Z","actor":"prototype-agent","action":"generate_prototype","input":"feature-tree.md@def456","output":"prototype/@mno345"}
```

Lưu ở `anso-docs/05-dev/agent-events.jsonl` (M1 reuse JSONL pattern từ ADR-001).

**Acceptance Criteria:**

- AC-1.9.1: Mọi action có log với actor, input refs, output refs, model, cost.
- AC-1.9.2: Audit log retention ≥ 1 năm.
- AC-1.9.3: UI cho phép trace 1 estimate ngược về brief gốc trong < 3 click.

---

## 5. Non-Functional Requirements


| #        | Loại          | Yêu cầu                                                                    |
| -------- | ------------- | -------------------------------------------------------------------------- |
| NFR-M1-1 | Performance   | Brief 5 trang → estimate trong < 5 phút end-to-end                         |
| NFR-M1-2 | Performance   | UI response < 2s cho mọi action thông thường                               |
| NFR-M1-3 | Cost          | Cost LLM cho 1 full M1 run ≤ $3 (target average); hard cap $10             |
| NFR-M1-4 | Reliability   | Brief Parser success rate ≥ 95% (không crash, output valid schema)         |
| NFR-M1-5 | Security      | Brief có thể chứa NDA — không log content vào telemetry, chỉ metadata      |
| NFR-M1-6 | i18n          | Brief input và output hỗ trợ Việt + Anh                                    |
| NFR-M1-7 | Accessibility | Web UI WCAG 2.1 AA                                                         |
| NFR-M1-8 | Privacy       | Historical RAG cross-tenant chỉ với explicit opt-in                        |
| NFR-M1-9 | Performance   | WBS + Prototype generation < 3 phút sau khi estimate xong                  |


---

## 6. UI / UX

### 6.1. Page structure

```
ANSO Control Plane
└── Project: ACME ERP HR  (created by Sales/Presale → full M1-M7 tabs visible)
    └── Presale (M1)
        ├── Brief
        │   ├── Upload area (drag-drop: PDF, DOCX, CSV, image, free-text)
        │   ├── Markdown editor (split view với render)
        │   ├── Attachments panel
        │   └── Ambiguity sidebar (clarify questions)
        │
        ├── Technical
        │   ├── Tech stack table (editable per layer)
        │   ├── Architecture overview (text)
        │   ├── Constraints applied panel
        │   └── Override / pin layer controls
        │
        ├── Feature Tree
        │   ├── Tree view (expandable nodes)
        │   ├── Drag-drop reorder
        │   ├── Inline edit complexity
        │   ├── Add/remove feature
        │   └── BA Copilot suggestions panel
        │
        ├── Estimate
        │   ├── Summary card (total md, cost, confidence)
        │   ├── Confidence warning banner (nếu < 0.50)
        │   ├── Method comparison (3 columns)
        │   ├── Per-feature breakdown table
        │   ├── Confidence breakdown (radar chart)
        │   ├── Sensitivity analysis (interactive)
        │   ├── Historical comparison panel
        │   └── Version history sidebar
        │
        ├── WBS
        │   ├── Phase timeline (Gantt-like overview)
        │   ├── WBS table (phase → module → effort)
        │   ├── Effort by role summary
        │   └── Version history sidebar
        │
        ├── Prototype
        │   ├── Screen list (từ feature tree)
        │   ├── Wireframe viewer per screen
        │   ├── Replace with upload (Figma link hoặc image)
        │   └── Status per screen (draft/review/approved)
        │
        ├── Output Package
        │   ├── Deliverables checklist (auto-compiled)
        │   ├── GitHub repo link
        │   ├── Confidence warning nếu < 0.50
        │   ├── [Confirm & Push to GitHub] button
        │   └── [Next → M2] button
        │
        └── Activity Log
            └── Audit trail timeline
```

> **Note về UI gate per role:**
>
> - Sales/Presale (creator): RW trên mọi screen M1.
> - BA / QA / Dev (assigned): **read-only** trên mọi screen M1 (xem brief, estimate, technical, wbs, prototype để có context); không thấy edit buttons.
> - **Cancel Project** button hiển thị cho Sales/Presale creator ở góc phải-trên bất kỳ lúc nào khi `current_phase == M1`.

> **Project được tạo bởi BA (alternative flow M2-start):** workspace KHÔNG có tab "Presale (M1)" — xem M2 PRD §2.4.1.

### 6.2. Key UX patterns

**Pattern 1 — Confidence visibility**
Mọi nơi có estimate, hiển thị confidence rõ ràng (color-coded: red/yellow/green). Confidence < 0.50 → warning banner với danh sách ambiguity, không block action.

**Pattern 2 — Inline ambiguity**
Trong feature tree, feature có ambiguity hiển thị badge [!] với tooltip. 1 click → mở Q&A panel để presale clarify với client.

**Pattern 3 — "Why this estimate?" explainability**
Mọi number trong estimate đều click được → hiển thị breakdown:

- Method nào contribute bao nhiêu.
- Historical project nào được compare.
- Assumption nào đang áp dụng.

**Pattern 4 — Diff-first iteration**
Khi estimate v2 được generate sau khi brief update, default view là **diff** với v1, không phải v2 đứng riêng.

**Pattern 5 — Markdown-first editor**
Mọi artifact mở được trong markdown editor với live preview. Power user có thể edit markdown trực tiếp; non-tech user dùng form-based UI render từ schema.

---

## 7. API & Integration

### 7.1. API Endpoints (Control Plane → Workspace VM)

```
POST   /workspaces/{id}/m1/brief                       # Upload brief content
POST   /workspaces/{id}/m1/brief/attachments           # Upload files (pdf, docx, csv, image)
POST   /workspaces/{id}/m1/feature-tree/extract        # Trigger extraction
GET    /workspaces/{id}/m1/feature-tree                # Read current tree
PUT    /workspaces/{id}/m1/feature-tree                # Update tree
POST   /workspaces/{id}/m1/estimate/generate           # Trigger estimation
GET    /workspaces/{id}/m1/estimate                    # Read estimate
GET    /workspaces/{id}/m1/estimate/versions           # List versions
GET    /workspaces/{id}/m1/estimate/{version}          # Read specific version
POST   /workspaces/{id}/m1/technical/generate          # Trigger tech spec generation
GET    /workspaces/{id}/m1/technical                   # Read technical spec
PUT    /workspaces/{id}/m1/technical                   # Override tech stack layer
POST   /workspaces/{id}/m1/wbs/generate                # Trigger WBS generation
GET    /workspaces/{id}/m1/wbs                         # Read WBS
POST   /workspaces/{id}/m1/prototype/generate          # Trigger prototype generation
GET    /workspaces/{id}/m1/prototype                   # Read prototype index
PUT    /workspaces/{id}/m1/prototype/{screen_id}       # Replace screen (upload/figma)
POST   /workspaces/{id}/m1/output-package/compile      # Compile deliverables list
POST   /workspaces/{id}/m1/output-package/push         # Push all artifacts to GitHub (requires confirm)
POST   /workspaces/{id}/m1/advance                     # Advance M1 → M2 (after push confirmed)

# Project lifecycle (cross-cutting, not module-scoped)
BLOCK /projects/{id}                                  # Hard block project.
                                                        #   - 403 nếu user ≠ creator
                                                        #   - Side-effect: workspace VM destroyed; repo archived (30-day grace period)
POST   /projects/{id}/members                          # Add member: body {user_id, role: ba|qa|dev}
BLOCK /projects/{id}/members/{user_id}                # Block member
```

### 7.2. CLI

Mọi action qua API cũng có CLI tương đương:

```bash
anso m1 brief upload --file RFP.pdf
anso m1 brief upload --file requirements.docx
anso m1 brief upload --file employee-data.csv
anso m1 brief upload --file system-diagram.png
anso m1 feature-tree extract
anso m1 estimate generate
anso m1 estimate show
anso m1 estimate compare v1 v2
anso m1 technical generate
anso m1 technical show
anso m1 wbs generate
anso m1 wbs show
anso m1 prototype generate
anso m1 output-package compile
anso m1 output-package push --confirm
anso m1 advance
```

### 7.3. Knowledge Base / RAG Setup

**Historical projects collection:**


| Field                   | Type      | Notes                                    |
| ----------------------- | --------- | ---------------------------------------- |
| project_id              | string    | Unique                                   |
| client_industry         | string    | Faceted search                           |
| domain                  | string    | erp_hr, ecom, fintech, ...               |
| stack                   | array     | Tech stack tags                          |
| total_features          | int       |                                          |
| feature_tree_embedding  | vector    | 1536-dim OpenAI ada-002 hoặc tương đương |
| brief_summary_embedding | vector    |                                          |
| total_man_days_actual   | int       | Ground truth                             |
| total_man_days_estimate | int       | Original estimate                        |
| deviation_percent       | float     | (actual - estimate) / estimate × 100     |
| started_at              | timestamp |                                          |
| ended_at                | timestamp |                                          |


**Bootstrap data:**

- Phase A early (chưa có dữ liệu): seed 20-30 projects synthesize từ kinh nghiệm BA/Sales senior (gồm PM cũ).
- Sau 6 tháng vận hành: dùng dữ liệu thực ANSO collect được.

**Privacy:**

- Cross-tenant retrieval chỉ với organization opt-in.
- PII anonymized trước khi index.
- Project name có thể replaced với industry+domain only.

---

## 8. Implementation Plan — 6 tuần


| Tuần   | Deliverable                                                               | Acceptance                                      |
| ------ | ------------------------------------------------------------------------- | ----------------------------------------------- |
| **W1** | Sprint 0 dependencies done + M1 scaffold                                  | Workspace VM provision ok, brief.md parse ok     |
| **W2** | Brief ingestion (PDF + DOCX + CSV + image + free text) + UI upload page   | AC-1.1.1, AC-1.1.3 pass                         |
| **W3** | Feature Extraction agent + UI tree view + edit                            | AC-1.2.1 → AC-1.2.5 pass                        |
| **W4** | Estimation Engine — Bottom-up + Three-point + Technical Spec Generator    | AC-1.3.1, AC-1.3.3, AC-1.4.1 pass               |
| **W5** | Analogy method + RAG bootstrap + WBS Generator + Prototype Generator      | AC-1.3.2, AC-1.5.1, AC-1.6.1 pass               |
| **W6** | Output Package + GitHub push + Next→M2 flow + Polish                      | AC-1.7.1, AC-1.7.2, AC-1.8.x, AC-1.9.x pass    |


**Exit criteria for Milestone 1:**

- ANSO M1 đang được dùng cho ≥ 5 deal thật của công ty.
- Average estimate time: < 8 giờ (vs baseline 3-5 ngày).
- ≥ 80% Sales/Presale team có ít nhất 1 estimate qua M1.
- Confidence score được thực sự sử dụng trong sales conversation (qualitative feedback).
- Output package (prototype + deliverables) được gửi cho ≥ 3 client thật.

---

## 9. Test Strategy

### 9.1. Test types


| Type             | Coverage                                                | Tooling           |
| ---------------- | ------------------------------------------------------- | ----------------- |
| Unit test        | Schema validation, scoring formula, RAG retrieval       | Pytest / Jest     |
| Integration test | Brief → feature-tree → estimate → wbs end-to-end        | Pytest + mock LLM |
| LLM eval         | Feature extraction accuracy trên 20 brief test set      | Ragas / custom    |
| Performance test | 5-page brief → full M1 output < 8 min                   | k6 / Artillery    |
| UI E2E test      | Happy path flow                                         | Playwright        |
| Regression test  | Cùng brief → cùng estimate (within 10%)                 | Snapshot test     |


### 9.2. Acceptance test scenarios

**AT-1: Happy path simple brief**

- Input: 3-page brief (PDF), ERP HR domain.
- Expected: Feature tree với ≥ 15 features, estimate trong 5 phút, confidence ≥ 0.65, WBS + prototype generated.

**AT-2: Multi-format brief**

- Input: PDF + DOCX + CSV + image (4 files).
- Expected: All 4 sources reflected trong brief.md, audit trail show từng source, feature tree extracted.

**AT-3: Iterative brief change**

- Input: Brief v1 → estimate v1 → modify brief → estimate v2.
- Expected: V2 reflects changes, diff visible, history preserved, WBS re-generated.

**AT-4: Confidence warning**

- Input: Vague brief với nhiều ambiguity.
- Expected: Confidence < 0.50, **warning banner** hiển thị với danh sách ambiguity, presales vẫn có thể proceed.

**AT-5: Output Package push**

- Input: Tất cả artifacts complete.
- Expected: output-package.md compile đúng danh sách file, presales confirm → push GitHub thành công → project advance M2.

**AT-6: Cost cap**

- Input: Very large brief (50 pages).
- Expected: Cost stay ≤ $10, hoặc graceful degradation với warning.

---

## 10. Decisions Status & Open Issues

### 10.1. Decisions đã chốt


| #   | Quyết định                                 | Giá trị                                     | Tham chiếu       |
| --- | ------------------------------------------ | ------------------------------------------- | ---------------- |
| D1  | 3 estimation methods chạy song song        | Analogy + Bottom-up + Three-point           | §3.4.2           |
| D2  | Confidence < 0.50 → warning, không block   | Warning banner + ambiguity list             | §3.4.3, AC-1.3.6 |
| D3  | Versioning qua git, status `superseded`    | Không xoá file                              | §3.4.4           |
| D4  | Audit trail dạng JSONL append-only         | Reuse pattern ADR-001                       | §4.9             |
| D5  | YAML front matter chạy ngầm                | Không render ra FE; AI internal check only  | §3               |
| D6  | Cross-tenant RAG opt-in                    | Privacy default                             | §7.3             |
| D7  | M1 ship được dùng độc lập                  | Không cần M2-M7                             | §1.1             |
| D8  | Không có lock button                       | Chỉ có [Next → M2] sau khi push GitHub      | §2.3 Step 8      |
| D9  | File export mặc định là `.md`              | Không auto-export DOCX/PDF                  | §3               |
| D10 | WBS ở M1 là high-level phase breakdown     | Không chi tiết task-level (đó là việc M2)   | §3.5             |
| D11 | AI auto-fill YAML từ brief, presales review | Không để trống để fill tay                 | §3.1.2           |


### 10.2. [!] Decisions còn pending


| #   | Quyết định                         | Phương án đề xuất                                                     | Khi nào quyết định      |
| --- | ---------------------------------- | --------------------------------------------------------------------- | ----------------------- |
| P1  | Method weights default             | 0.4/0.4/0.2 phase đầu, calibrate sau                                  | Sau 1 tháng có data     |
| P2  | LLM model cho từng task            | Claude Opus cho extract+estimate, Haiku cho minor                     | Cost benchmark tuần 1-2 |
| P3  | Bootstrap data lấy từ đâu          | Synthesize từ 20-30 BA/Sales senior interview hay xin từ project thực | Tuần 1                  |
| P4  | Cost cap default per full M1 run   | $10 hard cap (proposed)                                               | Cần input từ finance    |
| P5  | Prototype output format            | HTML wireframe vs PNG vs Markdown layout                              | **[TBD — pending Q8]**  |
| P6  | Output Package delivery channel    | GitHub link vs PDF export vs cả hai                                   | **[TBD — pending Q7]**  |


### 10.3. Future considerations

1. **Email-to-brief workflow** — forward email vào dedicated address → auto create project + brief.
2. **CRM integration** — sync với HubSpot / Salesforce để pull deal info.
3. **Multi-currency support** — output package pricing per region.
4. **Custom estimation rubric per organization** — thay vì 1 baseline.
5. **Competitor benchmark** — RAG so với public estimate data.
6. **Capacity planning** — estimate gắn với team availability hiện tại để propose realistic timeline.
7. **Voice/audio brief** — transcribe mp3/m4a → brief.md (Phase 2, nếu có demand).
8. **Figma integration** — fetch wireframe từ Figma link (Phase 2).

---

## 11. Dependencies

### 11.1. Internal (ANSO components)


| Component                            | Status                      | Mức blocking              |
| ------------------------------------ | --------------------------- | ------------------------- |
| Sprint 0 — Workspace VM provisioning | Required                    | Hard                      |
| Sprint 0 — GitHub App integration    | Required                    | Hard                      |
| Sprint 0 — LLM Gateway               | Required                    | Hard                      |
| Sprint 0 — `anso-md` library         | Required                    | Hard                      |
| Control Plane Auth                   | Required                    | Hard                      |
| Vector DB (Qdrant)                   | Required cho Analogy method | Soft (degrade gracefully) |


### 11.2. External


| Service    | Mục đích                      | Fallback          |
| ---------- | ----------------------------- | ----------------- |
| Claude API | Primary LLM                   | OpenAI GPT-4      |
| OpenAI API | Embedding + fallback LLM      | Local model       |
| AWS S3     | Attachments storage           | MinIO self-host   |
| GitHub API | Output package push, repo mgmt | GitLab self-host |


---

## 12. Trigger để re-evaluate Module

M1 nên được re-evaluate (PRD revision) nếu:

1. **Estimate accuracy < 70%** sau 3 tháng — model/method không work, cần redesign.
2. **Adoption rate < 50%** trong presale team sau 2 tháng — UX/value proposition issue.
3. **Cost per full M1 run > $5 average** — không sustainable, cần optimize hoặc đổi model.
4. **Confidence score không correlate với accuracy thực tế** — formula sai.
5. **Stakeholder feedback negative** từ phase B pilot — không phù hợp với customer outsourcing khác.
6. **Industry shift** lớn (vd LLM ecosystem thay đổi căn bản) — cần redesign.
7. **Output package không được client chấp nhận** sau 3 tháng — cần revisit format/content.

---

## 13. Tài liệu tham khảo

- ANSO PRD v1.3, §6.1 (M1 spec gốc), §10 Milestone 1.
- ADR-001: Markdown-First Storage — front matter pattern.
- ADR-002: Workspace-VM Pattern — runtime cho extraction agent.
- *Software Estimation: Demystifying the Black Art* — Steve McConnell (PERT, three-point).
- *Agile Estimating and Planning* — Mike Cohn (story points, planning poker).
- COCOMO II — historical estimation model.
- "Function Point Analysis" — IFPUG.
- Sample outcomes: `Phoenix_TiengAnh_V1.xlsx` (Estimate), `CRM_Estimation_v2.xlsx` (WBS), `edu-safecam-wifi.xlsx` (Technical + Output Package).

---

## 14. Phụ lục

### 14.1. Glossary

- **Brief** — tổng hợp yêu cầu khách hàng, source of truth cho M1.
- **Feature Tree** — cấu trúc cây tính năng có complexity.
- **Technical Spec** — tech stack + architecture overview, AI auto-generate từ brief constraints.
- **Analogy method** — estimation dựa similarity với historical projects (RAG).
- **Bottom-up method** — estimation per-feature × productivity.
- **Three-point / PERT** — (Optimistic + 4×Realistic + Pessimistic) / 6.
- **Confidence Score** — 0-1, đo độ tin cậy estimate.
- **Sensitivity Analysis** — impact của thay đổi feature lên total estimate.
- **Implicit Buffer** — pessimistic_md - realistic_md = contingency từ three-point.
- **WBS** — Work Breakdown Structure, high-level phase breakdown cho client.
- **Prototype** — wireframe/mockup AI-generated từ feature tree.
- **Output Package** — deliverables package tổng hợp giao cho client, push lên GitHub.

### 14.2. Sample brief → output package transformation

(Xem ví dụ end-to-end trong `anso-docs/01-presale/` của project ACME ERP HR — tham chiếu chéo từ các schema trong §3.)

### 14.3. Method Weight Calibration roadmap

```
Phase 1 (Tuần 1-6, M1 đang build):
  weights: bottom_up=0.6, three_point=0.4
  (Analogy chưa có data, không dùng)

Phase 2 (M1 live, 1-3 tháng đầu):
  weights: bottom_up=0.4, three_point=0.2, analogy=0.4
  (Bootstrap data từ 20-30 historical projects)

Phase 3 (3-6 tháng vận hành):
  weights calibrate dựa actual deviation:
  - Method với MAE thấp nhất → weight cao hơn
  - Quarterly review điều chỉnh

Phase 4 (>6 tháng):
  Per-domain weights (ERP có weight khác fintech)
  Per-team weights (team A productivity cao hơn → weight Bottom-up khác)
```

---

*Tài liệu này sẽ được iterate. M1 ship trong 6 tuần là MVP, không phải final state. Mọi feedback gửi về owner.*
