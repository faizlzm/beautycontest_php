# Beauty Contest - PHP Backend (WebSocket)

This is a PHP implementation of the Beauty Contest multiplayer backend, functionally equivalent to the original Python/FastAPI WebSocket server.

## Requirements

- PHP 8.1+
- Composer
- Extensions commonly enabled by default: `json`, `mbstring`, `openssl`, `sockets` (Ratchet uses ReactPHP sockets)

## Install

```bash
cd backend_php
composer install
```

## Run

```bash
php server.php
```

By default it listens on port `8000` (or `$PORT` env var) and accepts WebSocket connections on any path (frontend uses `/ws`, e.g. `ws://localhost:8000/ws`).

## Protocol

The server speaks the same JSON-over-WebSocket protocol as the original backend:
- Client -> server: `create_room`, `join_room`, `rejoin_room`, `start_game`, `submit`, `ready_next`, `next_round`, `leave`
- Server -> client: `joined`, `lobby_update`, `game_started`, `round_start`, `submission_update`, `round_result`, `ready_update`, `state_sync`, `session_invalid`, `error`

Game rules, timers (60s pick, 15s ready/auto-next), scoring, and history behavior are preserved.