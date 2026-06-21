# 🚖 Ride Booking Platform

A full-stack ride-booking app inspired by Uber/Ola — riders request a ride, nearby captains (drivers) get notified in real time, and both sides track the ride through to completion. Built with a Node.js/Express/MongoDB backend and a React/Vite frontend.

## Overview

This is a two-part project:

- **Backend** — an Express + MongoDB API handling user and captain (driver) accounts, JWT authentication, ride creation/lifecycle, geocoding/distance lookups, and a Socket.IO layer for real-time ride and location updates.
- **Frontend** — a React + Vite app (Tailwind + shadcn/ui components) with pages for the landing screen, user auth, and captain auth.

Geocoding and address autocomplete are powered by OpenStreetMap's Nominatim (no API key needed), and route distance/duration come from OpenRouteService (free API key required).

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, Tailwind CSS, shadcn/ui (Radix), React Router, Zustand, React Hook Form + Zod, React Leaflet, Axios |
| Backend | Node.js, Express 4 (ESM) |
| Database | MongoDB, Mongoose |
| Auth | JWT (cookie or Bearer token), bcrypt password hashing |
| Real-time | Socket.IO |
| Geocoding / Routing | OpenStreetMap Nominatim, OpenRouteService |
| Validation | express-validator (backend), Zod (frontend) |

## Project Structure

```
Ride-Booking-Platform/
├── Backend/
│   ├── server.js              # Entry point — HTTP server + Socket.IO init
│   ├── app.js                 # Express app, middleware, route mounting
│   ├── socket.js               # Socket.IO connection handling
│   ├── config/
│   │   └── db.config.js        # MongoDB connection
│   ├── models/                 # user, captain, ride, blacklisted-token schemas
│   ├── controllers/             # Request handlers for users, captains, rides, maps
│   ├── services/                # Business logic (geocoding, ride lifecycle, etc.)
│   ├── routes/                  # /users, /captain, /rides, /maps route definitions
│   ├── middleware/
│   │   └── auth.middleware.js    # authUser / authCaptain JWT guards
│   └── validators/               # express-validator rule sets
└── Frontend/
    ├── src/
    │   ├── App.jsx                # Route definitions
    │   ├── pages/                 # Home, Login, Register, UserLogin, UserRegister, CaptainLogin, CaptainRegister
    │   ├── features/
    │   │   ├── user/               # User auth forms + context
    │   │   ├── captain/             # Captain auth forms
    │   │   ├── home/                # Hero section, suggestions
    │   │   └── global/               # Navbar, footer, layout
    │   └── components/ui/             # shadcn/ui component library
    └── vite.config.js
```

## Prerequisites

- Node.js 18+
- A MongoDB instance (local or [MongoDB Atlas](https://www.mongodb.com/atlas))
- A free [OpenRouteService API key](https://openrouteservice.org/dev/#/signup) (needed for the distance/time endpoint)

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/thor-51/Ride-Booking-Platform.git
cd Ride-Booking-Platform
```

### 2. Set up the backend

```bash
cd Backend
npm install
```

Create a `.env` file inside `Backend/`:

```env
PORT=5001
MONGO_URI=mongodb://localhost:27017/ride-booking
JWT_SECRET=replace-this-with-a-long-random-string
ORS_API_KEY=your-openrouteservice-api-key
```

Start the backend (it runs on `nodemon` via the `start` script):

```bash
npm start
```

You should see:

```
Database connected successfully
Server is running on port 5001
```

### 3. Set up the frontend

In a separate terminal:

```bash
cd Frontend
npm install
npm run dev
```

Vite will print the local dev URL (typically `http://localhost:5173`).

> **Note:** The frontend currently calls the backend at a hardcoded `http://localhost:5001` (see `src/features/user/components/user-register-form.jsx`) rather than reading it from an environment variable. If you run the backend on a different port, update that URL accordingly.

## API Reference

All routes are prefixed with `/api/v1`. Routes marked **(auth)** require a valid JWT, sent either as an httpOnly `token` cookie or an `Authorization: Bearer <token>` header.

### Users — `/api/v1/users`

| Method | Route | Description |
|---|---|---|
| POST | `/register` | Register a new rider. Body: `{ fullname: { firstname, lastname }, email, password }` |
| POST | `/login` | Log in a rider. Body: `{ email, password }`. Sets a `token` cookie and returns the JWT. |
| GET | `/profile` **(auth)** | Returns the logged-in rider's profile. |
| GET | `/logout` **(auth)** | Clears the auth cookie. |

### Captains — `/api/v1/captain`

| Method | Route | Description |
|---|---|---|
| POST | `/register` | Register a new captain. Body: `{ fullname, email, password, vehicles: { color, plate, capacity, vehicleType } }`. `vehicleType` must be `car`, `motorcycle`, or `auto`. |
| POST | `/login` | Log in a captain. Body: `{ email, password }`. |
| GET | `/profile` **(auth)** | Returns the logged-in captain's profile. |
| GET | `/logout` **(auth)** | Clears the auth cookie. |

### Maps — `/api/v1/maps`

| Method | Route | Description |
|---|---|---|
| GET | `/get-coordinates` **(auth)** | `?address=` → `{ lat, lng }` via Nominatim. |
| GET | `/get-distance-time` **(auth)** | `?origin=&destination=` → distance (km) and duration (min) via OpenRouteService. |
| GET | `/get-suggestions` **(auth)** | `?input=` → address autocomplete suggestions via Nominatim. |

### Rides — `/api/v1/rides`

| Method | Route | Description |
|---|---|---|
| POST | `/create` **(auth, rider)** | Body: `{ pickup, destination, vehicleType }`. Creates a ride and notifies nearby captains (within 2 km of pickup) over Socket.IO. |
| GET | `/get-fare` **(auth, rider)** | Body: `{ pickup, destination }`. Returns the estimated fare. |
| POST | `/confirm` **(auth, captain)** | Body: `{ rideId }`. Captain accepts a ride. |
| GET | `/start-ride` **(auth, captain)** | Query: `rideId`, `otp`. Starts the ride after OTP verification. |
| POST | `/end-ride` **(auth, captain)** | Body: `{ rideId }`. Marks the ride complete. |

## Real-Time Events (Socket.IO)

| Event | Direction | Payload | Purpose |
|---|---|---|---|
| `join` | Client → Server | `{ userId, userType }` | Registers a socket ID against a user or captain so the server can message them directly. |
| `update-location-captain` | Client → Server | `{ userId, location: { ltd, lng } }` | Updates a captain's live location. |
| `new-ride` | Server → Client | Ride object | Sent to nearby captains when a ride is created, and to the rider when a captain confirms. |
| `ride-started` | Server → Client | Ride object | Sent to the rider when the captain starts the ride. |
| `ride-ended` | Server → Client | Ride object | Sent to the rider when the captain ends the ride. |

## Current State

This is an actively-developed project, and the frontend and backend aren't fully wired together yet:

- ✅ Backend: full auth, ride lifecycle, geocoding, and Socket.IO are implemented and functional.
- ✅ Frontend: user registration form is connected to the backend (`POST /api/v1/users/register`).
- 🚧 Frontend: the login forms (user and captain) and the captain registration form are UI-complete (with loading states) but not yet wired to their corresponding API calls.
- 🚧 The map/suggestions UI on the home page is currently a placeholder component.

## Known Limitations

- The frontend's backend URL is hardcoded rather than read from an environment variable.
- No `.env.example` is committed — the variable names above (`PORT`, `MONGO_URI`, `JWT_SECRET`, `ORS_API_KEY`) come from reading the backend source directly.
- No automated tests are included yet.
- No `LICENSE` file is currently included in the repository.

## Roadmap

- Wire up the remaining auth forms (user login, captain login, captain register) to the backend.
- Replace the hardcoded API base URL with an environment variable (e.g. `VITE_API_URL`).
- Build out the live map/location-tracking UI using `react-leaflet`.
- Add a captain dashboard for managing incoming ride requests.
- Add fare/payment integration (the `ride` model already has `paymentId`, `orderId`, and `signature` fields reserved for this).
- Add automated tests for the API routes and core ride-matching logic.
- Add a `.env.example` and a `LICENSE` file.

## Contributing

Contributions are welcome:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m "Add your feature"`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a pull request

## License

No `LICENSE` file is currently included in this repository. The backend's `package.json` specifies **ISC**; the frontend's `package.json` is marked `"private": true`.

## Author

Built by **Aryan Vatsal** ([@thor-51](https://github.com/thor-51))
