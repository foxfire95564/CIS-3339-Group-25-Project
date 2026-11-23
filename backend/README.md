# Data Platform Backend

## Overview

This folder contains the **Node.js and Express REST API** for the CIS 3339 Data Platform. It supports authentication, clients, events, services, organization information, and event registration/deregistration.

The backend uses:

- Node.js and Express
- MongoDB Atlas for remotely hosted MongoDB storage
- Mongoose for database models and queries
- JSON Web Tokens (JWT) for authentication
- bcrypt for application password hashing
- CORS for frontend-origin restrictions
- Morgan for request logging

The backend enforces authentication, role-based authorization, and organization isolation independently of the frontend.

## Architecture

```text
Browser / Vue frontend (http://localhost:5173)
                  |
                  | REST API requests (Axios)
                  v
Node.js / Express backend (http://localhost:3000)
                  |
                  | Mongoose, using backend MONGO_URL
                  v
MongoDB Atlas (remote cloud database: cis3339_project)
```

The frontend does **not** connect directly to MongoDB Atlas. The frontend remains on port `5173` and the backend on port `3000` for local development and the tested local production preview. Atlas replaces the previous *local database*, not the Express server.

---

## Prerequisites

- Node.js and npm
- Git if cloning the repository
- Internet access to MongoDB Atlas
- Authorized network access to the course Atlas cluster
- Working course-only Atlas database credentials

**A local MongoDB server is no longer required** for the normal Atlas configuration. A local database may still be used as an optional fallback when `MONGO_URL` is changed accordingly.

## Installation

From the repository root, open PowerShell or a terminal:

```powershell
cd backend
npm install
```

---

## MongoDB Atlas — Remote Database

The project database is hosted in **MongoDB Atlas**. The course cluster was created for Sprint 3 so the database can be reached over an authorized internet connection.

### Database migration

The original local `cis3339_project` database was backed up with `mongodump` and restored into Atlas with `mongorestore`. The restore output confirmed **49 restored documents, 0 failures**:

| Collection | Documents |
|---|---:|
| clients | 16 |
| events | 15 |
| services | 15 |
| users | 2 |
| org | 1 |
| **Total** | **49** |

The original local database and backup were retained rather than deleted. These counts document the migration snapshot; they may change as users add or modify records.

### Atlas database access

An authorized developer or grader connecting from another computer needs:

1. The **Atlas connection string** and course-specific database account credentials.
2. A source IP permitted by the Atlas **Network Access / IP Access List**.
3. A database user with adequate permissions to read and, where required, initialize or update the `cis3339_project` database.

Atlas database credentials are separate from the application's `admin/admin` and `user/user` login credentials. A successful login through the locally running application confirms its connection to Atlas; it does **not** independently prove access from every external IP. Confirm the instructor's network is authorized before grading.

For the application, prefer a database user with `readWrite` privileges limited to the course database. A user with broader Atlas administration permissions should be reserved for database administration, not routine application access.

### Inspecting the database

MongoDB Compass may be used to connect directly to Atlas, inspect collections, and confirm the restored records. Compass is a desktop client; **Atlas hosts the database**.

---

## Environment Configuration

Create or verify `backend/.env`. This is an **example with placeholders**, not a working connection string:

```env
PORT=3000
MONGO_URL=mongodb+srv://<DB_USERNAME>:<DB_PASSWORD>@<ATLAS_CLUSTER_HOST>/cis3339_project?retryWrites=true&w=majority
ORG_ID=6ac98d9c1b9fe52f029f044f
JWT_SECRET=<GENERATE_A_LONG_UNIQUE_COURSE_ONLY_SECRET>
CORS_ORIGIN=http://localhost:5173

SEED_VIEWER_USERNAME=user
SEED_VIEWER_PASSWORD=user
SEED_EDITOR_USERNAME=admin
SEED_EDITOR_PASSWORD=admin
```

Replace all placeholders in the actual environment file. **Do not put credentials into this README or frontend source code.** If the Atlas database password includes URI-reserved characters, URL-encode the password in the connection URI. Keep `ORG_ID` consistent with the restored organization record.

### Environment variable reference

| Variable | Purpose |
|---|---|
| `PORT` | Local Express API port, normally `3000` |
| `MONGO_URL` | MongoDB Atlas connection URI, including database name |
| `ORG_ID` | Organization configured for this deployment; must match the seeded/restored organization |
| `JWT_SECRET` | Course-only secret used to sign and validate JWTs |
| `CORS_ORIGIN` | Permitted browser frontend origin; `http://localhost:5173` locally |
| `SEED_VIEWER_USERNAME` | Seed viewer username, `user` |
| `SEED_VIEWER_PASSWORD` | Seed viewer password, `user` |
| `SEED_EDITOR_USERNAME` | Seed editor username, `admin` |
| `SEED_EDITOR_PASSWORD` | Seed editor password, `admin` |

Application login passwords are **hashed with bcrypt in MongoDB**. By contrast, the Atlas database password must be provided as the actual database credential (URI-encoded if necessary), **not** as a bcrypt hash.

### Submission and credential safety

Sprint 3 requires the working frontend/backend environment configuration and MongoDB connection details for grading. Provide those details **through the authorized Canvas submission method**. Follow the instructor's separate repository instructions, but do not expose a real database password or JWT secret in a public GitHub repository. Use a dedicated course database and course-only credentials, and rotate or revoke grading credentials after the course.

---

## Database Initialization

Manual MongoDB collection creation is not required. With `MONGO_URL` pointing to an **empty, correctly authorized** course database, run:

```powershell
npm run db:init
```

The initializer creates or verifies:

- The organization matching `ORG_ID`
- Viewer account (`user` / `user`)
- Editor account (`admin` / `admin`)
- Representative clients, services, and events
- Sample event registrations

The script is designed to be **idempotent**. Running it again should not duplicate seed organizations, accounts, or records or delete unrelated application data:

```powershell
npm run db:init
npm run db:init
```

**Important for the migrated Atlas database:** the 49 original documents were already restored. You do not need to run `db:init` just to start the application. Use it to initialize a new empty course database or to verify seed requirements as appropriate. Do not wipe the existing Atlas database during normal setup.

---

## Demo Application Accounts

| Role | Username | Password | Access |
|---|---|---|---|
| Viewer | `user` | `user` | Authenticated read operations |
| Editor | `admin` | `admin` | Reads plus permitted mutations and registrations |

These are **application** accounts managed by the backend, not Atlas database usernames.

---

## Start the Backend

From the `backend` directory:

```powershell
npm start
```

The API should run at:

```text
http://localhost:3000
```

The development start script uses nodemon to restart on backend source changes. Keep the terminal open while testing the frontend. Restart the backend after changing `.env` values.

### Verify Atlas connectivity

Open:

```text
http://localhost:3000/health
```

Expected response when MongoDB is connected:

```json
{
  "status": "ok",
  "db": "connected"
}
```

`GET /health` is public. With `MONGO_URL` pointing to Atlas, this confirms the backend is connected to the configured remote database. To confirm access for an external grader, also test an independently authorized connection from another network or host.

---

## Backend API Authentication

The backend issues JWTs after successful login. Tokens contain information such as:

```text
username
role
orgId
```

Protected routes use reusable middleware:

```js
requireAuth
```

An absent, expired, or invalid JWT results in HTTP `401`. Failed logins use the same response for an unknown username and an incorrect password:

```json
{
  "error": "Invalid username or password"
}
```

## Role Authorization

Mutating operations require the editor role:

```js
requireRole('editor')
```

A viewer directly attempting an editor-only operation receives HTTP `403`:

```json
{
  "error": "Forbidden"
}
```

The backend enforces these permissions even if a user bypasses frontend route guards or manually calls the API.

### Route permissions

| Operation | Required access |
|---|---|
| Read clients, events, services | Valid authenticated user |
| Create, update, delete | Authenticated editor |
| Register or deregister clients for events | Authenticated editor |

---

## Organization Isolation

Client, event, and service queries are restricted to the organization contained in the authenticated JWT:

```js
req.user.orgId
```

A shared query helper applies the organization filter. The backend does not trust an organization identifier sent by the browser to decide which data the user may access. Cross-organization records must not be visible or modifiable through another organization's authenticated session.

Testing cross-organization access requires suitably prepared records and credentials from separate organizations; simply testing `403` for a viewer mutation is not the same test.

---

## Event Registration and Deregistration

Both operations require authentication, the editor role, and clients/events within the authenticated organization.

Registering a client already registered for the event returns HTTP `200` without a duplicate attendee:

```json
{
  "message": "Already registered"
}
```

Deregistering a client who is not currently registered returns HTTP `200`:

```json
{
  "message": "Not registered"
}
```

If the referenced client or event does not exist in the permitted organization, the request is treated as not found.

---

## Error Handling

Centralized Express error handling returns a consistent JSON shape:

```json
{
  "error": "Human readable message"
}
```

Expected HTTP status codes include:

| Status | Meaning |
|---|---|
| `400` | Invalid request or validation failure |
| `401` | Missing or invalid authentication |
| `403` | Insufficient role permissions |
| `404` | Missing or inaccessible resource |
| `500` | Unexpected server error |

Responses should not reveal raw database errors, stack traces, or secrets.

---

## CORS and Frontend Integration

The backend accepts browser API requests from the origin configured in:

```env
CORS_ORIGIN=http://localhost:5173
```

Frontend `.env` contains:

```env
VITE_ROOT_API=http://localhost:3000
```

The frontend still talks to the **local Express backend**; Express talks to **remote Atlas**. Do not put the Atlas connection string in `frontend/.env`.

Local Vite development and production preview both use `http://localhost:5173`, so no CORS change is needed when switching between them. Only one frontend process can occupy the port at a time. For a deployed frontend or backend, update the relevant origins/URLs and restart/rebuild as appropriate.

The frontend production-preview script is:

```json
"serve": "vite preview --port 5173 --strictPort"
```

---

## Reproduce the Full Application

These steps assume a clean clone and an **already provisioned** Atlas cluster with an authorized database user and an approved network source.

1. Clone the repository and open its root directory.
2. Configure `backend/.env` with a working **course-only Atlas** URI, the matching `ORG_ID`, JWT secret, and CORS origin.
3. Verify `frontend/.env` contains `VITE_ROOT_API=http://localhost:3000`.
4. Install backend dependencies and initialize an empty database if necessary:

   ```powershell
   cd backend
   npm install
   npm run db:init
   ```

5. Start the backend:

   ```powershell
   npm start
   ```

6. Confirm `http://localhost:3000/health` returns `"db": "connected"`.
7. Open a second terminal from the repository root, install and start the frontend:

   ```powershell
   cd frontend
   npm install
   npm run dev
   ```

8. Open `http://localhost:5173`, select Login, and test `user/user` and `admin/admin`.

**For an already restored Atlas database:** skip `npm run db:init` on ordinary restarts. To run the frontend production build instead, stop `npm run dev`, then execute in `frontend/`:

```powershell
npm run build
npm run serve
```

Open `http://localhost:5173` and test production workflows. This is a local production-build preview, **not** a publicly hosted Express or frontend deployment.

---

## Integration and Security Verification Checklist

- [ ] `GET /health` reports connected MongoDB Atlas database.
- [ ] MongoDB Compass can connect to Atlas using authorized credentials.
- [ ] A second authorized network/host can reach the course database for grading.
- [ ] `user/user` logs in and can perform permitted reads.
- [ ] `admin/admin` logs in and can perform editor operations.
- [ ] Viewer mutation through a direct API request returns `403`.
- [ ] Cross-organization access to another organization's record is denied.
- [ ] Client, event, and service CRUD workflows function as permitted.
- [ ] Duplicate event registration returns `Already registered` without adding another record.
- [ ] Repeat deregistration returns `Not registered` without crashing.
- [ ] Frontend communicates with the backend using `VITE_ROOT_API`.
- [ ] `npm run build` succeeds and `npm run serve` displays the working frontend.

This checklist describes tests to perform; it is not an assertion that every item has already been independently re-run after the Atlas migration.

---

## Troubleshooting

### MongoDB authentication fails

- Confirm `MONGO_URL` uses the correct Atlas database username and **current** password.
- URL-encode reserved characters in the password.
- Confirm the account has access to `cis3339_project`.
- Restart the backend after editing `.env`.

### Connection timeout or network error

- Confirm Atlas network access permits the IP address of the machine connecting to the database.
- Check internet connectivity and cluster status.
- Ensure the backend is reading the intended `.env` file.

### `/health` shows disconnected

Check the backend terminal for connection errors, verify Atlas credentials and network access, and restart the backend. Don't share credential-bearing logs publicly.

### Login works but application records do not load

Confirm `ORG_ID` matches the organization in Atlas and that the expected collections are present in the `cis3339_project` database.

### Browser reports CORS errors

Verify `CORS_ORIGIN=http://localhost:5173` matches the address where the frontend is actually running. Restart Express after changes.

---

## Known Limitations and Notes

- This application instance is configured for a single organization using `ORG_ID`, even though database-level organization isolation is supported.
- Atlas access is controlled by database-user permissions **and** the network IP access list.
- The Express server and Vite frontend run locally unless separately deployed to public hosting; a cloud database alone does not publicly host the application.
- The development and production preview frontend both use port `5173`.
- `npm run db:init` is intended to preserve existing data and avoid duplicate seed records.
- Demo application passwords are intentionally simple for course grading; they are not appropriate for production accounts.
- Any `.env` file used for grading must use dedicated course-only credentials; rotate them after grading.

For project-wide design details and frontend setup, see the repository root README and `frontend/README.md`.
