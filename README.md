# AVINYA

## Community-assisted emergency response

AVINYA is a Smart India Hackathon 2026 MVP that helps a bystander share a structured emergency alert with nearby, verified community responders after contacting emergency services.

The prototype connects three roles in one workflow:

- **Bystander:** calls 108, describes the incident, optionally shares a scene photo, and confirms GPS location.
- **Responder:** registers with identity and qualification details, goes online, receives nearby alerts, and accepts or rejects them.
- **Admin:** reviews responder submissions and verifies or rejects uploaded documents.

AVINYA is a community coordination layer. It does not replace 108, an ambulance, a doctor, or an official emergency dispatch system.

## The problem

During the first few minutes of an emergency, professional help may still be on the way while capable people are already nearby. Bystanders often have no reliable way to share the situation with trained community responders, and responders have no shared view of which incidents need help.

## The solution

AVINYA adds a community-response layer alongside official emergency services. It helps a bystander share a structured alert and location, matches the alert with nearby verified responders, provides emergency-specific first-aid guidance, and keeps participants updated in real time.

The platform is designed to complement official response systems, not replace them. The bystander is prompted to call 108 before creating an alert, and professional emergency services remain the primary source of medical assistance.

## Why AVINYA matters

AVINYA is designed to reduce the coordination gap between the moment an emergency is reported and the moment professional help arrives. Its intended value is to:

- Make nearby trained assistance discoverable.
- Give bystanders a structured way to share location and incident details.
- Keep responders informed about alert status and ambulance arrival.
- Provide immediate, category-specific first-aid guidance while help is arranged.

The MVP measures the workflow through response status, responder matching, radius expansion, and estimated arrival information. It does not claim to replace emergency dispatch or provide clinical diagnosis.

## MVP workflow

1. The bystander selects **Send Emergency Alert** and is prompted to call 108.
2. They choose an emergency type: accident, cardiac emergency, breathing problem, injury, bleeding, burns, unconscious person, or other.
3. They capture an optional scene photo using the device camera.
4. The browser detects the bystander's GPS location and submits the alert.
5. The server matches the alert with verified responders who are available and have a recent location.
6. Matching starts within 1 km and expands to 2 km after 20 seconds if nobody has accepted.
7. Responders receive real-time updates, review the alert, and accept or reject it.
8. The accepted responder with the best estimated arrival time becomes primary; other accepted responders remain available as secondary support.
9. The bystander can view the alert status, responder details, map location, first-aid guidance, and ambulance arrival status.

## Key capabilities

- Structured emergency reporting with GPS coordinates and optional JPEG/PNG scene photo.
- Leaflet map views for emergency and responder locations.
- Fresh-location matching: responder locations older than 15 minutes are excluded.
- Haversine distance and straight-line ETA estimates for prototype dispatch logic.
- Real-time emergency updates using Socket.IO.
- Responder registration with document upload and pending verification status.
- Availability toggle and periodic responder location synchronisation.
- Deterministic first-aid guidance for each supported emergency category, with safety messaging.
- SQLite persistence with automatic database/table-column initialisation.
- Session/local storage recovery for an active bystander alert and selected responder.

## Responder matching flow

```text
Emergency created
	|
	v
Find verified, available responders
	|
	v
Ignore locations older than 15 minutes
	|
	v
Search within 1 km
	|
   +----+----+
   |         |
Accepted   No acceptance after 20 seconds
   |         |
   v         v
Help on   Expand search to 2 km
the way        |
	      v
       Notify newly matched responders
```

When multiple responders accept, the responder with the best estimated arrival time becomes primary. Other accepted responders remain available as secondary support. Distances use the Haversine formula and ETA is a straight-line estimate for this MVP; road routing and live traffic are not used.

## System architecture

```text
 Bystander / Responder / Admin
	      |
	      v
       React + Vite frontend
       Maps, forms, live status
	      |
       REST API + Socket.IO
	      |
	      v
      Express application server
      Matching, uploads, workflows
	      |
	      v
	 SQLite database
```

The Vite development server proxies `/api` and `/socket.io` requests to the Express server. Socket.IO rooms limit emergency updates to the relevant bystander and matched responder clients.

## Technology

- React 19 and Vite
- Express 5 on Node.js
- SQLite through `better-sqlite3`
- Socket.IO and `socket.io-client`
- Leaflet and React Leaflet
- Tailwind CSS with the Vite plugin
- `lucide-react` for interface icons

## Getting started

### Requirements

- Node.js 18 or newer
- npm
- A browser that supports camera and geolocation access

### Install and run

```bash
npm install
npm run dev
```

To use custom server settings, copy `.env.example` to `.env` and edit the values before starting the application:

```bash
copy .env.example .env
```

On macOS or Linux, use `cp .env.example .env` instead.

The Vite client is available at [http://localhost:5173](http://localhost:5173). The Express API runs at `http://localhost:3001` and is proxied by Vite for `/api` and `/socket.io` requests.

For camera and geolocation features, use `localhost` or an HTTPS deployment and allow the required browser permissions.

### Production build

```bash
npm run build
npm run preview
```

The preview command serves the built frontend only. A deployed environment must also run the Express server, provide persistent storage for the SQLite database and uploaded files, configure `CLIENT_ORIGIN`, and use HTTPS for camera and geolocation permissions.

## Configuration

The server loads optional values from a `.env` file:

| Variable | Default | Purpose |
| --- | --- | --- |
| `PORT` | `3001` | Express server port |
| `CLIENT_ORIGIN` | `http://localhost:5173` | Socket.IO CORS origin |
| `DATABASE_PATH` | `./data/emergency-response.db` | SQLite database path |
| `UPLOAD_DIR` | `./data/uploads` | Verification and scene photo storage |
| `PASSWORD_SALT` | development fallback | Salt used for prototype password hashing |

The `data/` directory is created automatically and should not be committed. Uploaded identity documents and scene photos contain potentially sensitive information and require suitable access controls in a real deployment.

## Validation

Run the available server-side workflow checks:

```bash
npm run test:step4
npm run test:step6
npm run test:step7
```

These checks cover responder selection and radius expansion, ambulance arrival persistence, responder visibility, emergency categories, first-aid guidance, and the accept/reject workflow.

| Command | Coverage |
| --- | --- |
| `npm run test:step4` | Responder selection, fresh locations, and search-radius expansion |
| `npm run test:step6` | Ambulance arrival persistence and responder visibility |
| `npm run test:step7` | Emergency categories, first-aid guidance, and accept/reject behavior |

Run linting with:

```bash
npm run lint
```

## Project structure

```text
src/
	App.jsx                 Client routing and shared page shell
	components/             Bystander, responder, admin, and map screens
	lib/                    Socket.IO client and first-aid guidance
	App.css, index.css      Interface styling
server/
	index.js                Express API, matching logic, uploads, and Socket.IO
	db.js                   SQLite setup and schema migrations
	test-step*.js           End-to-end server workflow checks
```

## API surface

The client uses the following main endpoints:

- `POST /api/emergencies` - create an emergency alert
- `GET /api/emergencies/:id` - retrieve current alert status
- `PATCH /api/emergencies/:id/accept` - accept an alert
- `PATCH /api/emergencies/:id/reject` - reject an alert
- `PATCH /api/emergencies/:id/ambulance-status` - record ambulance arrival status
- `POST /api/responders` - register a responder and upload verification document
- `GET /api/responders` - list responder records for the prototype consoles
- `PATCH /api/responders/:id/verification` - verify or reject a registration
- `PATCH /api/responders/:id/availability` - set responder availability
- `PATCH /api/responders/:id/location` - update responder location
- `GET /api/responder/emergencies` - retrieve matching alerts for a responder

### Example: create an emergency

```bash
curl -X POST http://localhost:3001/api/emergencies ^
	-H "Content-Type: application/json" ^
	-d "{\"emergency_type\":\"Accident\",\"description\":\"Two-vehicle collision\",\"latitude\":28.6139,\"longitude\":77.2090}"
```

The request requires a supported `emergency_type`, valid `latitude` and `longitude`, and an optional description of up to 240 characters. An optional JPEG or PNG scene photo can be included as a data URL in `scene_photo`.

### Example: accept an emergency

```bash
curl -X PATCH http://localhost:3001/api/emergencies/1/accept ^
	-H "Content-Type: application/json" ^
	-d "{\"responder_id\":2}"
```

The responder must be verified, available, have valid coordinates, and have a location updated within the last 15 minutes.

### Example: update ambulance status

```bash
curl -X PATCH http://localhost:3001/api/emergencies/1/ambulance-status ^
	-H "Content-Type: application/json" ^
	-d "{\"ambulance_arrived\":true}"
```

## Prototype boundaries

This repository is intended for demonstration and evaluation, not direct production emergency use. Before deployment, the system would need authenticated role-based access, protected admin routes, secure secret management, encrypted sensitive data, audit logging, stronger input and upload controls, official emergency-service integration, reliable routing/traffic ETAs, push notifications, monitoring, and a formal privacy and safety review.

The current admin console is a prototype UI without authentication. Responder selection can also be switched in the dashboard to support local demonstrations. ETA values are estimates based on straight-line distance and an assumed average urban speed; they are not live navigation times.

## Known limitations

- Authentication and authorization are not implemented for the prototype consoles.
- Uploaded identity documents and scene photos require stronger access controls in production.
- Location data is stored for matching and requires an explicit retention and privacy policy.
- The admin verification workflow is a demonstration and is not a formal identity-verification process.
- ETA values do not account for roads, traffic, weather, or responder movement.
- Browser permissions, network connectivity, and device GPS accuracy can affect the workflow.

## Future improvements

- Authentication and role-based access control for responders and administrators
- Stronger identity verification and audit trails
- Push notifications and mobile applications
- Official emergency-service integration
- Road-network routing and traffic-aware ETA
- Privacy-preserving location sharing and retention controls
- Better support for poor-connectivity environments
- Multi-language guidance and accessibility improvements
- Production monitoring, abuse prevention, and formal safety review

