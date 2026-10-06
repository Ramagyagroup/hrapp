# RAIL-ITIS — Detailed Build Plan

**Status:** Draft v0.1, 06 Oct 2026. The stack, theme and code patterns are confirmed only after the PMT-APP code review (Phase 0).
**Sources:** RAIL-ITIS Documents 1–17, Developer Build Pack (Modules 1–28), Execution Documents 1–8, Master Development Prompt & Handover Protocol, WORKING note, the approved UI mockup `mockups/RAIL-ITIS-Mockup.html`, and the PMT-APP audit (Express + Next.js, RBAC, audit logs, export module, ⌘K search).

---

## Guiding rules (from the Master Development Prompt §87–89)

- One platform, not 28 apps. Shared masters, no duplicates.
- Foundation first: auth, RBAC, masters, workflow, approval, audit, documents, notifications, events. Never start with the dashboard.
- No hard-coded approvals, verticals or thresholds. They are business configuration.
- Current FMS / sheet data is **not** verified truth. It goes through reconciliation.
- History never disappears: soft delete, immutable audit trail, custody changes only through approved movements.
- AI comes after the data foundation, and only as recommendations. AI never edits core records.
- Every release passes a gate: functional, data accuracy, security, integration, UAT (including negative cases), reconciliation, audit trail.
- Reuse PMT-APP's proven patterns (theme, shell, RBAC, audit, export, search, docs) instead of reinventing them.

---

## Phase 0 — Learn PMT-APP and lock the engineering baseline (week 1)

| Step | Output |
|---|---|
| 0.1 Read the whole PMT-APP codebase | Stack inventory: framework versions, ORM, DB, auth, job runner, mailer, file storage, deploy scripts |
| 0.2 Extract the UI and theme rules | Tokens (colours, type, spacing), app shell (sidebar, topbar, phone app bar, bottom tabs), components (tables → cards on phone, dialogs/sheets, forms, KPI tiles, badges), icons (lucide), date/money helpers, wording rules |
| 0.3 Extract the governance patterns | RBAC model (roles, permissions, per-user grants), audit log / activity log / request log, export jobs, ⌘K search, in-app docs, error handling |
| 0.4 Extract the checks | `ramagya-design-system` skill: compliance scanner (strict), 390 px phone audit, parity tools, review checklist |
| 0.5 Write the baseline | `docs/ENGINEERING-BASELINE.md` (what we copy as-is, what we adapt, what PMT-APP lacks and ITIS needs) and `CLAUDE.md` for the repo |
| 0.6 Scaffold the RAIL-ITIS repo | Same monorepo layout as PMT-APP, the design-system skill committed under `.claude/skills/`, lint/format/typecheck/test scripts, CI |

**Gate 0:** you approve the baseline doc and the empty app shell renders at 390 px and 1280 px with the RAIL look.

---

## Phase 1 — Document ingestion (the handover protocol's starting sequence, steps 1–10) (week 1–2)

1. **Documentation index:** every document, section and module mapped.
2. **Requirement Traceability Matrix:** one ID per requirement (`ITIS-<MOD>-<NNN>`), with source doc/section, module, release, priority, test case ID, status.
3. **Registers:** Contradiction, Gap, Assumption, Development Decision. Anything ambiguous goes here with a proposed resolution for you to approve. No silent decisions.
4. **Validation:** architecture, ERD (Execution Doc 2 entities), workflows, rules/scores (Execution Doc 5), APIs/events, roles/rights (Document 6).

**Gate 1:** you sign off the RTM and registers. Open contradictions are closed or explicitly deferred.

---

## Phase 2 — Platform foundation (weeks 3–5)

Built once, used by every module (Execution Doc 1 §33):

| Area | Scope |
|---|---|
| Identity | Login (SSO if available), session, lockout, user status synced from RAIL-HRIS (inactive → access off) |
| RBAC + ABAC | Role families from Document 6, permission catalogue, per-user grants, scope by vertical / location / department, role stacking, segregation-of-duties checks |
| Master data | Vertical, location, department, employee (from HRIS), asset category / model / classification, vendor, status masters, custom attributes |
| Workflow + approval engine | Configurable approval matrix and limits, maker ≠ checker ≠ approver, chain snapshot, delegation, expiry, no blind approval |
| Rules & score engine | Versioned, configurable thresholds and formulas (health score, repair/replace, audit risk…) |
| Audit trail | Immutable, before/after values, actor, reason, IP; no delete for any role |
| Documents | File store with checksums, access control, linked to any record |
| Notifications | Email first (provider TBD), templates, escalation timers, job scheduler |
| Events + integration log | Domain events (asset.created, custody.changed…), outbound/inbound log, retries |
| Search, export, logging | ⌘K search with permission filtering, governed export jobs, structured logs, error handling |
| Universal record page | Header, status, timeline, documents, audit tab, used by every entity (Master Prompt §18) |

**Gate 2:** security test matrix passes for the foundation, and a demo shows permission denial, maker-checker block, and audit trail entries.

---

## Phase 3 — Releases (Execution Doc 6 §63)

Each release: sprint plan → build → automated tests → UAT (happy + negative + permission + maker-checker) → reconciliation → release gate → pilot location → next.

| Release | Modules | Key outcomes | Mockup screens |
|---|---|---|---|
| **R1 — Asset Truth** | 2 Asset Master & Digital Passport, 3 QR/Barcode, 4 Inventory & Stock, 9 Allocation & Custody | One asset → one identity. QR passport. Custody only via acknowledged movement. Parent-child components. | 05, 09, 10, 11, 12 |
| **R2 — Employee IT Lifecycle** | 10 Employee IT Lifecycle + RAIL-HRIS integration | Joining entitlement, transfer review, exit clearance with config verification, inactive-with-asset exception | 18 |
| **R3 — Service & Audit** | 11 Service Desk, 12 Preventive Health Check, 13 Physical Verification & Audit, **Endpoint Health (manual + agent pilot)** | SLA + escalation, risk-based check frequency, employee acknowledgement, scan-and-verify, exception register | 13, 14, 15, 24–35 |
| **R4 — Lifecycle Economics** | 14 Repair, 15 Upgrade & Component, 16 Depreciation, 17 Warranty & AMC | Repair history and cost, component swaps, SLM book value, warranty leakage check | 16, 19 |
| **R5 — Budget & Procurement** | 5 Budget, 6 Purchase, 7 Vendor & RFQ, 8 Inward & QC (integrates with Purchase Intelligence) | Budget availability check, stock-before-purchase, quote comparison, GRN → asset creation | 06, 07, 08, 09 |
| **R6 — Replacement & Disposal** | 18 Replacement Intelligence, 19 Dead / Obsolete / Disposal | Economic health score, exceptional repair approval, cannibalisation, data destruction certificate, write-off | 17, 21 |
| **R7 — Software, Telecom, Infrastructure** | 20 Software & License, 21 Network, 22 Telecom & SIM | Seats vs assigned, renewals, ISP lines, SIMs of exited users | 20 |
| **R8 — Intelligence & Control Tower** | 1 Command Centre, 23 Vendor Intelligence, 24 AI & Predictive, 25 MIS & Promoter Control Tower | Drill-down Group → Vertical → Location → Department → Employee → Asset; AI recommendations with confidence and feedback | 04, 23, 24 |
| Cross-cutting (every release) | 26 Notification & Escalation, 27 Governance & Audit Trail, 28 Integration | Grown with each release, not built separately | 22 |

### Endpoint Health track (inside R3, extended in R8)

| Step | Scope |
|---|---|
| EH-1 | Policy model: roles → policies → requirements (operator, value, unit, severity, enabled), versioned; test-definition catalogue; scoring weights per policy |
| EH-2 | Server: job planner (only allowlisted capabilities), signed jobs, result schema validation, policy evaluation → PASS/WARNING/FAIL, device health score, role compatibility score, issue de-duplication by key, auto-resolve |
| EH-3 | **Windows ITIS Agent** (separate repo): Windows service, read-only collectors (CPU, memory, GPU, disk, battery, display, software inventory, security status, bounded performance tests, dev tools), mTLS, heartbeat, signed self-update, **no remote shell** |
| EH-4 | Monthly flow: due → email link → secure page → agent check → results → issues/tickets → employee + IT notified |
| EH-5 | UI: screens 24–35 of the mockup, the Endpoint Health section in the Digital Passport |
| EH-6 | Pilot on 20–30 devices (one department), then rollout in rings |

---

## Phase 4 — Migration & reconciliation (runs alongside R1–R2)

Legacy register + invoices/POs + issue records + HRIS employee master + physical verification → **opening verified asset register**, with each asset classified as Verified / Partially Verified / Data Incomplete / Not Located / Dead-pending-disposal. Import preview, error report, and reconciliation sign-off before cutover (Master Prompt §46).

## Phase 5 — Cutover, hypercare, 30/60/90-day reviews

Pilot location first, then location-by-location rollout, hypercare with daily defect triage, and 30/60/90-day reviews against the RTM.

---

## Quality bar for every screen and PR

- Theme: `ui-compliance-scan --strict` = 0 blocking; phone audit at 390 px = 0 failures; screenshots at 390 px and 1280 px.
- Tests: unit + API + permission + maker-checker + negative cases from Execution Doc 7; golden scenarios (Master Prompt §25).
- Every requirement implemented is ticked in the RTM with its test ID.
- No business logic change without an entry in the Decision Register.

## Immediate next steps (after your inputs)

1. Phase 0 code review of PMT-APP → `ENGINEERING-BASELINE.md` for your approval.
2. Phase 1 RTM + registers → your sign-off.
3. Foundation sprint plan with estimates.
