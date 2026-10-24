# Logística Local & Rotas — case study

[Português](README.md) · **English**

A last-mile delivery system for small and mid-sized carriers and retailers in Mato Grosso, Brazil. It connects the whole operation in real time: the dispatcher (web dashboard), the driver (mobile app) and the customer (tracking link).

- **Status:** pilot in production from October 8 to October 15, 2026; its final version was 1.5.0. It was replaced by [Tropeiro](https://sankler.com.br/en/#tropeiro), which took over the pilot's data and carried on the work.
- **Code:** private. This repository shows the product, the decisions behind it and the method.
- **Screenshots:** the system running locally with fictitious data in Cuiabá and Várzea Grande. The interface is in Portuguese because the product is built for the Brazilian market.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/painel-escuro.png">
  <img alt="Dispatch dashboard: live map of drivers, deliveries and risk zones in Cuiabá, next to the list of open deliveries" src="assets/painel.png">
</picture>

## Where it started: complaints, not features

Before writing any code, I mapped the pain points of the people who dispatch, deliver and receive orders in Brazil. Sources included Reclame Aqui (Brazil's largest consumer complaint site), the press, reviews of foreign competitors (Onfleet, Routific, Circuit), legislation, and interviews with users on both sides, such as drivers and administrators. The result was 22 pain points across three user profiles, each with a source, a priority and its status in the code, updated with every release. That record is the product backlog.

Some of the pain points that shaped the project (the figures describe the market, not the product):

- **Customer:** "marked as delivered, but I never got it" is one of the most frequent complaints on Reclame Aqui, and the store and the carrier can take up to 10 days to investigate it.
- **Customer:** "where is my order?". According to customer service vendors, 30% to 50% of support tickets at online stores are about order status.
- **Driver:** going all the way to the door, failing to deliver and not getting paid for the trip.
- **Driver:** poor mobile signal, addresses the map can't find, and safety on the street.
- **Dispatcher:** schedules put together by hand from WhatsApp messages and spreadsheets, and orders typed in again because they already exist in another system.

## How the system answers each pain point

| Pain point | What the system does |
|---|---|
| "Delivered, but I never got it" | A 4-digit PIN that only the customer has. Proof of delivery with photo, GPS, signature, a geofence of about 150 m around the address and an audit hash. The driver never receives the PIN through the API, and automated tests check this. |
| "Where is my order?" | A public tracking link with a live map, a timeline and masked personal data (as required by the LGPD, Brazil's data protection law), ready to send over WhatsApp. |
| Wasted trip | A failed attempt is recorded with the reason, call attempts, photo and GPS. Dispatch decides between another attempt and a return, and whether the driver gets paid for the attempt. |
| Address the map can't find | Postal code (CEP) lookup with a fallback source and visible pin accuracy. The driver suggests where the door is, the dispatcher approves it, and the corrected pin is used for future deliveries to the same address. |
| Poor signal | Offline queue: proofs of delivery and alerts are resent when the connection comes back. |
| Dispatch by WhatsApp and spreadsheet | Multi-stop route planning with vehicle load limits, real-time dispatch and CSV import with a per-row error preview. |
| Safety | A silent panic button that raises a real-time alert at dispatch, and risk zones on the map. |
| Self-employed drivers (MEI) | A wallet with a statement, credit when a delivery is completed with proof, and tracking of the annual revenue cap for MEI, Brazil's micro-entrepreneur regime. |
| Payment on delivery | Dynamic Pix, Brazil's instant payment system, following the Central Bank standard (QR code and copy-and-paste code). |

## Screens

**Route planning.** Optimized stop order, street-level route and vehicle load gauge.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/rotas-escuro.png">
  <img alt="Route planning screen with four numbered stops on the map, the street route and the motorcycle's weight and volume capacity" src="assets/rotas.png">
</picture>

**Deliveries.** A failed attempt stays highlighted until dispatch decides what to do.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/entregas-escuro.png">
  <img alt="Delivery list with a notice about one delivery awaiting a dispatch decision, plus PIN, address, driver and driver payout" src="assets/entregas.png">
</picture>

**Mobile.** Customer tracking with PIN, the driver's route with shortcuts to Waze and Google Maps, and the MEI wallet.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/celulares-escuro.png">
  <img alt="Three phone screens: order tracking with the confirmation PIN, the driver's list of stops, and the MEI wallet with balance and annual cap" src="assets/celulares.png">
</picture>

**Reports.** Deliveries, proofs of delivery, time on route, driver credits and Pix by period and by driver, with CSV and PDF export.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/relatorios-escuro.png">
  <img alt="Reports with completed deliveries, proofs within the geofence, average time on route, driver credits and a chart of deliveries per day" src="assets/relatorios.png">
</picture>

## How it was built

- **Architecture.** A monorepo with three applications:
  - API in Node.js, Express, Prisma and Socket.io;
  - dashboard in Next.js 15, Tailwind and Leaflet;
  - driver app in React Native with Expo, which also runs in the phone's browser.
- **Real time with role-based permissions.**
  - Dispatch receives everything, a driver only their own deliveries, and the public page only its own delivery.
  - Events carry only a notification. The data is then fetched from the API, which enforces the permissions.
- **Self-hosted maps.**
  - The map of the whole state of Mato Grosso was hosted on the server itself (PMTiles with OpenStreetMap data), with no dependency on a third-party map service.
  - Stop order uses nearest neighbor with 2-opt, and the street route comes from OSRM.
- **Security.**
  - Drivers can only sign up through a single-use invite.
  - Changing the email or password ends all previous sessions.
  - Two-step verification by email is optional and comes with recovery codes.
  - Tests with real Socket.io clients check who receives each event.
- **Quality.**
  - 169 API regression tests and end-to-end tests with Playwright.
  - Every release is checked in Chromium and WebKit, on desktop and mobile screens, in light and dark themes.
- **Deployment.**
  - Releases are published as a package with a SHA-256 checksum.
  - Every migration is rehearsed on a copy of the database, with a backup before it is applied.
  - After a release, production only gets read-only checks.
  - On the micro VM, the API used about 71 MiB of memory and the dashboard about 95 MiB.

## Next steps

After the switch, these steps went to Tropeiro:

- Separate data by client company, to offer the system as a subscription.
- Automatic customer notifications ("out for delivery", "we couldn't deliver").
- Automatic payment confirmation through a Pix provider integration.

## Contact

Built by **Sankler Bergman**. The code is private; I'm happy to show you the system running.

[LinkedIn](https://www.linkedin.com/in/sankler-bergman-64962511b/) · [Other projects](https://github.com/sanklerbergman)
