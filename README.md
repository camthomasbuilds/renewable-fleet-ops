# Case Study: Renewable Fleet Operations Intelligence Platform

## Context

A large energy company operates 100+ utility-scale solar and wind sites generating thousands of megawatts. Operations teams tracked site performance through disconnected spreadsheets, manual SCADA pulls, and siloed reporting tools. There was no unified system to answer fundamental questions: "How is the fleet performing today?", "Where are we losing generation?", or "Are we on track for the month?"

Month-end financial close required weeks of manual reconciliation: operators at each site categorizing downtime events, commercial teams chasing curtailment data, and finance teams stitching together generation actuals vs. budget.

## My Role

Senior Product Manager: responsible for product vision, roadmap, and delivery of the fleet-wide operations reporting platform. I managed the full product lifecycle from discovery through architecture decisions, delivery, and adoption.

## Discovery & Problem Framing

**User research findings:**
- Site managers spent hours daily pulling SCADA data into spreadsheets to track generation
- Month-end close took 2–3 weeks because DTE (Down-Time Event) categorization was manual and inconsistent across sites
- Curtailment data was tracked informally: commercial teams couldn't quantify lost revenue from grid-directed curtailments
- No fleet-wide view existed: regional directors couldn't compare site performance without requesting ad-hoc reports
- Data quality issues (missing intervals, duplicate records, timezone inconsistencies) were discovered reactively

**Key insight:** This wasn't a dashboard problem. It was a **data product** problem. The fleet needed a system of record for generation performance that could be trusted for financial close, operational decisions, and commercial negotiations.

## Product Strategy

I structured the platform around three user workflows:

1. **Daily Operations**: Real-time generation monitoring at inverter/turbine granularity, anomaly surfacing, and site-level health checks
2. **DTE Management**: Structured workflow for categorizing, validating, and closing downtime events with full audit trails
3. **Monthly Close & Reporting**: Automated generation variance reporting, curtailment quantification, and plan-vs-actual analysis

## Architecture Decisions I Drove

### Rust/Axum over Python/FastAPI for the backend
- **Decision:** Migrate the production backend from Python/FastAPI to Rust/Axum
- **Rationale:** Fleet-wide time-series queries (aggregating millions of data points across 100+ sites) were too slow on the Python backend (~3-5s response times). Rust delivered 10x performance improvement, enabling real-time interactive dashboards. The legacy Python service remained for non-critical batch jobs.

### TimescaleDB for time-series storage
- **Decision:** Use TimescaleDB (PostgreSQL extension) rather than a purpose-built time-series database (InfluxDB, TimescaleCloud)
- **Rationale:** Operations data has both relational characteristics (sites, inverters, meters, DTE events with categories and workflows) and time-series characteristics (sub-hourly generation data). TimescaleDB gave us PostgreSQL's relational power with time-series query performance: no need to split the data across two systems.

### Site-level DTE workflow over fleet-level automation
- **Decision:** Build a structured DTE categorization workflow where operators close out events per-site, rather than auto-categorizing losses from SCADA data
- **Rationale:** DTE categorization has financial and contractual implications (warranty claims, curtailment compensation, insurance). Automated categorization would have been faster but couldn't be trusted for financial close without human validation. The workflow gives operators the tools to close events quickly while maintaining accountability.

### Vue 3/Nuxt 3 + ECharts for visualization
- **Decision:** Build a purpose-built frontend rather than using a BI tool (Tableau, Power BI)
- **Rationale:** BI tools couldn't support the interactive workflows we needed (DTE editing, drill-down from fleet to site to inverter, real-time refresh). A custom frontend let us build the exact operational experience users needed, integrated with the backend APIs.

## Go-to-Market & Adoption

I owned the go-to-market strategy end-to-end:

- **Rollout plan**: Phased regional deployment starting with the highest-pain sites (largest fleets, most complex DTE workflows), then expanding based on lessons learned
- **Stakeholder alignment**: Ran executive demos, operations leadership buy-in sessions, and "day in the life" walkthroughs showing before/after workflows
- **Training & onboarding**: Coordinated hands-on training for site operators, regional managers, and commercial teams: tailored to each persona's workflow
- **Adoption playbook**: Defined success metrics per site (daily active usage, DTE close rate, time-to-month-end), tracked adoption weekly, and intervened when sites stalled
- **Legacy retirement**: Managed the transition from spreadsheet-based reporting, including a parallel-run period where both systems operated side-by-side until users gained confidence

The GTM approach was deliberate: prove value at high-pain sites first, use those wins to build internal demand, then scale. This avoided the "mandated tool no one uses" failure mode.

## Delivery & Results

### Phase 1: Generation Visibility
- Automated daily ingestion pipeline pulling from SCADA systems across all sites
- Fleet-level dashboard with drill-down to site → inverter/turbine level
- Plan-vs-actual generation comparison with variance explanations

### Phase 2: DTE & Loss Accounting
- Structured DTE workflow: create, categorize (equipment failure, grid curtailment, weather, planned maintenance), validate, close
- Daily and monthly DTE summaries with loss quantification in MWh and dollars
- Audit trail on every DTE edit for financial compliance

### Phase 3: Curtailment & Close
- Curtailment breakdown by category (economic, reliability, environmental/avian for wind)
- Month-end close dashboard showing open items, unvalidated DTEs, and reconciliation status
- Automated report generation for executive review

### Business impact
- Consolidated 100+ site spreadsheets into a single system of record
- Reduced month-end close cycle time through automated reconciliation
- Surfaced millions in previously-untracked curtailment losses for commercial recovery
- 10x query performance improvement enabling real-time operational decisions
- Adopted as the authoritative source for renewable fleet performance data

## Lessons Learned

1. **Data quality is the product**: The dashboards and workflows are important, but the real value is trustworthy data. We spent as much time on ingestion validation (gap detection, duplicate removal, timezone normalization) as on the user-facing features. If operations teams don't trust the numbers, they won't use the system.

2. **Month-end close drives adoption**: The feature that drove adoption wasn't the real-time dashboard ((nice to have). It was the DTE close workflow that eliminated 2 weeks of manual spreadsheet reconciliation. Building for the highest-pain workflow first created urgency around adoption.

3. **Performance is a feature, not an optimization**: When fleet-wide queries took 3-5 seconds on the Python backend, users defaulted to their spreadsheets. When the Rust migration brought response times under 300ms, usage patterns changed completely. Users started doing ad-hoc analysis they never would have attempted before. The backend rewrite wasn't premature optimization; it was a product decision.

4. **Don't automate what needs accountability**: We deliberately chose human-in-the-loop DTE categorization over fully automated loss accounting. In a domain where categories have contractual and financial implications, the cost of an automated wrong answer is higher than the cost of a human review step.
