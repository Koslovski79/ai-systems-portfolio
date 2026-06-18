# POS System

Restaurant point-of-sale system with table management, kitchen
display, and full reporting. Production-deployed.

**Status:** Production. Private repository (proprietary).

## What it does

End-to-end restaurant operations:
- Order entry (dine-in, takeaway, delivery)
- Table management with floor plan
- Kitchen display system (KDS) for chefs
- Split bills, discounts, voids (with reason tracking)
- Stock movement tracking
- Daily / weekly / monthly reports
- Cash-up reconciliation

Built for a single-location restaurant first, then expanded to multi-location.

## Architecture

```
  +-----------+   +-----------+   +-----------+
  | Front-of- |   | Back-of-  |   |  Manager   |
  | house     |   | house     |   |  console   |
  | (waiter)  |   | (kitchen) |   |            |
  +-----+-----+   +-----+-----+   +-----+-----+
        |               |               |
        +-------+-------+-------+-------+
                |               |
                v               v
        +---------------+   +-----------+
        |  Django REST  |   |  WebSocket|
        |  API          |   |  (KDS)    |
        +-------+-------+   +-----+-----+
                |                 |
                +--------+--------+
                         |
                         v
                  +-------------+
                  | PostgreSQL  |
                  |  (46 models)|
                  +-------------+
```

## Stack

- **Django 4.2** — backend + admin
- **Django REST Framework** — JSON API for terminals
- **Django Channels** — WebSocket for real-time kitchen display
- **PostgreSQL** — primary data store
- **Redis** — channel layer + cache
- **htmx + Alpine.js** — lightweight front-end interactivity
- **Tailwind CSS** — UI styling

## Key features

- **46 data models** — menu, modifiers, tables, orders, payments, voids,
  discounts, shifts, employees, suppliers, stock, etc.
- **Kitchen Display System (KDS)** — orders flow to the kitchen in
  real-time; chefs mark items as fired/plated/served
- **Table management** — visual floor plan, table states (free/seated/
  ordered/served/bill-out/paid)
- **Split bills + partial payments** — flexible enough for any real
  table scenario
- **Stock movement tracking** — every order triggers automatic stock
  decrements with reconciliation reports
- **Cash-up reconciliation** — end-of-shift cash count vs system totals
- **Offline-tolerant terminals** — front-of-house tablets survive brief
  network drops and sync on reconnect

## Lessons learned

1. **A POS is a real-time system pretending to be a CRUD app.** Order
   flow, kitchen updates, and bill payments must feel instant. WebSocket
   for KDS, async writes for the rest.
2. **Money math is still unforgiving.** Decimals everywhere.
3. **Voids and discounts need a reason field, every time.** Audit +
   training + theft prevention — all three depend on it.
4. **Offline mode is hard but non-negotiable.** Restaurant WiFi is bad.
   The terminal must keep working when the network drops.
5. **Stock reconciliation is the operationally painful part.** Build
   the reports first, then the POS UI on top.