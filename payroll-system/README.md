# Payroll System

Full-stack payroll management for South African small and medium enterprises.
SARS-compliant, multi-tenant, production-deployed.

**Status:** Production — private repository (handles employee PII)

---

## What It Does

End-to-end payroll for South African SMEs who need SARS compliance without
paying R500+/month for legacy SaaS platforms.

- Employee management with SA ID validation
- SARS tax calculations — PAYE, UIF, SDL
- Payslip generation (PDF)
- EMP201 monthly submissions
- EMP501 biannual reconciliations (interim and final)
- Leave management
- Full audit log — every change tracked with user and timestamp
- Role-based access — admin, payroll clerk, employee viewer

---

## Architecture

```
  +------------------+        +-------------------+
  |   Django admin   |  HTTP  |   Django REST     |
  |   (HR interface) | <----> |   Framework API   |
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
                              |   Celery workers  |
                              | (payslip gen,     |
                              |  SARS submissions)|
                              +-------------------+
                                        |
                                        v
                              +-------------------+
                              |   SARS submission |
                              |   integration     |
                              +-------------------+
```

---

## Stack

| Layer | Technology |
|---|---|
| Web framework | Django 4.2 |
| API | Django REST Framework |
| Database | PostgreSQL (multi-tenant) |
| Task queue | Celery + Redis |
| PDF generation | ReportLab |
| Test suite | pytest + factory_boy |

---

## Key Features

**SARS compliance**
PAYE, UIF, and SDL computed correctly per tax year. Tax tables stored as
versioned database rows — not hardcoded — so annual SARS updates are a
data change, not a code deployment.

**EMP201 and EMP501 submissions**
Monthly and biannual SARS submissions generated as SARS-ready CSV/Excel.
EMP501 handles both interim and final reconciliations.

**Multi-tenancy**
Single deployment, multiple companies. Row-level tenant isolation — easier
to operate, cheaper to host, and simpler to back up than database-per-tenant.

**Audit log**
Every change logged with user, timestamp, and before/after values.
A compliance requirement, not a nice-to-have.

**Payslip generation**
Deterministic PDF output via Celery + ReportLab. Identical output across
runs — critical for reissuing historical payslips.

---

## Lessons Learned

1. **Money math is unforgiving.** Float arithmetic is fine for display, never for storage or calculation. `Decimal` types everywhere, from database to PDF.
2. **Tax tables change annually.** Store them as versioned database rows with an effective date. Hardcoding tax rates means a code deployment every February.
3. **Audit log is a compliance requirement.** Build it from day one. Retrofitting it into a production payroll system is painful.
4. **Payslip generation is the hot path.** It must be reliable, fast, and deterministic. A Celery task with a template-driven PDF engine handles this cleanly.
5. **Row-level multi-tenancy beats database-per-tenant for SMEs.** Simpler operations, lower hosting cost, easier backups — with the same data isolation guarantees.
