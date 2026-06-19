# POS System

Restaurant point-of-sale system with table management, kitchen display, and
full operational reporting. Production-deployed.

**Status:** Production — private repository (proprietary)

---

## What It Does

End-to-end restaurant operations for a single-location deployment, expanded
to support multi-location.

- Order entry — dine-in, takeaway, delivery
- Table management with visual floor plan
- Kitchen Display System (KDS) — real-time order flow to kitchen
- Split bills, partial payments, discounts, voids (with reason tracking)
- Stock movement tracking with automatic decrements per order
- Daily, weekly, and monthly operational reports
- Cash-up reconciliation — end-of-shift cash count vs system totals
- Offline-tolerant terminals — survives brief network drops and syncs on reconnect

---

## Architecture

```
  +-----------+   +-----------+   +-----------+
  | Front-of- |   | Back-of-  |   |  Manager  |
  |   house   |   |   house   |   |  console  |
  |  (waiter) |   | (kitchen) |   |           |
  +-----+-----+   +-----+-----+   +-----+-----+
        |               |               |
        +-------+-------+-------+-------+
                        |
              +---------+---------+
              |                   |
              v                   v
      +---------------+   +-----------+
      |  Django REST  |   | WebSocket |
      |     API       |   |   (KDS)   |
      +-------+-------+   +-----+-----+
              |                 |
              +---------+-------+
                        |
                        v
                 +-------------+
                 | PostgreSQL  |
                 | (46 models) |
                 +-------------+
```

---

## Stack

| Layer | Technology |
|---|---|
| Backend | Django 4.2 |
| API | Django REST Framework |
| Real-time KDS | Django Channels (WebSocket) |
| Database | PostgreSQL (46 models) |
| Channel layer + cache | Redis |

---

## Key Features

**46 data models**
Menu, modifiers, tables, orders, payments, voids, discounts, shifts,
employees, suppliers, stock, and more. The schema reflects the full
complexity of restaurant operations — not a simplified demo.

**Kitchen Display System**
Orders flow to the kitchen in real-time via WebSocket. Chefs mark items
as fired, plated, and served. Front-of-house sees status updates instantly.

**Table management**
Visual floor plan with live table states: free / seated / ordered /
served / bill-out / paid.

**Split bills and partial payments**
Flexible enough to handle any real table scenario — split by seat, by
item, or by arbitrary amount.

**Stock movement tracking**
Every order triggers automatic stock decrements. Reconciliation reports
show theoretical vs actual stock levels per shift.

**Cash-up reconciliation**
End-of-shift cash count reconciled against system totals. Discrepancies
flagged with drill-down to individual transactions.

**Offline-tolerant terminals**
Front-of-house tablets continue accepting orders during brief network
outages and sync on reconnect. Restaurant WiFi is unreliable — the
terminal cannot be.

---

## Lessons Learned

1. **A POS is a real-time system pretending to be a CRUD app.** Order flow, kitchen updates, and payment processing must feel instant. WebSocket for the KDS; async writes everywhere else.
2. **Money math is still unforgiving.** Decimals everywhere — split-bill rounding errors compound fast across a busy service.
3. **Voids and discounts need a reason field, every time.** Three reasons: audit trail, staff training, and theft prevention. All three matter.
4. **Offline mode is hard but non-negotiable.** Restaurant WiFi drops. The terminal must keep working. Build sync-on-reconnect from the start, not as an afterthought.
5. **Build the reconciliation reports first.** Understanding the data model deeply enough to produce accurate stock and cash reports forces you to get the schema right before building the UI on top.
