# Booking page redesign — clickable prototype

A static prototype of a redesigned **Book a job** page for the DFRNT booking app.
Live: https://deliver-different-testing.github.io/booking-page-redesign/

Plain HTML, CSS and a little JavaScript. No build step. Colours, type and radii are the DFRNT Hub's own tokens (`site.less`, Feb 2026 rebrand).

| Page | What it shows |
|---|---|
| `gate.html` | "Who are you booking for?" — the staff gate, one card |
| `index.html` | The booking form (desktop). Vehicle, preset/dimensions, dangerous goods, when, tracking and service selection all work |
| `index.html#extras` | Accessorial charges as a side sheet over the form (`extras.html` redirects here) |
| `booked.html` | Confirmation with labels, tracking and follow-ups |
| `mobile.html` | The same form at phone width with a pinned total and Book button |

Addresses, prices and services are sample data. `booking-page-inventory.md` is the audit of the current AngularJS page this design was drawn from.

This is separate from the earlier `booking-redesign` repo (React/Vite prototype, March 2026).
