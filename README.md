# Tester Guide - Reminders System

This guide is intended for a QA tester and explains how to set up the environment, how to run the system, and which users are available for testing.

## Prerequisites

| Component | Required Version | Check |
|---|---|---|
| .NET SDK | 10.0 or later | `dotnet --version` |
| Node.js | 20 or later (recent LTS recommended) | `node --version` |
| npm | Bundled with Node.js | `npm --version` |
| Angular CLI | 20.x (installed automatically via `devDependencies`, no global install needed) | - |

The project consists of two parts that must run **at the same time**:
- **RemindersApi** - Backend server, .NET 10 Minimal API (port 5197)
- **reminders-app** - Frontend application, Angular 20 (port 4200)

## How to Run

### 1. Run the server (Backend)

In a separate terminal:

```
cd RemindersApi
dotnet run
```

The server will start at: `http://localhost:5197`

> Note: the database is **In-Memory** - all data (including reminders you create) is wiped every time the server restarts.

### 2. Run the application (Frontend)

In another terminal (without closing the first one):

```
cd reminders-app
npm install
npm start
```

The application will open at: `http://localhost:4200`

> Important: start the Backend first and make sure it's running, since the Frontend is configured to talk to it only at `http://localhost:5197` (CORS is restricted to this address).

## Test Users

The database is seeded with two users by default:

| Username | Password | Role | Permissions |
|---|---|---|---|
| `admin` | `admin123` | Admin | View + create + edit reminders |
| `viewer` | `viewer123` | Viewer | View only (cannot create or edit) |

Use the login screen in the app with one of these users. It's recommended to test with both users to confirm that the `viewer` role's read-only restriction is actually enforced (the server rejects it even if the UI doesn't block it).

## Reminder Status Lifecycle

After a reminder is created, its status changes automatically:

1. **Pending** - immediately after creation.
2. **Running** - a background service on the server checks every 30 seconds for pending reminders and moves them to this state. This stage lasts about 10 seconds.
3. **Success / Failed** - at the end of the 10 seconds, the server "flips a coin" (random 50/50) and sets the final status. There is currently no real business logic behind the failure - it's a simulation.

The Frontend automatically refreshes the list every 3 seconds, so you can watch all status stages in real time without manually refreshing the page.

## Things Worth Testing

- Create a reminder with the `admin` user and follow all status stages (Pending → Running → Success/Failed).
- Try to create/edit a reminder with the `viewer` user - it should fail (403/error message).
- Edit an existing reminder with `admin`.
- Different frequencies (`Once`, `Daily`, `Weekly`, `Monthly`) and the `FutureRunsCount` field.
- Behavior when the server isn't running (a friendly error message in the app).
- Refreshing the page/closing the browser - whether you need to log in again (depends on how the token is stored).
