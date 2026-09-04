# samaran

**Domain:** Fleet/Vehicle Rental. **Type:** Odoo customization project (bespoke, not a shared
product).
**Repo:** `motox` (`git@bitbucket.org:teapottechies/motox.git`, branch `17.0`) — in the
`odoo-17/tpt-ind` workspace.
**Host:** `ap-southeast-1` (standard backend, threaded).

## What's known from code

Vehicle rental/fleet-booking functionality built on top of Odoo's `fleet` module: 3 modules —
`fleet_booking` (extends `fleet.vehicle` with a booking/rental workflow and ID-proof file
uploads), `fleet_booking_timeline` (Gantt/timeline view via OCA `web_timeline`),
`fleet_vehicle_extend` (Fleet menu reorganization).

**Caution:** `motox/README.md`, `implementation-v1.md`, and `implementation-v1-fn.md` describe
an unrelated "VisWoX ERP" construction-management product — these are stale/copied docs from a
different project and don't reflect what `motox` actually does. Trust the manifests/code instead.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, why this client needed fleet booking
specifically.
