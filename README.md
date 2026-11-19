# Beauty Contest (Web) — PHP version

A real-time, web-based implementation of the **Beauty Contest** game (guess 2/3 of the average). Play solo against bots or multiplayer with friends in a room.

This branch is a functionally equivalent port of the original Python/FastAPI backend to **PHP** so it can be deployed as a PHP project, while keeping the same React frontend and gameplay behavior.

## How to Play

- Pick a number from **0–100** each round.
- The **target** is the average of all active picks × **0.8**. The closest valid pick wins the round.
- Everyone starts at **0 points**. Round losers lose **1 point**; reaching **-10** eliminates a player.
- The **last player standing** wins the match.

**Special rules (by active player count):**
- **≤ 4 players:** duplicate picks are invalid for that round.
- **≤ 3 players:** an exact hit on the rounded target is a *Critical Hit* (losers lose **2** points).
- **2 players:** if the picks are 0 and 100, the player who picked **100** wins the round.

## Features

- **Single player:** play locally against 1–9 AI opponents.
- **Multiplayer rooms:** create/join rooms via room code over WebSockets, with host-controlled rounds.
- **Session persistence:** multiplayer sessions can rejoin after a refresh.
- **Round history & scoreboard:** per-round picks, averages/targets, logs, and live scores.
- **Light/dark mode** and an in-app "How to Play" guide on the main screen.

## Project Structure

- `backend_php/` — PHP WebSocket server (Ratchet/ReactPHP) and game logic.
  - `server.php` — WebSocket server, message handling, and timers (pick timeout / ready auto-next).
  - `room_manager.php` — rooms, sessions, round flow, and state snapshots.
  - `game_logic.php` — core Beauty Contest rules and scoring.
  - `config.php` — game constants.
  - `composer.json` — PHP dependencies.
- `backend/` — original Python backend (kept for reference).
- `frontend/` — React (Vite) + Tailwind CSS client (`src/App.jsx`, `src/gameLogic.js`).
- `index.php` — PHP front controller to serve the built SPA from `frontend/dist`.
- `.htaccess` — rewrite rules for SPA routing on Apache/PHP hosts.
- `run_php.php` — convenience script to start PHP WS backend + PHP static frontend server.
- `run.py` — original Python/Node convenience script.

## Run Locally (PHP)

**Prerequisites:** PHP 8.1+ with Composer, and Node.js/npm (to build the frontend assets).

### 1. Install PHP backend dependencies

```bash
cd backend_php
composer install
```

### 2. Build the frontend

```bash
cd frontend
npm install --include=dev
npm run build
```

For local use, the frontend defaults to `ws://localhost:8000/ws` when running on localhost. For deployed builds, set `VITE_WS_URL` to your PHP WebSocket endpoint before building (e.g. `wss://<your-ws-host>/ws`):

```bash
# example
VITE_WS_URL=wss://your-ws-host/ws npm run build
```

### 3. Start the PHP servers

From the project root:

```bash
php run_php.php
```

- WebSocket backend: `ws://localhost:8000` (accepts `/ws`, same protocol as original)
- Frontend (built SPA): http://localhost:5173

Open the frontend URL, choose Single Player or Multiplayer, and start playing. Press `Ctrl+C` to stop.

## Deployment (PHP)

- **Frontend/static SPA:** deploy the project on a PHP-capable web server with the document root at the project root (or route all non-file requests to `index.php`). `index.php` serves files from `frontend/dist` with SPA fallback. Ensure `frontend/dist` is built with the correct `VITE_WS_URL`.
- **WebSocket backend:** run the Ratchet server as a long-running PHP process on a host that supports persistent processes (VPS/container/PaaS with worker support):

  ```bash
  cd backend_php && composer install
  php server.php
  ```

  It listens on port `8000` by default, or the `$PORT` environment variable if provided.

- The WebSocket protocol and game behavior (rules, scoring, timers, history, session rejoin, etc.) match the original implementation. Single-player mode in the UI runs locally in the browser (same as original) and does not require the backend.