# lab-web

The student-facing web app for the **Remote Robotics Learning Lab**, part of an EPICS in IEEE project with the IEEE Chicago Section.

High school students sign in from a school Chromebook, write Python code for a real robot, press **Run**, and watch the robot move on a live camera feed. Their code's output is shown alongside the video.

---

## Where this fits

```
 ┌────────────┐     HTTPS + WebSocket     ┌──────────────┐    Station API    ┌──────────────┐
 │  lab-web   │ ────────────────────────▶ │  lab-server  │ ────────────────▶ │ lab-station  │ ──▶ Robot
 │ (browser)  │ ◀──── output, status ──── │  (backend)   │ ◀──── events ──── │ (per robot)  │
 └────────────┘                           └──────────────┘                   └──────────────┘
       ▲                                                                            │
       └──────────────────────── live video (WebRTC) ───────────────────────────────┘
```

- This app runs **in the student's browser**. It's served as static files from the lab server.
- It talks **only to `lab-server`**. It never talks to robots or stations directly.
- Students write code using the functions from [`lab-sdk`](https://github.com/ieee-chicago-epics/lab-sdk).

---

## What to build

### Milestone 1: first full loop

| # | Feature | Details |
|---|---|---|
| 1 | **Sign-in page** | "Sign in with Google" button. Redirects to the backend's sign-in flow. |
| 2 | **Student page layout** | Four areas: code editor, live video, output panel, session bar (timer + status). |
| 3 | **Code editor** | Monaco Editor (the editor used in VS Code) with Python syntax highlighting and a starter template. |
| 4 | **Run / Stop buttons** | Run sends the code to the backend; Stop cancels a running program. Disable buttons when they can't be used. |
| 5 | **Output panel** | Shows `print` output and errors live, as they arrive over the WebSocket. Clear it on each new run. |
| 6 | **Validation errors** | If the backend rejects code (syntax error, blocked import), show the message clearly, with the line number. |
| 7 | **Live video panel** | Plays the station's camera stream (WebRTC). Use a test stream until real cameras exist. |
| 8 | **Session bar** | Countdown timer for the session, station name and station status (Available, Running, Resetting, Needs attention). |
| 9 | **Reset robot button** | Asks the backend to send the robot back to its home position. |
| 10 | **Session ended screen** | When time runs out: stop editing, show "Session ended", offer to download the code. |

### Later

- SDK help panel and autocomplete for SDK functions
- Sensor dashboard (charts of joint angles, distances and similar readings)
- Booking calendar for time slots across schools
- Teacher view (class list, student progress)
- Admin view (station health, remote reset and power-cycle)
- Saved projects and run history

---

## Tech stack

| Tech | Used for |
|---|---|
| **React + TypeScript** | The UI |
| **Vite** | Dev server and build tool |
| **Monaco Editor** (`@monaco-editor/react`) | Code editor |
| **WebSocket** (browser built-in) | Live output, status and timer updates from the backend |
| **WebRTC / WHEP player** | Live video from the lab's video server (MediaMTX) |
| **CSS modules or Tailwind** | Styling (decide as a team) |

Must work well on a **Chromebook screen (1366×768)** in Chrome. No installs on the student's machine.

---

## Talking to the backend

The full API is defined in the [`lab-server`](https://github.com/ieee-chicago-epics/lab-server) README. This app uses:

| Call | Purpose |
|---|---|
| `GET /auth/login` | Start Google sign-in |
| `GET /me` | Who is signed in |
| `POST /sessions` | Join a station; returns station, session end time, video URL |
| `POST /sessions/{id}/runs` | Submit code to run |
| `POST /runs/{id}/stop` | Stop a running program |
| `POST /sessions/{id}/reset-robot` | Send the robot home |
| `WS /sessions/{id}/stream` | Live events: output, errors, run started/ended, station status, timer |

Until the backend is ready, build against a **fake API** (mock responses in the frontend) that returns the same shapes.

---

## Suggested structure

```
lab-web/
├── src/
│   ├── pages/          SignIn, StudentPage, SessionEnded
│   ├── components/     CodeEditor, OutputPanel, VideoPanel, SessionBar, RunControls
│   ├── api/            REST calls + WebSocket client (and a mock version)
│   ├── types/          Shared TypeScript types matching the backend API
│   └── main.tsx
├── public/
├── package.json
└── README.md
```

---

## Getting started

### Prerequisites

- **Node.js LTS** (version 22 or newer; Vite requires `^20.19.0` or `>=22.12.0`). Download it from [nodejs.org](https://nodejs.org/) and check with `node -v`.
- **Git**

### Set up and run

```bash
git clone https://github.com/ieee-chicago-epics/lab-web.git
cd lab-web
npm install
npm run dev
```

`npm run dev` starts the Vite dev server and prints a local address (usually <http://localhost:5173>). Open it in Chrome; you should see the placeholder home page. The page reloads automatically when you edit a file.

### Other commands

| Command | What it does |
|---|---|
| `npm run build` | Type-checks and builds the app into `dist/` |
| `npm run preview` | Serves the built `dist/` locally |
| `npm run lint` | Runs the linter |

---

## Ground rules

- All changes go through a **pull request** and are reviewed before merging.
- Keep PRs small: one feature or fix each.
- **Never commit secrets** (API keys, passwords, tokens). This repo is public.
- If the backend API needs to change, agree on it first and update the `lab-server` README.
