# Renewable Fleet Operations Platform

## The Business Problem

A Fortune 200 energy company operates 700+ utility-scale solar and wind sites representing $2.4B in annual generation revenue. There was no system of record for fleet performance. Operations teams ran on disconnected spreadsheets. Month-end financial close took 2-3 weeks of manual reconciliation. Millions in curtailment losses went untracked because no one could quantify them.

**The strategic question:** How do you build a trusted system of record for $2.4B in revenue from scratch, and get 300+ users to adopt it without a mandate?

## My Role

Senior Product Manager. Owned product vision, architecture decisions, delivery, and go-to-market for the fleet-wide operations platform (GenE + OpsReport).

## The Key Decisions

### 1. This is a data product, not a dashboard

User research revealed the core problem wasn't visibility. It was trust. Site managers wouldn't use any system they couldn't trust for financial close. I framed the product around data integrity first, visualizations second.

### 2. Rust backend over Python (product decision, not technical)

The Python backend delivered 3-5 second response times on fleet queries. Users went back to spreadsheets. I drove the migration to Rust/Axum, delivering sub-300ms responses. This wasn't optimization. It was a product bet: interactive performance would change usage patterns. It did. DAU went from 30 to 300+.

### 3. Human-in-the-loop for financial accountability

We could have automated loss categorization from SCADA data. I chose structured human workflows instead. DTE (Down-Time Event) categories have contractual and financial implications: warranty claims, curtailment compensation, insurance. The cost of an automated wrong answer exceeded the cost of a human review step.

### 4. Custom frontend over BI tools

BI tools (Tableau, Power BI) couldn't support the interactive workflows we needed: DTE editing, fleet-to-inverter drill-down, real-time refresh. A purpose-built Vue 3/Nuxt 3 frontend delivered the exact operational experience required.

### 5. TimescaleDB for hybrid data

Operations data is both relational (sites, equipment, DTE events with workflows) and time-series (sub-hourly generation). TimescaleDB gave us PostgreSQL's relational power with time-series performance. One system instead of two.

## Go-to-Market Strategy

I owned adoption end-to-end:

- **Started with highest-pain sites** (most complex DTE workflows, largest fleets). If it works there, it works everywhere.
- **Parallel-run before cutover**. Both systems ran side-by-side until operators confirmed the numbers matched.
- **Trained by persona, not feature**. Site operators, regional managers, commercial teams, and finance each got workflow-specific training.
- **Tracked adoption weekly by site**. When sites stalled, I diagnosed why (data quality issue, training gap, missing feature) and intervened specifically.
- **Retired the spreadsheets**. The product isn't adopted until the old process is dead.

Result: 30 to 300+ DAU in 18 months. Zero usage mandates.

## Results

| Metric | Outcome |
|--------|---------|
| **System of record** | Consolidated 100+ site spreadsheets into single authoritative source |
| **Revenue visibility** | Surfaced millions in previously-untracked curtailment losses |
| **Adoption** | 300+ DAU through product-led growth |
| **Performance** | 10x query improvement (3-5s → sub-300ms) |
| **Financial close** | Reduced month-end cycle from weeks to days |
| **Scale** | 700+ sites, 15GW, 900M+ time-series records |

## What This Demonstrates

- **0-to-1 product creation** at enterprise scale
- **Architecture decisions as product decisions** (Rust migration drove adoption, not just performance)
- **GTM discipline** in enterprise environments (adoption without mandates)
- **Business judgment on automation boundaries** (human-in-the-loop where accountability matters)
- **Data quality as product strategy** (trust drives adoption more than features)
