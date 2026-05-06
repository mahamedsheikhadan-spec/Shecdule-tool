# Smart Labor Scheduling Platform
## Product Requirements Document (PRD)

**Document version:** 1.0
**Date:** May 2026
**Status:** Draft for internal review
**Owner:** Mahamed (Product / Operations)
**Audience:** Internal engineering, operations, and leadership

---

## 1. Executive Summary

This document defines the product requirements for an internal multi-location labor scheduling platform built for a restaurant and/or retail business. The system gives managers the tools to build cost-efficient, compliant schedules driven by projected hourly sales, labor budget targets, employee availability, time-off requests, maximum weekly hours, and trained positions. Employees get a separate mobile-friendly portal to view schedules, manage availability, request time off, and trade shifts.

The primary outcome is to **reduce time spent building schedules** while keeping **labor cost as a percentage of sales within target**, without violating availability, compliance, or training rules. The secondary outcome is to give employees more transparency and self-service control, which has a measurable effect on retention.

---

## 2. Goals, Non-Goals, and Success Metrics

### 2.1 Goals
- Cut weekly schedule-creation time by **at least 60%** versus spreadsheet-based scheduling.
- Hold actual labor as a percentage of sales within **±1.0%** of the budgeted target on a per-location, per-week basis.
- Eliminate scheduling errors related to availability conflicts, time-off overlap, max-hour violations, and untrained position assignments — target **zero** of these per published schedule.
- Reduce no-shows and last-minute call-outs by giving employees an easy way to drop, swap, and pick up shifts.
- Provide a single source of truth for schedules across all locations.

### 2.2 Non-Goals (for v1)
- Full payroll processing (we will export hours; we will not cut checks).
- Tip pooling or tip-out calculations.
- Applicant tracking / hiring workflows.
- Inventory or POS replacement (we **integrate** with POS for sales; we do not replace it).
- Performance reviews, training delivery, or LMS functionality.

### 2.3 Success Metrics (KPIs)
| Metric | Baseline | Target (90 days post-launch) |
|---|---|---|
| Avg. minutes to build a weekly schedule per location | 120 min | < 30 min |
| Locations within ±1% labor target | ~30% | 80% |
| % of shifts filled via self-service swap/pickup | 0% | 50% |
| Manager weekly active usage | n/a | 100% |
| Employee weekly active usage | n/a | ≥ 85% |
| Schedule rework after publish (edits in 48 hr) | unknown | < 10% of shifts |

---

## 3. User Roles and Permissions

The system uses **role-based access control (RBAC)** with role inheritance. Permissions are scoped by location; a user can have different roles at different locations.

### 3.1 Roles

**Owner / Corporate Admin**
- Full access across all locations.
- Manages locations, role definitions, labor targets, integrations, and billing.
- Can impersonate any other role for support.

**General Manager (per location)**
- Full access to their assigned location(s).
- Manages employees, schedules, availability, time-off, sales projections, and labor budgets for that location.
- Can publish schedules.

**Shift Manager / Assistant Manager**
- Same as General Manager **except**: cannot deactivate employees, cannot change labor budget targets, cannot edit position pay rates.
- Can publish schedules only with approval setting toggled on by GM.

**Employee**
- Views own schedule and (if enabled) the team schedule.
- Submits availability, time-off, drop, pickup, and swap requests.
- Receives notifications.

**Read-only Viewer (e.g., HR, Finance)**
- Can view schedules, labor reports, and rosters across permitted locations.
- Cannot edit anything.

### 3.2 Permission Matrix (abridged)

| Capability | Owner | GM | Shift Mgr | Employee | Viewer |
|---|---|---|---|---|---|
| Create / deactivate employees | ✅ | ✅ | ❌ | ❌ | ❌ |
| Edit position assignments & max hours | ✅ | ✅ | ❌ | ❌ | ❌ |
| Edit labor budget targets | ✅ | ✅ | ❌ | ❌ | ❌ |
| Generate suggested schedule | ✅ | ✅ | ✅ | ❌ | ❌ |
| Edit draft schedule | ✅ | ✅ | ✅ | ❌ | ❌ |
| Publish schedule | ✅ | ✅ | conditional | ❌ | ❌ |
| Approve time-off | ✅ | ✅ | ✅ | ❌ | ❌ |
| Approve drops/pickups/swaps | ✅ | ✅ | ✅ | ❌ | ❌ |
| Submit time-off / availability change | ✅ | ✅ | ✅ | ✅ | ❌ |
| View own schedule | ✅ | ✅ | ✅ | ✅ | ✅ |
| View team schedule | ✅ | ✅ | ✅ | toggle | ✅ |
| View labor reports | ✅ | ✅ | ✅ (own location) | ❌ | ✅ |

### 3.3 Authentication & Account Model
- Single account per person; email + password with optional magic-link login for employees.
- SSO (Google / Microsoft) for corporate users.
- Phone-number-based login is optional for hourly employees who don't check email.
- 2FA required for Owner and GM roles.
- All sessions are device-aware; admins can revoke sessions.

---

## 4. Core Features

This section catalogs every feature in the system, organized by domain. Detailed workflows live in sections 5 and 6.

### 4.1 Employee management
- Employee profile (name, contact, photo, hire date, employment type, pay rate, emergency contact).
- Trained positions / role tags per employee, with optional certification expiry (e.g., food handler card).
- Maximum weekly hours, maximum daily hours, and minimum guaranteed hours per employee.
- Employment status: active, on leave, terminated.
- Pay rate history (for accurate retroactive labor cost).

### 4.2 Availability management
- Recurring weekly availability template per employee.
- Date-range overrides (e.g., "Tuesdays unavailable from June 1 – Aug 15 due to summer class").
- Availability change requests with approval workflow.
- Effective-date controls so changes don't disrupt already-published schedules.

### 4.3 Time-off management
- Employee submits PTO/UPTO/sick request with date range, hours, and reason.
- Manager approves, denies, or requests modification.
- Once approved, the engine treats the dates as hard unavailability.
- Visibility of remaining PTO balance (if location tracks balances).

### 4.4 Sales projections and labor budget
- Hourly sales forecast per location per day.
- Three input modes: (a) manual entry, (b) CSV import, (c) auto-pulled from POS integration.
- Labor budget rules: target labor % of sales overall, plus per-role caps if needed (e.g., BOH ≤ 12%, FOH ≤ 14%).
- Optional fixed-cost lines (salaried managers) excluded from variable labor budget.
- Comparison view: prior-week actual vs. current projection vs. same-week last year.

### 4.5 Position / role definitions
- Configurable positions per location (e.g., Cashier, Line Cook, Server, Supervisor, Floor Lead, Dishwasher).
- Per-position required staffing curves: minimum bodies needed at each hour given the sales projection band.
- Position pay rate defaults (overridden by employee pay rate).

### 4.6 Schedule generation engine
- "Suggest schedule" button generates a draft for the selected week and location.
- Engine output is fully editable; nothing is final until **Publish**.
- Real-time labor cost and labor % displayed as the manager edits.
- Visual conflict warnings (availability, time-off, double-booking, missing certification, OT risk, untrained for position).

### 4.7 Manual schedule editing
- Drag-and-drop shift placement on a grid (employee × hour).
- Shift templates (e.g., "11–4 line cook") to speed manual entry.
- Bulk actions: copy last week, copy from template, clear day, swap two employees, shift entire day by N hours.
- Open shifts (unassigned) supported and visible to qualified employees once published.

### 4.8 Schedule publishing
- Publish action generates notifications and locks the schedule into a "published" state.
- Re-publishing surfaces a diff so employees see exactly what changed.
- Compliance check on publish (predictive scheduling laws where applicable; configurable lead time, e.g., must publish 14 days out).

### 4.9 Shift trades: drops, pickups, swaps
- Drop: employee requests to give up a shift; manager approves; shift becomes "open" and offered to qualified employees.
- Pickup: employee claims an open shift; manager approves (auto-approve toggle available).
- Swap: two employees agree to trade specific shifts; manager approves.
- All three respect rules engine (availability, max hours, training, OT).

### 4.10 Notifications
- Channels: in-app, push (mobile web / PWA), email, SMS.
- Triggers: schedule published, schedule changed, request status change, shift reminder (configurable lead time), open shift available for qualified employees.
- Per-user notification preferences.

### 4.11 Reporting
- Live dashboard: current-week scheduled hours, projected labor cost, projected labor %.
- Variance report: scheduled vs. actual (when timekeeping data is connected).
- Per-employee weekly hours report (with OT flag).
- Time-off and availability change history.
- Audit log of every schedule change with actor, timestamp, before/after.

### 4.12 Integrations (v1 scope)
- POS sales import (Toast, Square, Clover via API or scheduled CSV).
- Calendar export per employee (iCal feed).
- CSV export of schedules and hours for payroll.
- Webhook endpoints for downstream systems.

---

## 5. Manager / Admin Workflow

### 5.1 First-time setup (per location)
1. Create location, set time zone, week start day, and overtime rules.
2. Define positions and pay rate defaults.
3. Configure labor budget rules (target %, per-role caps, fixed costs).
4. Add employees: profile, positions, max hours, base availability, pay rate.
5. Connect POS integration (or import a CSV of last 8 weeks of hourly sales for forecasting).
6. Invite employees via email/SMS with a one-tap onboarding link.

### 5.2 Weekly scheduling cycle (the core flow)
The target is to complete the full cycle in **under 30 minutes per location per week**.

1. **Open the next-week planner.** Manager sees: prior-week actuals, current sales forecast, applicable PTO and approved availability, and any open requests requiring attention.
2. **Review and adjust the sales forecast** for special events, weather, or promotions.
3. **Confirm staffing requirements per role per hour.** Defaults come from the demand model; manager can override.
4. **Click "Generate Schedule."** The engine produces a draft optimized against the rules in section 7.
5. **Review the draft.** Color-coded warnings flag any issue. Manager drag-and-drops, splits shifts, or marks shifts as "open."
6. **Watch the live labor % readout.** Edits update the forecasted labor cost and percentage in real time.
7. **Resolve all blocking conflicts** (hard violations cannot be published).
8. **Click "Publish."** The system sends notifications, locks the schedule, and writes an audit entry.

### 5.3 Mid-week management
- Approve/deny incoming time-off, availability change, drop, pickup, and swap requests from a single inbox.
- Edit published schedule when needed; affected employees are notified of the diff.
- Monitor day-of dashboard: who's clocked in, projected vs. actual labor for the day, open shifts still uncovered.

### 5.4 End-of-week / payroll handoff
- Review the variance report (scheduled vs. actual hours and cost).
- Reconcile any open exceptions.
- Export hours to payroll (CSV or direct integration).

---

## 6. Employee Workflow

The employee experience is designed mobile-first. The target is "everything an employee needs in three taps."

### 6.1 Onboarding
1. Receive invite link via SMS or email.
2. Set password, confirm contact info, upload profile photo.
3. Set initial recurring weekly availability.
4. Enable push notifications.

### 6.2 Day-to-day
- **Home screen** shows: next shift, this week's scheduled hours, any pending requests, any open shifts the employee qualifies for.
- View full personal schedule for the week or month.
- View team schedule (if enabled by management).
- Submit a time-off request (date range, hours, reason).
- Update availability (with effective date so it doesn't break the current week).
- Drop a shift: select shift, optionally add a note, submit. Status moves to "Pending."
- Pick up a shift: tap an open shift in the marketplace, submit. Status moves to "Pending."
- Swap a shift: pick which of your shifts you want to trade and which coworker's shift you want; both parties confirm; manager approves.

### 6.3 Notifications and transparency
- Push notification when a schedule is published or when a request changes status.
- Reminder before each shift (configurable, default 2 hours prior).
- Every request shows clear status: Pending, Approved, Denied (with reason if provided).

---

## 7. Scheduling Rules Engine

The rules engine is the core differentiator of this product. It is split into **hard rules** (cannot be violated) and **soft rules** (penalties in the optimizer, can be relaxed).

### 7.1 Hard rules
1. Employee must be marked **available** for every hour of the assigned shift.
2. Employee must not have **approved time off** overlapping the shift.
3. Employee must be **trained** in the position they're assigned to.
4. Employee must not exceed **maximum daily hours**.
5. Employee must not exceed **maximum weekly hours** (jurisdiction-aware: legal vs. preference vs. visa cap).
6. Two shifts for the same employee must respect the **minimum rest interval** (e.g., 8 hours between shifts).
7. Required staffing minimums per position per hour must be met.
8. Minor labor laws: under-18 employees follow jurisdiction rules (school night cutoffs, daily/weekly caps, prohibited tasks).

### 7.2 Soft rules (weighted optimization objectives)
These trade off against each other; weights are tunable per location.

- Minimize total labor cost (largest weight by default).
- Stay within target labor % of projected sales.
- Honor employee-preferred hours (each employee can express preferred min/max).
- Honor preferred shift patterns (some employees prefer mornings, some closes).
- Minimize fragmentation (avoid 3-hour shifts unless preferred).
- Distribute desirable shifts fairly across employees.
- Prefer continuity (give the same employee opening shifts across the week).
- Avoid back-to-back close-then-open ("clopen") unless explicitly allowed.
- Reward seniority on shift preference where two employees are equally qualified.

### 7.3 Demand model
For each hour of each day, the engine computes required headcount per position from:
- Hourly sales forecast.
- Sales-per-labor-hour benchmarks per role (configurable; e.g., "1 cashier per $250/hr in sales").
- Minimum staffing floor (e.g., never fewer than 2 people on the floor).
- Service standards (e.g., no more than 4 active tables per server).

### 7.4 Optimization approach
- Frame as a constrained optimization problem: variables are (employee, position, hour) assignments.
- For MVP: a greedy heuristic with local-search improvement is sufficient (build a feasible schedule, then iterate to reduce cost while staying feasible).
- For v2: replace heuristic with a Mixed-Integer Program (MIP) using OR-Tools CP-SAT for true optimality on schedules up to a few hundred employees per location.
- Always return a feasible draft within 5 seconds for typical 30-employee locations.

### 7.5 Conflict reporting
Every generated schedule includes a structured report:
- Hard violations (block publish).
- Soft-rule penalty score and breakdown.
- Coverage gaps by hour and position.
- Forecasted vs. budgeted labor delta.

---

## 8. Database Structure

The data model is normalized and multi-tenant from day one. Every business-level table carries a `location_id` (or is joined to one) so multi-location is native, not bolted on.

### 8.1 Core tables

**organizations**
- id, name, billing_email, plan_tier, created_at

**locations**
- id, organization_id, name, address, time_zone, week_start_day (0–6), overtime_rule_id, created_at

**users**
- id, organization_id, email, phone, password_hash, full_name, photo_url, status (active/inactive), created_at

**user_locations** (many-to-many)
- user_id, location_id, role (owner / gm / shift_mgr / employee / viewer)

**employee_profiles**
- user_id (PK, FK to users), hire_date, employment_type (FT/PT/temp), max_weekly_hours, max_daily_hours, min_weekly_hours, notes

**pay_rates**
- id, user_id, location_id, position_id (nullable for general rate), rate_cents, effective_from, effective_to

### 8.2 Position / training tables

**positions**
- id, location_id, name, default_pay_rate_cents, color, sort_order

**employee_positions** (many-to-many; trained-for tracking)
- user_id, position_id, certified_at, expires_at (nullable), is_primary

### 8.3 Availability and time-off

**availability_rules**
- id, user_id, location_id, day_of_week (0–6), start_time, end_time, available (bool), effective_from, effective_to

**availability_change_requests**
- id, user_id, proposed_rules (JSONB), status (pending/approved/denied), submitted_at, decided_at, decided_by, manager_note

**time_off_requests**
- id, user_id, location_id, start_at, end_at, hours, reason, status (pending/approved/denied), submitted_at, decided_at, decided_by, manager_note

### 8.4 Sales and budget

**sales_forecasts**
- id, location_id, date, hour, projected_sales_cents, source (manual/import/pos), updated_at

**sales_actuals**
- id, location_id, date, hour, actual_sales_cents, source

**labor_budgets**
- id, location_id, effective_from, target_labor_pct, per_role_caps (JSONB), fixed_cost_cents

### 8.5 Schedules and shifts

**schedules**
- id, location_id, week_start_date, status (draft/published/archived), version, published_at, published_by, generation_metadata (JSONB)

**shifts**
- id, schedule_id, user_id (nullable for "open"), position_id, start_at, end_at, break_minutes, notes, status (scheduled/canceled), created_at, updated_at

**shift_changes** (audit log)
- id, shift_id, actor_user_id, change_type, before_state (JSONB), after_state (JSONB), changed_at

### 8.6 Trades

**shift_trade_requests**
- id, type (drop / pickup / swap), shift_id, counterparty_shift_id (nullable, swap only), requester_user_id, target_user_id (nullable, pickup), status (pending/approved/denied/expired), submitted_at, decided_at, decided_by

### 8.7 Notifications and audit

**notifications**
- id, user_id, type, title, body, payload (JSONB), channel, sent_at, read_at

**audit_log**
- id, organization_id, location_id, actor_user_id, entity_type, entity_id, action, diff (JSONB), at

### 8.8 Indexing and integrity notes
- Composite index on `shifts(location_id, start_at)` for daily/weekly queries.
- Composite index on `availability_rules(user_id, day_of_week)`.
- Foreign keys with `ON DELETE RESTRICT` on anything tied to a published schedule.
- Use a check constraint to ensure `shifts.end_at > shifts.start_at`.
- Soft delete (status flag) preferred over hard delete for any record that may appear in historical reports.
- Row-level security policies in Postgres ensure users see only their organization/location data.

---

## 9. Recommended Tech Stack

These choices optimize for **fast iteration, multi-location scale, low ops burden, and a strong mobile employee experience without shipping native apps in v1**.

### 9.1 Frontend
- **Next.js (App Router) + React + TypeScript** for both manager and employee portals as a single PWA codebase.
- **Tailwind CSS** with **shadcn/ui** for a consistent design system and fast UI work.
- **Installable PWA** with offline-aware caching for the employee portal so schedules are viewable on a spotty connection.
- **TanStack Query** for server-state caching; **Zustand** for local UI state.

### 9.2 Backend
- **Node.js (TypeScript) with Fastify or NestJS** as the API layer.
- **PostgreSQL** as the primary database (RLS, JSONB, time-zone handling are all first-class).
- **Prisma** as the ORM (schema-first, good migrations).
- **Redis** for session storage, rate limiting, queue backbone, and notification fan-out.
- **BullMQ** for background jobs (notification sending, schedule generation, integrations).
- **OR-Tools (Python service)** invoked over a small internal RPC for the MIP solver in v2; greedy heuristic in TypeScript for v1.

### 9.3 Auth
- **NextAuth / Auth.js** for browser sessions plus magic-link.
- **Twilio Verify** for phone OTP login (used by hourly employees).
- **WorkOS** for enterprise SSO when corporate accounts need it later.

### 9.4 Integrations
- **Toast / Square / Clover SDKs** for POS sales pulls.
- **Twilio** for SMS.
- **SendGrid or Resend** for email.
- **Web Push (VAPID)** for browser/PWA push notifications.

### 9.5 Hosting and ops
- **Vercel** for the Next.js app, **Render or Fly.io** for the API and worker services.
- **Neon or Supabase** for managed Postgres with branching for preview environments.
- **S3-compatible object storage** for profile photos and exports.
- **Sentry** for error monitoring, **PostHog** for product analytics, **Grafana Cloud** for infra metrics and logs.

### 9.6 Why not native mobile apps in v1?
A well-built PWA installed to the home screen covers ~95% of employee use cases (view schedule, swap, request time off, push notifications) without app store approvals, cold-start friction, or two extra codebases. We can revisit native apps in v3 if push reliability or biometric login become blockers.

---

## 10. MVP Version

The MVP is the smallest scope that lets a real location run a real week's schedule end-to-end. Everything outside MVP is deliberately deferred.

### 10.1 In scope for MVP
- Multi-location data model (even if launch starts with one location).
- Owner, GM, and Employee roles. Shift Manager and Viewer deferred.
- Employee management: add, edit, deactivate, assign positions, set max hours, set base availability.
- Time-off requests: submit, approve, deny.
- Sales projections: manual entry and CSV import (POS integration deferred).
- Labor budget: single target labor % per location.
- Required staffing per role per hour, set manually by manager (auto-suggested in v2).
- Schedule generation: greedy heuristic respecting all hard rules and the top three soft rules (cost, fairness, fragmentation).
- Manual drag-and-drop schedule editing with live labor % display.
- Conflict warnings on draft schedules.
- Publish schedule, lock it, and notify employees by email + push.
- Employee portal: view schedule, submit/track time-off, set availability, drop a shift, pick up an open shift.
- Manager request inbox (single screen for all approvals).
- Basic reporting: weekly scheduled hours, labor cost, labor %.
- CSV export of hours.

### 10.2 Explicitly out of MVP
- Shift swap (only drop + pickup in MVP).
- POS integration (manual + CSV only).
- SMS notifications (push + email only).
- Predictive scheduling compliance checks.
- Audit log UI (data is recorded; UI deferred).
- Multi-location switching UI polish (functional but minimal).
- AI features (see section 12).

### 10.3 Definition of done for MVP
- A new GM can onboard a location and 25 employees in under 60 minutes.
- A weekly schedule can be generated, edited, and published in under 30 minutes.
- Employees receive notifications and can self-serve drops/pickups.
- Zero hard-rule violations on any published schedule across two weeks of beta.
- 10 consecutive published schedules at the pilot location with manager satisfaction ≥ 4/5.

---

## 11. Future / Advanced Features (Post-MVP)

Grouped by the version they would land in.

### v2 (3–4 months after MVP)
- Shift swap workflow with two-party confirmation.
- POS integrations (Toast, Square, Clover).
- True MIP optimizer via OR-Tools.
- Auto-suggested staffing curves derived from historical sales-per-labor-hour.
- SMS notifications and reminders.
- Predictive scheduling / fair workweek compliance for applicable jurisdictions.
- Per-role labor caps in budget rules.
- Audit log UI and full role-based permission matrix.
- Multi-location dashboard for Owners (rolled-up labor %, exceptions, attention queue).
- Calendar feed (iCal) per employee.

### v3 (6–9 months after MVP)
- Native iOS and Android apps (if research justifies it).
- Time clock with geofencing, replacing or complementing POS clock-ins.
- Tip pool and tip-out helper.
- Employee messaging (1:1 and per-location announcements).
- Open shifts marketplace across nearby locations (for employees willing to cross-cover).
- Skill-level ratings (e.g., "expert line cook" vs. "trained line cook") used as soft rule.
- Forecast accuracy reporting (variance of forecast vs. actual sales).
- Shift bidding (employees rank preferred open shifts; engine assigns).

### v4+ (longer horizon)
- Demand forecasting using machine learning that incorporates weather, local events, marketing pushes, and historical patterns.
- Workforce planning (annual headcount and hiring plan tied to growth forecast).
- Compliance pack: jurisdiction-aware rules library covering minor labor laws, predictive scheduling, paid sick leave accruals, meal/break rules.
- White-label / multi-tenant SaaS version if we decide to commercialize externally.
- Mobile manager app (currently web-only on phones).

---

## 12. Development Roadmap

A pragmatic 9–12 month plan assuming a small team (2–3 engineers, 1 designer, 1 PM/operator).

### Phase 0 — Discovery (Weeks 1–3)
- Shadow current scheduling at the pilot location for 2 weeks.
- Document the current labor cost workflow, current availability collection method, and current trade-shift method.
- Define the canonical role list, position list, and labor-target rules used today.
- Collect 8 weeks of historical hourly sales for forecasting baselines.
- Lock the MVP scope and design wireframes.

### Phase 1 — Foundations (Weeks 4–8)
- Repo, CI/CD, environments (dev, preview, prod).
- Auth, organization/location/user/role data model.
- Employee management UI.
- Availability rules and time-off requests, with approvals.
- Notifications infrastructure (in-app + email; push wired but deferred surface-level UX).

### Phase 2 — Scheduling core (Weeks 9–14)
- Sales forecast entry/import.
- Labor budget configuration.
- Required staffing per hour per position.
- Greedy schedule generator with hard-rule enforcement.
- Drag-and-drop draft editor with live labor % readout and conflict warnings.
- Publish flow with locking and notifications.

### Phase 3 — Employee self-service (Weeks 15–18)
- Mobile-first employee portal.
- Drop and pickup workflows.
- Manager approval inbox.
- Shift reminders and push notifications.
- PWA installability.

### Phase 4 — Pilot and iterate (Weeks 19–24)
- Pilot at 1–2 locations for 6 weeks.
- Weekly feedback cycles with managers and employees.
- Tighten scheduling heuristics, fix edge cases, polish UI.
- MVP launch readiness review.

### Phase 5 — Rollout and v2 prep (Weeks 25–36)
- Roll out to all locations in waves.
- Begin v2 work in parallel: swap workflow, POS integration, OR-Tools optimizer, audit-log UI.
- Establish ongoing operating cadence: weekly bug bash, monthly retro with managers, quarterly roadmap review.

---

## 13. Possible AI Automation Features

AI is positioned as **augmentation, not replacement**. In every case below, AI proposes; manager confirms.

### 13.1 Smart sales forecasting
A time-series model trained on each location's historical hourly sales, factoring weather, day of week, holiday calendar, and known marketing events. Outputs an hourly forecast and a confidence band. Replaces manual sales-projection entry once a location has 60+ days of data.

### 13.2 Auto-tuned staffing curves
Once we have actual labor data alongside actual sales, the system learns the optimal sales-per-labor-hour for each role at each location and surfaces curve recommendations. Manager can accept, edit, or reject.

### 13.3 Schedule narrative
After generation, the system writes a plain-English summary: "Generated 84 shifts totaling 612 hours and $9,840 in labor cost (target 14.2%, actual 14.1%). 3 shifts left open in BOH on Friday close — recommend offering to Maria or Devon based on availability and history." This makes the engine's decisions transparent and reviewable in seconds.

### 13.4 Conflict resolution suggestions
When a manager rejects a generated assignment, the AI proposes the next-best swap rather than making the manager hunt for it.

### 13.5 No-show and call-out prediction
Using historical patterns (lateness, last-minute drops, day-of-week behavior), surface a per-shift "risk score" so managers can have a backup plan in mind for high-risk shifts. Used cautiously — the model is suggestive, never punitive.

### 13.6 Smart open-shift offers
When a shift opens up, instead of broadcasting to everyone, the system targets the employees most likely to accept (based on past pickup behavior), most qualified, and within hours/availability rules — increasing fill rate while reducing notification fatigue.

### 13.7 Natural language schedule edits
Manager types "give Marcus Saturday off and move his shift to Devon if she's available" and the system proposes the change for one-tap approval. Built on top of the rules engine so suggestions are always feasible.

### 13.8 Onboarding assistant
A conversational helper that walks a new GM through location setup, asking questions and pre-filling defaults from similar locations.

### 13.9 Anomaly detection in actuals
Flags when actual labor diverged unusually far from forecast and offers a likely cause ("unexpectedly slow Tuesday lunch — over-staffed by 12 hours") to feed back into next week's forecast.

### 13.10 Compliance copilot
A rule-aware checker that warns: "Two of next week's shifts violate the 11-hour rest rule for Sarah. Suggest moving her Friday close to Saturday open." Especially valuable as the compliance pack expands to multiple jurisdictions.

---

## 14. Open Questions and Decisions Needed

These should be resolved before development starts so the team isn't blocked.

1. Which POS platform(s) do we need to integrate with first?
2. What payroll system will we export to, and what file format does it expect?
3. Do we operate in any jurisdictions with predictive scheduling laws (Seattle, NYC, Oregon, San Francisco, Philadelphia, Los Angeles, Chicago, etc.)?
4. Do we have any unionized locations with collective-bargaining-driven scheduling rules?
5. What overtime rules apply (federal vs. California-style daily OT)?
6. Will employees use shared tablets at the location or only personal phones?
7. Do we want employees to see each other's contact info, or only first names and shift times?
8. Who approves availability changes — any manager, or only GM?
9. How far in advance do we publish schedules today, and is that a constraint we want to enforce in software?
10. What is our launch location, and who are the manager and employee design partners?

---

## 15. Glossary

- **FOH / BOH:** Front of house / Back of house.
- **Labor %:** Labor cost divided by sales for a given period.
- **Open shift:** A shift on the schedule with no employee assigned, available for pickup.
- **Drop:** Employee request to be relieved of an already-assigned shift.
- **Pickup:** Employee request to take an open shift.
- **Swap:** Two employees trade specific shifts.
- **Hard rule:** A constraint the engine cannot violate.
- **Soft rule:** A weighted preference the engine optimizes against.
- **Predictive scheduling:** Local laws requiring schedules be published a fixed number of days in advance with penalties for late changes.
- **Clopen:** A close shift followed immediately by the next morning's open shift.

---

*End of document.*
