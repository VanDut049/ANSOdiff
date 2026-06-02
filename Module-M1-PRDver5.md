# Module M1 PRD — Presale & Estimation

| Trường | Giá trị |
| ------ | ------ |
| **Module** | M1 — Presale & Estimation |
| **Phiên bản** | 5.0 |
| **Ngày tạo** | 02/05/2026 |
| **Ngày cập nhật** | 02/06/2026 |
| **Owner** | [TBD] |
| **Liên quan** | PRD v1.3 §6.1, §10 (Milestone 1); ADR-001 (Markdown-First); ADR-002 (Workspace-VM) |
| **Milestone** | Milestone 1 — M1 Presale Live (6 tuần sau Sprint 0) |
| **Module ship được dùng độc lập** | Có — không cần M2-M7 |


---

## 1. Tổng quan module

### 1.1. Mục đích

M1 là module **đầu tiên** trong vòng đời SDLC của ANSO, đảm nhiệm giai đoạn presale: từ **brief khách hàng** đến **documents có thể giao cho client**. Đây cũng là module **ship đầu tiên** trong ANSO Phase A (sau Sprint 0).

> **Triết lý:** M1 không thay thế Sales/Presale, mà **biến brief thô thành estimate có cấu trúc, technical spec, WBS, prototype và documents — có audit trail, có khả năng tái sử dụng**.

### 1.2. Vấn đề cụ thể đang giải quyết

Outsourcing companies hiện tại có pain rõ rệt ở giai đoạn presale:

| Pain | Hiện trạng | Hệ quả |
|------|-----------|--------|
| Estimate lâu | 3-5 ngày cho brief 5-10 trang | Lỡ deal vì client chờ; BA/senior bị ngắt việc khác |
| Phụ thuộc senior cá nhân | Chỉ 2-3 senior estimate được | Bottleneck; junior không có cơ hội học |
| Không tận dụng dữ liệu lịch sử | Mỗi estimate làm lại từ đầu | Sai số lớn (50-100%); estimate không cải thiện theo thời gian |
| Format không đồng nhất | Mỗi người một template | Client review khó; brand inconsistent |
| Thiếu confidence info | Estimate là 1 số duy nhất | Client không biết rủi ro; sales khó negotiate |
| Khó iterate | Brief đổi → estimate làm lại từ đầu | Không support agile presale |
| Output cho client thiếu sức nặng | Chỉ có file estimate đơn giản | Client khó hình dung sản phẩm; thiếu technical credibility |

### 1.3. Outcome target sau khi M1 live

| Metric | Trước M1 | Sau M1 (3 tháng) |
|------|------|------|
| Thời gian từ brief → documents | 3-5 ngày | 4-8 giờ |
| Số người làm được estimate | 2-3 senior (BA/PM cũ) | Mọi Sales/Presale (qua tool) |
| Sai số estimate | 50-100% | ≤ 20% (target KPI) |
| Số deal có thể serve song song | 5-10/tháng | 30-50/tháng |
| Confidence score communicate với client | Không có | 100% deal có |
| Documents chất lượng cho client | Không có | Prototype + documents |

### 1.4. Out of Scope (M1 MVP)

Các tính năng sau **không thuộc phạm vi M1 MVP** — có thể xem xét ở phase sau:

- **Multi-scenario estimate side-by-side** (MVP thuần / MVP + extend / MVP + extend + add-on): M1 không auto-generate parallel scenarios trong cùng một session. Presale có thể tạo nhiều version estimate riêng biệt qua iteration flow (brief update → re-estimate), nhưng UI không hỗ trợ so sánh song song các scenario trong M1.
- **Real-time co-editing** nhiều người cùng lúc trên 1 artifact.
- **Client-facing portal** — client không tương tác trực tiếp với M1 workspace.
- **Tích hợp CRM** (HubSpot, Salesforce) — xem §10.3 Future considerations.
- **Voice/audio brief** input — xem §10.3.

---

## 2. Personas & User Flows

### 2.1. Primary Personas

**P1 — Sales/Presale ("Tuấn")**

- Vai trò: nhận brief, làm estimate, tạo documents, negotiate với client, manage project ở phase presale, advance sang M2.
- Tần suất: 3-5 brief/tuần, theo deal pipeline.
- Pain: estimate tốn thời gian (3-5 ngày), phụ thuộc PM senior cũ, format lại liên tục, khó defend confidence trước client.
- Mục tiêu với M1: tự độc lập làm 80% estimate, chỉ escalate case phức tạp; confidence score + rationale rõ để dùng trong sales conversation; advance project sang M2 (BA team) khi documents sẵn sàng.

### 2.2. Secondary Personas

- **BA** (downstream "Lan"): nhận estimate đã chốt làm input cho M2.
- **Client** (qua documents): nhận Prototype + Brief + Estimate, không tương tác trực tiếp với M1.
- **Dev** (downstream "Khoa"): review tech stack đề xuất khi feature có complexity cao.

> **Note về visibility:** BA, QA, Dev đã được assign vào project ngay từ phase M1 (qua `Sales/Presale` add member) có **read-only access** vào artifact của M1 (brief, feature-tree, estimate, technical, wbs, prototype) để có context — ANSO-PRD §7.6.6.

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
│ Output: feature list (draft) — embedded trong estimate.md   │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 4: Review & Edit Feature List                          │
│ Actor: Sales/Presale                                        │
│ ANSO Action: BA Copilot suggest missing features            │
│ Output: feature list (refined) trong estimate.md            │
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
│   WBS Generator — từ feature list + estimate                │
│   Prototype Generator — Hi-fi prototype từ feature list     │
│ Output: anso-docs/01-presale/wbs.md                         │
│         anso-docs/01-presale/prototype/prototype.html       │
│         (PNG generate ngầm cho export, không hiển thị UI)   │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 7: Review & Edit All Artifacts                         │
│ Actor: Sales/Presale                                        │
│ Action: Review brief, estimate (incl. feature list),        │
│         technical, wbs, prototype                           │
│ Output: All artifacts refined, ready for documents      │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 8: Finish & Advance to M2                              │
│ Actor: Sales/Presale click "Next → M2"                      │
│ ANSO Action:                                                │
│   Visibility Folder (right panel) — push .md:              │
│     Brief        → brief.md                                 │
│     Estimate     → estimate.md                              │
│     WBS          → wbs.md                                   │
│     Technical    → technical.md                             │
│     Prototype    → prototype.md                             │
│     Deliverables → deliverables.md                          │
│   Mỗi artifact có nút Export modal — on-demand khi user     │
│   bấm vào file muốn xuất:                                   │
│     Brief        → Brief_NNN.docx                           │
│     Estimate     → Estimate_NNN.xlsx                        │
│     WBS          → WBS_NNN.xlsx                             │
│     Technical    → Technical_NNN.xlsx                       │
│     Prototype    → prototype.html + PNG                     │
│     Deliverables → Deliverables_NNN.xlsx                    │
│   Export generate lúc user trigger — không auto-generate    │
│   toàn bộ khi advance (tiết kiệm tài nguyên)                │
│   Sang M2: chỉ .md được carry over vào Visibility Folder    │
│   phase tiếp theo; export format generate lại nếu cần       │
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
        ANSO diff: feature list v1 vs v2 → highlight changes
        ANSO recompute estimate, wbs, show delta
                ↓
        Estimate v2, WBS v2 (có git history với v1)
```

Quan trọng: **không** bắt đầu lại từ đầu — ANSO biết artifact đã có, chỉ update phần thay đổi.

---

### 2.5. Project Lifecycle Controls

M1 là module **đầu tiên** trong vòng đời project (khi Sales/Presale là creator). Các rule lifecycle áp dụng cho project ở phase M1:

**Creator quyền hạn:**

- Sales/Presale (creator) có quyền **cancel project** bất kỳ lúc nào khi `current_phase == M1`.
- Sales/Presale (creator) có quyền **add/cancel/swap member** (BA, QA, Dev) bất kỳ lúc nào.
- Sales/Presale (creator) có quyền **edit RW** trên toàn bộ artifact M1 (brief, technical, feature list, estimate, wbs, prototype).
- Sau khi click "Next → M2": project chuyển phase, Sales/Presales không được cancel project, BA team unlock access M2.

**Member visibility tại M1:**

- BA, QA, Dev được assign sẽ thấy project trong list của họ, kể cả khi project đang ở phase M1.
- Member click vào project → **read-only access full** vào artifact M1 (xem brief, estimate, technical, wbs, prototype) — purpose: cho member chuẩn bị context và capacity, không edit được ở phase M1.
- Member chưa được assign **KHÔNG** thấy project.

**Multiple members per role:** project có thể có N BA, N QA, N Dev (vd 3 Dev) — tất cả đều có read-only access.

Cross-reference: chi tiết permission matrix tại **ANSO-PRD §7.6**.

---

## 3. Artifacts & Schema

> Mọi artifact tuân theo ADR-001 §4.3 (front matter + markdown body). YAML front matter chạy ngầm để AI internal check — không render ra FE. File export mặc định là `.md`. Khi presale tải xuống, file được đặt tên theo convention `[File]_NNN.md` (vd: `Brief_001.md`, `Estimate_004.md`, `WBS_001.md`, `Technical_001.md`, `Prototype_005.md`). Phần này định nghĩa schema cụ thể cho từng artifact của M1.

### 3.1. Brief (`anso-docs/01-presale/brief.md`)

**Vai trò:** Single source of truth về yêu cầu khách hàng — tổng hợp từ mọi nguồn input.

#### 3.1.1. Schema

```yaml
---
id: BRIEF-001
type: brief
title: ACME Corp — ERP Module HR
version: 1                  # mỗi version mới tăng số
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
    path: 00-attachments/RFP-ACME-HR.pdf
    pages: 12
    received_at: 2026-04-26T09:30:00Z
  - type: docx
    path: 00-attachments/requirements.docx
    received_at: 2026-04-26T09:35:00Z
  - type: csv
    path: 00-attachments/employee-data-sample.csv
    received_at: 2026-04-26T09:40:00Z
  - type: image
    path: 00-attachments/system-diagram.png
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
- RFP đính kèm: 00-attachments/RFP-ACME-HR.pdf
- Requirements doc: 00-attachments/requirements.docx
- Sample data: 00-attachments/employee-data-sample.csv
- System diagram: 00-attachments/system-diagram.png

> **Note:** File upload qua chat UI chỉ là entry point — source thực tế của tất cả input files luôn resolve về `00-attachments/[tên file].[format]`. Folder `00-attachments/` dành riêng cho file input từ bên ngoài do Presales hoặc BA cung cấp.
```

#### 3.1.2. Quy tắc

- **YAML front matter** được AI auto-fill khi parse brief upload; presales chỉ review và correct. YAML chạy ngầm, không render ra FE.
- Front matter `input_sources` track mọi nguồn → audit trail và RAG retrieval reference.
- `ambiguities` được ANSO sinh tự động khi parse brief, presale có thể manually thêm.
- Brief có thể có nhiều version qua git — version mới nhất luôn là file active.
- **Attachment path convention:** Tất cả file input (PDF, DOCX, CSV, image) sau khi upload được lưu tại `00-attachments/[tên file].[format]`. Upload qua chat UI chỉ là entry point — path trong YAML `input_sources` và body `## Reference & attachments` luôn reference folder này. Folder `00-attachments/` dành riêng cho file input từ bên ngoài do Presales hoặc BA cung cấp (không dùng cho output artifacts).

---

### 3.2. Estimate (`anso-docs/01-presale/estimate.md`)

**Vai trò:** Estimate chi tiết theo nhiều phương pháp, có confidence score. Bao gồm feature list (trước đây là Feature Tree riêng) làm input trực tiếp cho estimation.

#### 3.2.1. Feature List — Complexity Rubric

ANSO đánh complexity dựa rubric sau (áp dụng cho mọi feature trong Estimate):

| Code    | Mô tả                                         | Man-day estimate (single dev, mid-level) |
| ------- | --------------------------------------------- | ---------------------------------------- |
| **XS**  | Trivial change/setup                          | < 0.5                                    |
| **S**   | Single endpoint/screen, ít edge case          | 0.5 - 1                                  |
| **M**   | Multiple endpoint/screen, moderate logic      | 1 - 3                                    |
| **L**   | Complex business logic, multiple integrations | 3 - 7                                    |
| **XL**  | Đa module, nhiều unknowns                     | 7 - 15                                   |
| **XXL** | Cần break thêm                                | > 15 (warning)                           |

Quy tắc feature list:
- Mỗi feature ID format: `F-NNN` hoặc `F-NNN.N` (sub-feature).
- Complexity gắn ở `[X]` cuối tên feature.
- ANSO **không gán XXL tự động** — flag để presale break down thêm.
- Tree có thể có 2-3 cấp depth tối đa; sâu hơn → cân nhắc thành module riêng.
- `ambiguity_flags` list các feature ID cần presale clarify trước khi estimate.

#### 3.2.2. Schema

```yaml
---
id: EST-001
type: estimate
title: ACME ERP HR — Estimate v1
version: 1                  # mỗi version mới tăng số
created_at: 2026-04-26T14:00:00Z
updated_at: 2026-04-26T14:30:00Z
created_by: estimate-engine
updated_by: estimate-engine
schema_version: 1

brief_ref: BRIEF-001

methods_used: [analogy, bottom_up, three_point]

# Feature list metadata
total_features: 47
total_features_by_complexity:
  XS: 0
  S: 18                     # 0.5 - 1 man-day
  M: 15                     # 1 - 3 man-day
  L: 10                     # 3 - 7 man-day
  XL: 4                     # 7 - 15 man-day
derived_from:
  - input_source: RFP-ACME-HR.pdf
    confidence: 0.85
  - input_source: requirements.docx
    confidence: 0.80
  - input_source: system-diagram.png
    confidence: 0.72
ambiguity_flags: [F-023, F-031]

summary:
  total_man_days: 187
  total_man_days_by_role:
    ba: 23
    qa: 24
    dev: 140
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
---

# Estimate v1: ACME ERP HR

## Feature List

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

## Tổng quan

| Metric           | Giá trị                                      |
| ---------------- | -------------------------------------------- |
| Total effort     | 187 man-day                                  |
| Calendar timeline | 16 tuần với team 4 người                    |
| Cost range       | $62,000 - $78,000                            |
| Confidence       | Medium-High (0.78)                           |
| Implicit buffer  | 58 man-day (pessimistic - realistic)         |

## Detailed breakdown by feature

### F-001: Authentication & Authorization (L — 12 md)

| Sub     | Sub content                        | Complexity | BA  | BE  | FE  | QA  | DevOps | Total |
| ------- | ---------------------------------- | ---------- | --- | --- | --- | --- | ------ | ----- |
| F-001.1 | Email/password login               | S          | 0.5 | 1   | 0.5 | 0.5 | -      | 2.5   |
| F-001.2 | SSO Azure AD integration           | M          | 1   | 2   | 1   | 1   | 0.5    | 5.5   |
| F-001.3 | 2FA via OTP                        | M          | 0.5 | 1.5 | 0.5 | 0.5 | -      | 3.0   |
| F-001.4 | Role-based access control (5 roles)| L          | 1   | 3   | 1   | 1   | -      | 6.0   |
| F-001.5 | Session management                 | S          | 0.5 | 0.5 | 0.5 | 0.5 | -      | 2.0   |

### F-002: Employee Profile Management (XL — 24 md)

| Sub     | Sub content                        | Complexity | BA  | BE  | FE  | QA  | DevOps | Total |
| ------- | ---------------------------------- | ---------- | --- | --- | --- | --- | ------ | ----- |
| F-002.1 | Personal info CRUD                 | M          | 1   | 2   | 2   | 1   | -      | 6.0   |
| F-002.2 | Document upload (CV, contracts)    | M          | 0.5 | 2   | 1   | 0.5 | -      | 4.0   |
| F-002.3 | Organization chart visualization   | L          | 1   | 2   | 3   | 1   | -      | 7.0   |
| F-002.4 | Employee history tracking          | M          | 1   | 2   | 1   | 1   | -      | 5.0   |
| F-002.5 | Search & filter                    | M          | 0.5 | 1   | 1   | 0.5 | -      | 3.0   |

...

## Method comparison

| Method               | Estimate (md) | Notes                                          |
| -------------------- | ------------- | ---------------------------------------------- |
| Analogy-based        | 195           | Dựa trên 3 project ERP HR tương tự             |
| Bottom-up            | 178           | Per-feature × productivity rate                |
| Three-point PERT     | 192           | (O + 4R + P) / 6                               |
| **Final (weighted)** | **187**       | 0.4×Analogy + 0.4×Bottom-up + 0.2×PERT         |

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

#### 3.2.3. 3 Estimation Methods

**Method 1 — Analogy-based (RAG)**

```
1. Embed feature list → vector
2. Search Qdrant trong historical projects (cross-tenant với consent)
3. Top-K similar projects (default K=5, similarity > 0.6)
4. Weight by similarity, lấy weighted average actual_man_days
5. Adjust theo project meta (team size, stack)
```

Ưu: dùng dữ liệu thật. Nhược: cần dữ liệu lịch sử (cold start vấn đề).

**Method 2 — Bottom-up**

```
For each feature in feature list:
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

#### 3.2.4. Confidence Score Formula

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

| Confidence range | Label       | Hành động đề xuất                                                        |
| ---------------- | ----------- | ------------------------------------------------------------------------ |
| ≥ 0.85           | High        | Có thể generate documents trực tiếp                                     |
| 0.70 - 0.85      | Medium-High | Recommend BA review trước khi gửi client                                |
| 0.50 - 0.70      | Medium      | Bắt buộc BA review, có thể cần more discovery                           |
| 0.30 - 0.50      | Low-Medium  | **Cảnh báo:** cần discovery phase trước khi commit                      |
| < 0.30           | Low         | **Cảnh báo rủi ro cao:** recommend không advance, list ambiguity rõ ràng |

> **Thay đổi v2:** Confidence < 0.50 không còn hard-block documents generation. Thay bằng **warning rõ ràng** với danh sách ambiguity — presales tự quyết định có proceed không.

#### 3.2.5. Versioning

- Mỗi estimate có `version` field, tăng dần.
- Estimate cũ giữ trong git history, `status: superseded`.
- File `estimate.md` luôn là version mới nhất; version cũ xem qua `git log` hoặc UI history.
- Khi presale tải xuống: `Estimate_NNN.md` (vd `Estimate_001.md`).

---

### 3.3. WBS (`anso-docs/01-presale/wbs.md`)

**Vai trò:** Work Breakdown Structure high-level theo phase — đủ để client thấy cấu trúc dự án, không cần chi tiết task-level. AI auto-generate từ feature list + estimate.

#### 3.3.1. Schema

```yaml
---
id: WBS-001
type: wbs
title: ACME ERP HR — Work Breakdown Structure
created_at: 2026-04-26T15:00:00Z
updated_at: 2026-04-26T15:00:00Z
created_by: wbs-generator-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
estimate_ref: EST-001

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

| Phase                            | Timing      | Milestone            | Notes                                                      |
| -------------------------------- | ----------- | -------------------- | ---------------------------------------------------------- |
| Phase 0: Setup & Architecture    | Week 1-2    | Kickoff & design     | Scope confirm, DB design, architecture, UI wireframe       |
| Phase 1: Core Development        | Week 2-10   | Phase 1 delivery     | Auth, Employee Profile, Time Attendance                    |
| Phase 2: Integration & Advanced  | Week 10-14  | Phase 2 delivery     | SAP integration, Payroll, Reports                          |
| Phase 3: Testing & Launch        | Week 14-16  | Final acceptance     | UAT, bug fix, deploy, handover                             |

## Detailed WBS

| No | Code    | Module / Group          | Task                                      | Complexity  | Effort (MD) | Role                  |
| -- | ------- | ----------------------- | ----------------------------------------- | ----------- | ----------- | --------------------- |
| 1  | F-001   | Phase 0: Setup          | Infrastructure & Architecture             | —           | 12          | DevOps + Tech Lead    |
| 2  | F-001.1 |                         | DB schema design                          | Simple      | 4           | Tech Lead             |
| 3  | F-001.2 |                         | Core API server setup (NestJS, Auth)      | Medium      | 6           | BE Engineer           |
| 4  | F-001.3 |                         | CI/CD setup (GitHub Actions), Docker      | Simple      | 2           | DevOps                |
| 5  | F-002   | Phase 1: Core           | Authentication & Authorization            | —           | 14          | Full-stack            |
| 6  | F-002.1 |                         | Email/password login                      | Simple      | 2           | Full-stack            |
| 7  | F-002.2 |                         | SSO Azure AD integration                  | Medium      | 5           | BE + FE               |
| 8  | F-002.3 |                         | 2FA via OTP                               | Simple      | 2           | BE + FE               |
| 9  | F-002.4 |                         | RBAC (5 roles)                            | Complex     | 5           | BE Engineer           |
| 10 | F-003   |                         | Employee Profile Management               | —           | 24          | Full-stack            |
| 11 | F-003.1 |                         | Personal info CRUD                        | Medium      | 6           | Full-stack            |
| 12 | F-003.2 |                         | Document upload                           | Medium      | 4           | BE + FE               |
| 13 | F-003.3 |                         | Organization chart visualization          | Complex     | 7           | FE Engineer           |
| 14 | F-003.4 |                         | Employee history tracking                 | Medium      | 5           | BE Engineer           |
| 15 | F-003.5 |                         | Search & filter                           | Medium      | 3           | Full-stack            |
| …  | …       | …                       | …                                         | …           | …           | …                     |
| N  | F-00N   | Phase 3: Launch         | UAT & Deployment                          | —           | 15          | QA + DevOps           |

## Effort Summary by Role

| Role                                  | Effort (MD) | Man-months (1mm = 22 days) |
| ------------------------------------- | ----------- | -------------------------- |
| BA (incl. PM/Delivery Lead)           | 23          | 1.05                       |
| Dev (BE + FE + DevOps consolidated)   | 140         | 6.36                       |
| QA                                    | 24          | 1.09                       |
| **Total**                             | **187**     | **8.50**                   |
```

#### 3.3.2. Quy tắc

- WBS ở M1 là **high-level phase breakdown** — không đi sâu đến sub-task level (đó là việc của M2/BA).
- AI generate dựa trên feature list modules → group thành phases logic.
- Mã code theo format `F-NNN` hoặc `F-NNN.N` — đồng bộ với feature list trong estimate.md.
- Mỗi line item có complexity (Very Simple / Simple / Medium / Complex / Very Complex) tham chiếu từ `CRM_Estimation_v2.xlsx` guideline.
- Total effort WBS phải khớp với `estimate.md` total_man_days.

---

### 3.4. Technical (`anso-docs/01-presale/technical.md`)

**Vai trò:** Tech stack và architecture overview — AI auto-suggest từ brief + constraints, presales review.

#### 3.4.1. Schema

```yaml
---
id: TECH-001
type: technical_spec
title: ACME ERP HR — Technical Specification
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

| # | Layer             | Framework / Language   | Purpose                     | Rationale                                  |
| - | ----------------- | ---------------------- | --------------------------- | ------------------------------------------ |
| 1 | Framework         | React 18 / Next.js 14  | Cross-platform web app      | Reduced dev cost, near-native performance  |
| 2 | State Management  | Redux Toolkit / Zustand| App state (auth, data cache)| Scalable, testable, async-friendly         |

## 2. Backend & API

| # | Layer          | Solution                      | Purpose                         | Notes                              |
| - | -------------- | ----------------------------- | ------------------------------- | ---------------------------------- |
| 1 | Runtime        | Node.js 20+                   | API server, auth, logging       | High performance, rich ecosystem   |
| 2 | Framework      | NestJS / Express              | REST API framework              | Structured, TypeScript-first       |
| 3 | Database       | PostgreSQL 16                 | Users, HR records, audit logs   | ACID, mature, full-text search     |
| 4 | Authentication | Firebase Auth / JWT + OAuth 2.0 | User auth, Azure AD SSO       | Built-in MS/Google login integration |
| 5 | ORM            | Prisma / TypeORM              | PostgreSQL ORM                  | Type-safe, migration management    |

## 3. Infrastructure & DevOps

| # | Layer            | Platform                  | Purpose                        | Config                          |
| - | ---------------- | ------------------------- | ------------------------------ | ------------------------------- |
| 1 | Hosting          | AWS EC2 / GCP Cloud Run   | Backend API server             | Auto-scale, high reliability    |
| 2 | CI/CD            | GitHub Actions            | Automated build/test/deploy    | Direct GitHub integration       |
| 3 | Containerization | Docker + Docker Compose   | Backend packaging              | Reproducible deploy             |
| 4 | Monitoring       | Firebase Crashlytics + Sentry | Crash/error tracking       | Real-time crash reports         |

## 4. Security & Compliance

| # | Aspect                         | Solution                    | Purpose                  |
| - | ------------------------------ | --------------------------- | ------------------------ |
| 1 | Authentication & Authorization | JWT Auth                    | Secure user access       |
| 2 | Password & Token Security      | bcrypt + HTTPS (TLS)        | Data encryption          |
| 3 | API Security                   | Helmet + CORS + Rate limiter| DDoS/XSS/CSRF protection |
| 4 | Data Privacy                   | AES encryption for PII      | User data protection     |

## 5. Development Tools

| # | Tool                       | Purpose                        |
| - | -------------------------- | ------------------------------ |
| 1 | VSCode + ESLint + Prettier | Coding, linting, formatting    |
| 2 | Postman                    | API testing                    |
| 3 | Git + GitHub               | Source control                 |
| 4 | pgAdmin                    | Database management            |

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


#### 3.4.2. Quy tắc

- AI auto-generate từ `brief.md` constraints + feature list domain — presales review và override từng layer.
- Mỗi lựa chọn tech có `rationale` rõ ràng — dùng trong sales conversation khi client hỏi "tại sao chọn X?".
- YAML `stack` chạy ngầm để AI validate consistency (vd nếu brief có SAP constraint mà tech stack không có adapter → flag).
- Body markdown là human-readable table — đây là phần presales show cho client/Dev review.

---

### 3.5. Prototype (`anso-docs/01-presale/prototype/`)

**Vai trò:** Wireframe để client hình dung sản phẩm. AI auto-generate từ feature list — đủ để show sườn màn hình chính, không cần pixel-perfect.

#### 3.5.1. Schema

```yaml
---
id: PROTO-001
type: prototype
title: ACME ERP HR — Prototype
created_at: 2026-04-26T16:00:00Z
updated_at: 2026-04-26T16:00:00Z
created_by: prototype-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
estimate_ref: EST-001

tool: ai_generated            # ai_generated | figma_upload | image_upload
fidelity: hi_fi               # hi_fi: có style, màu sắc, component đầy đủ
output_format: "HTML single-file (multi-screen, basic routing) + PNG (export only)"
# prototype.html chứa tất cả màn hình, navigate in-page
# PNG generate ngầm — không hiển thị trong workspace, chỉ available qua Export modal

screens:
  - id: SCR-001
    name: Login / Authentication
    features: [F-001.1, F-001.2]
  - id: SCR-002
    name: Employee Profile Management
    features: [F-002.1, F-002.2, F-002.3]
  - id: SCR-003
    name: Time Attendance Dashboard
    features: [F-003.1, F-003.2]
  - id: SCR-004
    name: Organization Chart
    features: [F-002.3]

prototype_path: prototype/prototype.html   # single file duy nhất
---

# Prototype: ACME ERP HR

## Danh sách màn hình

| Screen                              | Features               |
| ----------------------------------- | ---------------------- |
| SCR-001: Login / Authentication     | F-001.1, F-001.2       |
| SCR-002: Employee Profile           | F-002.1 → F-002.5      |
| SCR-003: Time Attendance            | F-003.1, F-003.2       |
| SCR-004: Organization Chart         | F-002.3                |

> Toàn bộ màn hình được render trong 1 file `prototype/prototype.html`.
> Navigation giữa các màn hình qua routing in-page (sidebar/tabs).
> Xem file: `prototype/prototype.html`
```

#### 3.5.2. Quy tắc

- AI generate **1 file HTML duy nhất** (`prototype/prototype.html`) chứa toàn bộ màn hình, với basic in-page routing (sidebar navigation hoặc tabs để chuyển màn hình).
- Fidelity: **Hi-fi** — có style, màu sắc, component đầy đủ đủ để client hình dung sản phẩm thực. Không phải lo-fi wireframe.
- **PNG**: generate ngầm per screen (dùng headless render) — không hiển thị trong Prototype workspace tab. PNG chỉ available qua Export modal trong Visibility Folder.
- Có thể upload ảnh mockup / Figma export từ client thay thế AI-generated screen (replace toàn bộ prototype.html).
- Prototype chỉ cần show màn hình **chính** (không cần edge case, error state).

---

### 3.6. Deliverables (`anso-docs/01-presale/deliverables.md`)

**Vai trò:** Danh sách thành phẩm bàn giao cho client — tổng hợp toàn bộ output của dự án theo nhóm công việc. AI auto-suggest từ feature list + technical spec; presale review và chỉnh sửa trước khi giao client. Export dưới dạng `.xlsx`.

#### 3.6.1. Schema

```yaml
---
id: DELIV-001
type: deliverables
title: ACME ERP HR — Deliverables List
version: 1
created_at: 2026-04-26T16:30:00Z
updated_at: 2026-04-26T16:30:00Z
created_by: deliverables-agent
updated_by: tuan.presale
schema_version: 1

brief_ref: BRIEF-001
estimate_ref: EST-001
technical_ref: TECH-001

total_groups: 6
---

# Deliverables List: ACME ERP HR

## 1. Thiết kế & Kiến trúc

| Tên công việc | Nội dung giao | Chi tiết | Lưu/Nộp | Định dạng |
| --- | --- | --- | --- | --- |
| Thiết kế hệ thống & Kiến trúc | Tài liệu kiến trúc hệ thống | Thiết kế kiến trúc, sơ đồ luồng dữ liệu, tài liệu API | GitHub Repository | `.md` |
| Wireframe & Thiết kế UI/UX | Hi-fi Prototype | Các màn hình chính của ứng dụng (single HTML, multi-screen) | Figma / Google Drive | `.html` / `.pdf` |

## 2. Ứng dụng Mobile (nếu có)

| Tên công việc | Nội dung giao | Chi tiết | Lưu/Nộp | Định dạng |
| --- | --- | --- | --- | --- |
| Source code ứng dụng mobile | Mobile App Source Code | iOS/Android application (Flutter/React Native hoặc native) | GitHub Repository (Private) | Source Code |
| Build file | Build file (APK/IPA) | File cài đặt dùng để test và triển khai nội bộ | Google Drive / TestFlight / Firebase Distribution | `.apk` / `.ipa` |

## 3. Backend & API

| Tên công việc | Nội dung giao | Chi tiết | Lưu/Nộp | Định dạng |
| --- | --- | --- | --- | --- |
| Backend source code | Backend Source Code | Authentication, user management, business logic, integrations | GitHub Repository (Private) | Source Code |
| Tài liệu API | API Document | Toàn bộ endpoints dạng Swagger/OpenAPI spec | GitHub Repository | `.yaml` / `.md` |

## 4. Admin Panel (nếu có)

| Tên công việc | Nội dung giao | Chi tiết | Lưu/Nộp | Định dạng |
| --- | --- | --- | --- | --- |
| Source code quản trị | Admin Web Source Code | Quản lý user, dữ liệu, log sử dụng | GitHub Repository (Private) | Source Code |

## 5. Tài liệu

| Tên công việc | Nội dung giao | Chi tiết | Lưu/Nộp | Định dạng |
| --- | --- | --- | --- | --- |
| Hướng dẫn sử dụng | User Manual | Tài liệu hướng dẫn chi tiết cho người dùng cuối (có hình minh hoạ) | Google Drive | `.pdf` |
| Hướng dẫn triển khai | Deployment Guide | Cấu hình server, cài đặt, kết nối API, môi trường production | GitHub Repository | `.md` |

## 6. Kiểm thử & QA

| Tên công việc | Nội dung giao | Chi tiết | Lưu/Nộp | Định dạng |
| --- | --- | --- | --- | --- |
| Báo cáo kiểm thử | Test Report | Test case, kết quả test, báo cáo bug MVP và bản mở rộng | Google Drive | `.xlsx` / `.pdf` |
```

#### 3.6.2. Quy tắc

- AI auto-suggest nội dung bảng Deliverables dựa trên feature list (estimate.md) + technical spec (technical.md) — presale review và chỉnh sửa.
- Cấu trúc 6 nhóm mặc định (có thể thêm/bỏ nhóm tùy project): Thiết kế & Kiến trúc / Mobile App / Backend & API / Admin Panel / Tài liệu / Kiểm thử & QA.
- Mỗi row có 5 cột: **Tên công việc** / **Nội dung giao** / **Chi tiết** / **Lưu/Nộp** / **Định dạng**.
- Presale có thể thêm row, xoá row, sửa nội dung inline trong UI.
- Export `.xlsx` qua Export modal trong Visibility Folder — format giữ nguyên bảng 5 cột, group theo nhóm.
- Khi export `.xlsx`, header row có màu nền xanh đậm (consistent với sample format).

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
- AC-1.1.4: Sau khi upload, tất cả file được move vào `00-attachments/`. Path trong brief.md YAML (`input_sources[].path`) và body (`## Reference & attachments`) luôn reference folder này. Folder dành riêng cho file input từ bên ngoài (Presales hoặc BA).

### 4.2. Feature Extraction (F1.2)

**Brief Parser Agent flow:**

```
1. Read brief.md + all attachments (resolved content)
2. LLM prompt: "Extract feature list from this requirement"
3. Validate schema (front matter)
4. Detect ambiguity → flag
5. Suggest missing common features (auth, audit log, ...)
6. Write feature list vào estimate.md
```

**Acceptance Criteria:**

- AC-1.2.1: Cho brief 5-10 trang → output feature list với ≥ 80% feature được extract đúng (so với manual).
- AC-1.2.2: Mỗi feature có complexity gắn.
- AC-1.2.3: Ambiguity flag chính xác ≥ 70% (precision).
- AC-1.2.4: Feature list có 2-3 cấp depth, không nested quá sâu.
- AC-1.2.5: Edit thủ công feature list → không bị overwrite khi re-trigger nếu đã có content (chỉ append/suggest, không destroy).

### 4.3. Estimation Engine (F1.3)

**Đã chi tiết ở §3.2.3.**

**Acceptance Criteria:**

- AC-1.3.1: 3 methods chạy song song, total time < 2 phút cho 50 features.
- AC-1.3.2: Final estimate = weighted average với rationale rõ ràng.
- AC-1.3.3: Confidence score formula deterministic — cùng input cho cùng output.
- AC-1.3.4: Method comparison table luôn hiển thị (transparency).
- AC-1.3.5: Confidence < 0.50 → hiển thị **warning** rõ ràng kèm danh sách ambiguity; không hard-block.

### 4.4. Technical Spec Generator (F1.4)

**Technical Spec Agent flow:**

```
1. Read brief.md (constraints, domain, stack preferences)
2. Read feature list (feature categories → infer tech needs)
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
1. Read feature list (features, complexity) từ estimate.md
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
1. Read feature list → identify main screens (top-level modules)
2. For each screen: map features → UI components + layout
3. Generate 1 file prototype.html (single file, multi-screen):
   - Semua màn hình trong 1 file
   - Basic in-page routing (sidebar navigation / tabs)
   - Hi-fi: có style, màu sắc, component đầy đủ
4. Generate PNG per screen ngầm (headless render) → lưu riêng cho export
5. Write prototype/prototype.html + update prototype.md index
```

**Acceptance Criteria:**

- AC-1.6.1: Mỗi top-level feature module → ít nhất 1 screen trong prototype.html.
- AC-1.6.2: Mỗi screen có đủ: tên màn hình, feature IDs liên quan, layout Hi-fi (style, màu sắc, component đầy đủ để client hình dung sản phẩm thực).
- AC-1.6.3: Presales có thể replace toàn bộ AI-generated prototype bằng upload (Figma export hoặc HTML).
- AC-1.6.4: Output: **1 file `prototype.html` duy nhất**, multi-screen với basic in-page navigation (sidebar/tabs). PNG generate ngầm per screen — không render trong Prototype workspace, chỉ available qua Export modal.

### 4.7. Versioning & Iteration (F1.7)

**Acceptance Criteria:**

- AC-1.7.1: Mỗi artifact thay đổi → git commit auto với message rõ ("brief updated by tuan", "estimate v2 generated").
- AC-1.7.2: UI hiển thị version history với diff view.
- AC-1.7.3: Compare 2 versions estimate → hiển thị delta theo feature, total.
- AC-1.7.4: Rollback estimate sang version cũ qua UI 1 click.

### 4.8. Audit Trail (F1.8)

**Mọi action của ANSO/agent ghi log:**

```jsonl
{"ts":"2026-04-26T11:00:00Z","actor":"brief-parser-agent","action":"extract_features","input":"brief.md@abc123","output":"estimate.md@def456","model":"claude-opus-4","tokens_in":12500,"tokens_out":3200,"cost_usd":0.42,"duration_s":45}
{"ts":"2026-04-26T14:00:00Z","actor":"estimate-engine","action":"compute_estimate","method":"analogy","retrieved_projects":["PROJ-2024-018","PROJ-2024-031","PROJ-2025-007"],"output":"estimate.md@ghi789"}
{"ts":"2026-04-26T15:00:00Z","actor":"wbs-generator-agent","action":"generate_wbs","input":"estimate.md@ghi789","output":"wbs.md@jkl012"}
{"ts":"2026-04-26T16:00:00Z","actor":"prototype-agent","action":"generate_prototype","input":"estimate.md@def456","output":"prototype/@mno345"}
```

Lưu ở `anso-docs/05-dev/agent-events.jsonl` (M1 reuse JSONL pattern từ ADR-001).

**Acceptance Criteria:**

- AC-1.8.1: Mọi action có log với actor, input refs, output refs, model, cost.
- AC-1.8.2: Audit log retention ≥ 1 năm.
- AC-1.8.3: UI cho phép trace 1 estimate ngược về brief gốc trong < 3 click.

### 4.9. Deliverables List (F1.9)

**Deliverables Agent flow:**

```
1. Read feature list từ estimate.md → identify output categories
2. Read technical.md → infer storage/delivery channels per artifact type
3. LLM: suggest deliverables list theo 6 nhóm mặc định
4. Map mỗi deliverable: tên công việc, nội dung, chi tiết, lưu/nộp, định dạng
5. Write deliverables.md
```

**Acceptance Criteria:**

- AC-1.9.1: AI auto-suggest deliverables list sau khi estimate + technical spec đã có; presale trigger 1 click.
- AC-1.9.2: Kết quả có ≥ 80% deliverables phù hợp với feature list và tech stack (so với manual review).
- AC-1.9.3: Presale có thể thêm row, xoá row, sửa nội dung inline — không bị overwrite khi re-generate (chỉ suggest dòng mới chưa có).
- AC-1.9.4: Export `.xlsx` qua Export modal: giữ cấu trúc bảng 5 cột, group theo nhóm, header row có định dạng màu sắc nhất quán.
- AC-1.9.5: Visibility Folder push `deliverables.md` khi advance sang M2.

---

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
        │   ├── Attachments panel (files tại 00-attachments/)
        │   └── Ambiguity sidebar (clarify questions)
        │
        ├── Estimate
        │   ├── Feature List section
        │   │   ├── Tree view (expandable nodes)
        │   │   ├── Drag-drop reorder
        │   │   ├── Inline edit complexity
        │   │   ├── Add/cancel feature
        │   │   └── BA Copilot suggestions panel
        │   ├── Summary card (total md, cost, confidence)
        │   ├── Confidence warning banner (nếu < 0.50)
        │   ├── Method comparison (3 columns)
        │   ├── Per-feature breakdown table (incl. Sub content column)
        │   ├── Confidence breakdown (radar chart)
        │   ├── Historical comparison panel
        │   └── Version history sidebar
        │
        ├── WBS
        │   ├── Phase timeline (Gantt-like overview)
        │   ├── WBS table (phase → module → effort)
        │   ├── Effort by role summary
        │   └── Version history sidebar
        │
        ├── Technical
        │   ├── Tech stack table (editable per layer)
        │   ├── Architecture overview (text)
        │   ├── Constraints applied panel
        │   └── Override / pin layer controls
        │
        ├── Prototype
        │   ├── HTML Hi-fi prototype viewer (single file, in-page navigation)
        │   │   └── Sidebar/tabs để navigate giữa các màn hình
        │   └── Replace with upload (Figma export hoặc HTML)
        │   [PNG không hiển thị tại đây — chỉ available qua Export modal]
        │
        ├── Deliverables
        │   ├── Bảng editable 5 cột (Tên công việc / Nội dung giao / Chi tiết / Lưu/Nộp / Định dạng)
        │   ├── Group theo category (Thiết kế & KT / Mobile / Backend / Admin / Tài liệu / QA)
        │   ├── Add row / Delete row / Inline edit
        │   └── Trigger "Suggest deliverables" button (AI auto-fill từ feature list + technical)
        │
        └── Activity Log
            └── Audit trail timeline
```

> **Note về Visibility Folder:** Khi presale click "Next → M2", toàn bộ artifacts được push vào **Visibility Folder** (right panel) dưới dạng `.md` — preview inline. Cạnh mỗi file có **nút Export modal**: user chọn file và format muốn xuất, generate on-demand (tiết kiệm tài nguyên). Sang M2: chỉ `.md` được carry over vào Visibility Folder phase tiếp theo.
>
> | Artifact     | Visibility Folder (.md) | Export modal (on-demand) |
> | ------------ | ----------------------- | ------------------------ |
> | Brief        | `brief.md`              | `Brief_NNN.docx`         |
> | Estimate     | `estimate.md`           | `Estimate_NNN.xlsx`      |
> | WBS          | `wbs.md`                | `WBS_NNN.xlsx`           |
> | Technical    | `technical.md`          | `Technical_NNN.xlsx`     |
> | Prototype    | `prototype.md`          | `prototype.html` + PNG   |
> | Deliverables | `deliverables.md`       | `Deliverables_NNN.xlsx`  |

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
Trong feature list, feature có ambiguity hiển thị badge [!] với tooltip. 1 click → mở Q&A panel để presale clarify với client.

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
POST   /workspaces/{id}/m1/feature-list/extract        # Trigger extraction
GET    /workspaces/{id}/m1/estimate                    # Read estimate (incl. feature list)
PUT    /workspaces/{id}/m1/estimate                    # Update estimate / feature list
GET    /workspaces/{id}/m1/estimate/versions           # List versions
GET    /workspaces/{id}/m1/estimate/{version}          # Read specific version
POST   /workspaces/{id}/m1/estimate/generate           # Trigger estimation
POST   /workspaces/{id}/m1/technical/generate          # Trigger tech spec generation
GET    /workspaces/{id}/m1/technical                   # Read technical spec
PUT    /workspaces/{id}/m1/technical                   # Override tech stack layer
POST   /workspaces/{id}/m1/wbs/generate                # Trigger WBS generation
GET    /workspaces/{id}/m1/wbs                         # Read WBS
POST   /workspaces/{id}/m1/prototype/generate          # Trigger prototype generation
GET    /workspaces/{id}/m1/prototype                   # Read prototype index
PUT    /workspaces/{id}/m1/prototype/{screen_id}       # Replace screen (upload/figma)
POST   /workspaces/{id}/m1/advance                     # Advance M1 → M2: push .md artifacts vào Visibility Folder:
                                                        #   brief.md, estimate.md, wbs.md, technical.md,
                                                        #   prototype.md, deliverables.md
                                                        #   Export format (docx/xlsx/html+png) generate on-demand
                                                        #   khi user trigger Export modal — không auto-generate khi advance
                                                        #   Chỉ .md carry over sang M2; phase change M1→M2

# Project lifecycle (cross-cutting, not module-scoped)
DELETE /projects/{id}                                  # Cancel project.
                                                        #   - 403 nếu user ≠ creator
                                                        #   - Side-effect: workspace VM destroyed; repo archived (30-day grace period)
POST   /projects/{id}/members                          # Add member: body {user_id, role: ba|qa|dev}
DELETE /projects/{id}/members/{user_id}                # Cancel member
```

### 7.2. CLI

Mọi action qua API cũng có CLI tương đương:

```bash
anso m1 brief upload --file RFP.pdf
anso m1 brief upload --file requirements.docx
anso m1 brief upload --file employee-data.csv
anso m1 brief upload --file system-diagram.png
anso m1 feature-list extract
anso m1 estimate generate
anso m1 estimate show
anso m1 estimate compare v1 v2
anso m1 technical generate
anso m1 technical show
anso m1 wbs generate
anso m1 wbs show
anso m1 prototype generate
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
| feature_list_embedding  | vector    | 1536-dim OpenAI ada-002 hoặc tương đương |
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

| Tuần   | Nội dung                                                                  | Acceptance                                      |
| ------ | ------------------------------------------------------------------------- | ----------------------------------------------- |
| **W1** | Sprint 0 dependencies done + M1 scaffold                                  | Workspace VM provision ok, brief.md parse ok     |
| **W2** | Brief ingestion (PDF + DOCX + CSV + image + free text) + UI upload page   | AC-1.1.1, AC-1.1.3 pass                         |
| **W3** | Feature Extraction agent + UI feature list view + edit (in Estimate tab)  | AC-1.2.1 → AC-1.2.5 pass                        |
| **W4** | Estimation Engine — Bottom-up + Three-point + Technical Spec Generator    | AC-1.3.1, AC-1.3.3, AC-1.4.1 pass               |
| **W5** | Analogy method + RAG bootstrap + WBS Generator + Prototype Generator      | AC-1.3.2, AC-1.5.1, AC-1.6.1 pass               |
| **W6** | Visibility Folder + Next→M2 flow + Export via Project Details + Polish    | AC-1.7.1, AC-1.7.2, AC-1.8.x pass               |

**Exit criteria for Milestone 1:**

- ANSO M1 đang được dùng cho ≥ 5 deal thật của công ty.
- Average estimate time: < 8 giờ (vs baseline 3-5 ngày).
- ≥ 80% Sales/Presale team có ít nhất 1 estimate qua M1.
- Confidence score được thực sự sử dụng trong sales conversation (qualitative feedback).
- Documents (prototype + brief + estimate) được gửi cho ≥ 3 client thật.

---

## 9. Test Strategy

### 9.1. Test types

| Type             | Coverage                                                | Tooling           |
| ---------------- | ------------------------------------------------------- | ----------------- |
| Unit test        | Schema validation, scoring formula, RAG retrieval       | Pytest / Jest     |
| Integration test | Brief → feature list → estimate → wbs end-to-end        | Pytest + mock LLM |
| LLM eval         | Feature extraction accuracy trên 20 brief test set      | Ragas / custom    |
| Performance test | 5-page brief → full M1 output < 8 min                   | k6 / Artillery    |
| UI E2E test      | Happy path flow                                         | Playwright        |
| Regression test  | Cùng brief → cùng estimate (within 10%)                 | Snapshot test     |

### 9.2. Acceptance test scenarios

**AT-1: Happy path simple brief**

- Input: 3-page brief (PDF), ERP HR domain.
- Expected: Feature list với ≥ 15 features, estimate trong 5 phút, confidence ≥ 0.65, WBS + prototype generated.

**AT-2: Multi-format brief**

- Input: PDF + DOCX + CSV + image (4 files).
- Expected: All 4 sources reflected trong brief.md, audit trail show từng source, feature list extracted.

**AT-3: Iterative brief change**

- Input: Brief v1 → estimate v1 → modify brief → estimate v2.
- Expected: V2 reflects changes, diff visible, history preserved, WBS re-generated.

**AT-4: Confidence warning**

- Input: Vague brief với nhiều ambiguity.
- Expected: Confidence < 0.50, **warning banner** hiển thị với danh sách ambiguity, presales vẫn có thể proceed.

**AT-5: Visibility Folder & Advance**

- Input: Tất cả artifacts complete.
- Expected: Presale click "Next → M2" → artifacts đẩy vào Visibility Folder đầy đủ → project advance M2.

**AT-6: Cost cap**

- Input: Very large brief (50 pages).
- Expected: Cost stay ≤ $10, hoặc graceful degradation với warning.

---

## 10. Decisions Status & Open Issues

### 10.1. Decisions đã chốt

| #   | Quyết định                                  | Giá trị                                                 | Tham chiếu       |
| --- | ------------------------------------------- | ------------------------------------------------------- | ---------------- |
| D1  | 3 estimation methods chạy song song         | Analogy + Bottom-up + Three-point                       | §3.2.3           |
| D2  | Confidence < 0.50 → warning, không block    | Warning banner + ambiguity list                         | §3.2.4, AC-1.3.5 |
| D3  | Versioning qua git, status `superseded`     | Không xoá file                                          | §3.2.5           |
| D4  | Audit trail dạng JSONL append-only          | Reuse pattern ADR-001                                   | §4.8             |
| D5  | YAML front matter chạy ngầm                 | Không render ra FE; AI internal check only              | §3               |
| D6  | Cross-tenant RAG opt-in                     | Privacy default                                         | §7.3             |
| D7  | M1 ship được dùng độc lập                   | Không cần M2-M7                                         | §1.1             |
| D8  | Không có lock button                        | Chỉ có [Next → M2] sau khi artifacts vào Visibility Folder | §2.3 Step 8   |
| D9  | File export mặc định là `.md`               | Visibility Folder chỉ push `.md`; export format khác (docx/xlsx/html+png) generate on-demand qua Export modal trong Visibility Folder | §3, §6.1    |
| D10 | WBS ở M1 là high-level phase breakdown      | Không chi tiết task-level (đó là việc M2)               | §3.3             |
| D11 | AI auto-fill YAML từ brief, presales review | Không để trống để fill tay                              | §3.1.2           |
| D12 | Feature Tree merge vào Estimate             | Không còn tab/file riêng; feature list embedded trong estimate.md | §3.2  |
| D13 | Prototype output: 1 HTML single-file Hi-fi  | 1 file prototype.html, multi-screen, basic in-page routing, Hi-fi fidelity; PNG generate ngầm cho export, không hiển thị trong workspace | §3.5.2 |
| D14 | Visibility Folder model                     | Visibility Folder hiển thị `.md` preview inline; Export modal per artifact generate on-demand (Brief→docx, Estimate→xlsx, WBS→xlsx, Technical→xlsx, Prototype→html+PNG, Deliverables→xlsx); sang M2 chỉ `.md` carry over | §2.3 Step 8 |
| D15 | Thêm tab Deliverables vào workspace M1      | Workspace order: Brief → Estimate → WBS → Technical → Prototype → Deliverables; Deliverables artifact = deliverables.md, export xlsx qua Export modal | §3.6, §6.1 |

### 10.2. [!] Decisions còn pending

| #   | Quyết định                         | Phương án đề xuất                                                     | Khi nào quyết định      |
| --- | ---------------------------------- | --------------------------------------------------------------------- | ----------------------- |
| P1  | Method weights default             | 0.4/0.4/0.2 phase đầu, calibrate sau                                  | Sau 1 tháng có data     |
| P2  | LLM model cho từng task            | Claude Opus cho extract+estimate, Haiku cho minor                     | Cost benchmark tuần 1-2 |
| P3  | Bootstrap data lấy từ đâu          | Synthesize từ 20-30 BA/Sales senior interview hay xin từ project thực | Tuần 1                  |
| P4  | Cost cap default per full M1 run   | $10 hard cap (proposed)                                               | Cần input từ finance    |
| P5  | Documents delivery channel         | Visibility Folder (.md preview) + Export modal per artifact (on-demand)  | **[Chốt tại v5 — xem D14]** |

### 10.3. Future considerations

1. **Email-to-brief workflow** — forward email vào dedicated address → auto create project + brief.
2. **CRM integration** — sync với HubSpot / Salesforce để pull deal info.
3. **Multi-currency support** — documents pricing per region.
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

| Service    | Mục đích                       | Fallback          |
| ---------- | ------------------------------ | ----------------- |
| Claude API | Primary LLM                    | OpenAI GPT-4      |
| OpenAI API | Embedding + fallback LLM       | Local model       |
| AWS S3     | Attachments storage            | MinIO self-host   |
| GitHub API | Repo mgmt, version history     | GitLab self-host  |

---

## 12. Trigger để re-evaluate Module

M1 nên được re-evaluate (PRD revision) nếu:

1. **Estimate accuracy < 70%** sau 3 tháng — model/method không work, cần redesign.
2. **Adoption rate < 50%** trong presale team sau 2 tháng — UX/value proposition issue.
3. **Cost per full M1 run > $5 average** — không sustainable, cần optimize hoặc đổi model.
4. **Confidence score không correlate với accuracy thực tế** — formula sai.
5. **Stakeholder feedback negative** từ phase B pilot — không phù hợp với customer outsourcing khác.
6. **Industry shift** lớn (vd LLM ecosystem thay đổi căn bản) — cần redesign.
7. **Documents không được client chấp nhận** sau 3 tháng — cần revisit format/content.

---

## 13. Tài liệu tham khảo

- ANSO PRD v1.3, §6.1 (M1 spec gốc), §10 Milestone 1.
- ADR-001: Markdown-First Storage — front matter pattern.
- ADR-002: Workspace-VM Pattern — runtime cho extraction agent.
- *Software Estimation: Demystifying the Black Art* — Steve McConnell (PERT, three-point).
- *Agile Estimating and Planning* — Mike Cohn (story points, planning poker).
- COCOMO II — historical estimation model.
- "Function Point Analysis" — IFPUG.
- Sample outcomes: `Phoenix_TiengAnh_V1.xlsx` (Estimate), `CRM_Estimation_v2.xlsx` (WBS), `edu-safecam-wifi.xlsx` (Technical).

---

## 14. Phụ lục

### 14.1. Glossary

- **Brief** — tổng hợp yêu cầu khách hàng, source of truth cho M1.
- **Feature List** — danh sách tính năng có cấu trúc và complexity, embedded trong Estimate.
- **Technical Spec** — tech stack + architecture overview, AI auto-generate từ brief constraints.
- **Analogy method** — estimation dựa similarity với historical projects (RAG).
- **Bottom-up method** — estimation per-feature × productivity.
- **Three-point / PERT** — (Optimistic + 4×Realistic + Pessimistic) / 6.
- **Confidence Score** — 0-1, đo độ tin cậy estimate.
- **Implicit Buffer** — pessimistic_md - realistic_md = contingency từ three-point.
- **WBS** — Work Breakdown Structure, high-level phase breakdown cho client.
- **Prototype** — wireframe HTML AI-generated từ feature list; output HTML + PNG.
- **Visibility Folder** — right panel hiển thị toàn bộ artifact `.md` sau khi presale finish luồng M1, dạng preview inline. Cạnh mỗi file có **nút Export modal**: user chọn artifact và format muốn xuất (Brief→docx, Estimate→xlsx, WBS→xlsx, Technical→xlsx, Prototype→html+PNG, Deliverables→xlsx), generate on-demand — không auto-generate toàn bộ khi advance (tiết kiệm tài nguyên). Sang M2: chỉ `.md` carry over vào Visibility Folder phase tiếp theo.

### 14.2. Sample brief → documents transformation

(Xem ví dụ end-to-end trong `anso-docs/01-presale/` của project ACME ERP HR — tham chiếu chéo từ các schema trong §3. Lưu ý: attachment files nằm tại `00-attachments/`.)

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
