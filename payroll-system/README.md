# Payroll System

Full-stack payroll management for South African small and medium
enterprises. SARS-compliant, multi-tenant, production-deployed.

**Status:** Production. Private repository (handles employee PII).

## What it does

End-to-end payroll for SMEs:
- Employee management with SA ID validation
- SARS tax tables (PAYE, UIF, SDL)
- Payslip generation (PDF)
- EMP201 / EMP501 monthly and biannual submissions
- Leave management
- Audit log

Designed for businesses that don't want to pay R500+/month for legacy
SaaS payroll but need SARS compliance.

## Architecture

```
  +------------------+        +-------------------+
  |   Django admin   |  HTTP  |   Django REST     |
  |   (UI for HR)    | <----> |   Framework API   |
  +------------------+        +---------+---------+
                                        |
                                        v
                              +-------------------+
                              |   PostgreSQL      |
                              |   (multi-tenant)  |
                              +-------------------+
                                        |
                                        v
                              +-------------------+
                              |   Background      |
                              |   workers         |
                              |   (Celery)        |
                              +-------------------+
                                        |
                                        v
                              +-------------------+
                              |   SARS submission |
                              |   integration     |
                              +-------------------+
```

## Stack

- **Django 4.2** — web framework + admin
- **Django REST Framework** — JSON API for mobile/3rd-party
- **PostgreSQL** — primary data store (multi-tenant)
- **Celery + Redis** — async task queue (payslip generation, SARS submissions)
- **reportlab** — PDF payslip generation
- **pytest + factory_boy** — test suite

## Key features

- **SARS tax tables** — PAYE, UIF, SDL computed correctly per tax year
- **EMP201 monthly submissions** — generate SARS-ready CSV/Excel
- **EMP501 biannual reconciliations** — interim and final reconciliations
- **Multi-tenant** — single deployment, multiple companies
- **Audit log** — every change tracked with user + timestamp
- **Role-based access** — admin, payroll clerk, employee viewer

## Lessons learned

1. **Money math is unforgiving.** Float arithmetic is fine for
   visualization, never for storage. Decimal types everywhere.
2. **Tax tables change.** Don't hardcode them — store as DB rows,
   versioned by effective date.
3. **Audit log is a compliance requirement, not a nice-to-have.** Build
   it from day one.
4. **Payslip generation is the hot path.** It must be reliable, fast,
   and produce identical PDFs across runs. Celery + a deterministic
   template engine is the answer.
5. **Multi-tenancy with row-level isolation beats database-per-tenant
   for SMEs.** Easier to operate, cheaper to host, easier to back up.