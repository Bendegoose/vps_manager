# VPS Manager

A full-stack web platform for managing remote VPS infrastructure, Docker containers, SSL certificates, and deployment workflows behind an authenticated dashboard.

Status: Work in progress. See the roadmap below.

## About The Project

VPS Manager is a self-hosted administration platform designed to centralize the operations of multiple remote Linux servers and their containerized workloads.

Instead of manually maintaining SSH sessions, writing Nginx reverse proxy configurations, and running par-command Certbot routines, the platform provides a unified web dashboard. It manages multi-server connections, streams live system telemetry, orchestrates Docker Compose projects, tracks Git repository status, automates Let's Encrypt certificates, and exposes an in-browser SSH terminal.

The project is built with a React frontend and a Node.js Express backend communicating over REST APIs and WebSockets, backed by a PostgreSQL database for credential storage and audit logs.

## Features

- JWT-based authentication with secure session handling
- Multi-server VPS registry with encrypted credential storage (AES-256-GCM)
- Live server telemetry (CPU, RAM, disk utilization) streamed via WebSockets
- Docker container and Docker Compose project lifecycle management (start, stop, restart, delete, rebuild)
- Automatic reverse proxy configuration and port detection for exposed services
- Automated SSL certificate provisioning using Certbot and Let's Encrypt
- SSL certificate management (tracking domains, expiry dates, and clean official revocation)
- Remote workspace Git drift detection (ahead, behind, diverged) and one-click pull
- Temporary public URL generation for container ports using Cloudflare Tunnels
- In-browser interactive SSH terminal powered by xterm.js and WebSockets
- Structured audit logging for deployment and server actions

## Built With

Backend: Node.js, Express

Frontend: React, Vite

Real-Time Communication: WebSocket (ws)

Database: PostgreSQL

Authentication: JSON Web Tokens (JWT), bcrypt

Terminal: xterm.js

Containerization: Docker, Docker Compose

## Prerequisites

For local development:

- Git
- Node.js (v18 or higher) and npm
- PostgreSQL (v14 or higher)
- Code editor such as Visual Studio Code

For target servers managed by the platform:

- Linux OS (Ubuntu 20.04/22.04/24.04 recommended)
- SSH server (OpenSSH)
- Docker and Docker Compose plugin

## How to Run

1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd vps-manager
```

2. Configure the database

Create a PostgreSQL database for the application:

```sql
CREATE DATABASE vps_manager;
```

3. Configure environment variables

In `server/.env`:

```env
PORT=5000
CLIENT_URL=http://localhost:5173
DATABASE_URL=postgresql://postgres:password@localhost:5432/vps_manager
JWT_SECRET=replace_with_a_secure_random_jwt_secret
REFRESH_TOKEN_SECRET=replace_with_a_secure_random_refresh_secret
ENCRYPTION_KEY=replace_with_a_32_byte_hex_key_for_aes256
```

In `client/.env`:

```env
VITE_API_URL=http://localhost:5000/api
VITE_WS_URL=ws://localhost:5000/ws
```

4. Install dependencies and start the backend

```bash
cd server
npm install
npm run migrate
npm run dev
```

The backend server runs on port 5000 by default.

5. Start the frontend

Open a separate terminal:

```bash
cd client
npm install
npm run dev
```

The Vite development server runs at `http://localhost:5173`.

## API

The backend exposes the following REST endpoints:

| Method | Path | Description |
| :--- | :--- | :--- |
| POST | /api/auth/login | Authenticate user and issue JWT tokens |
| POST | /api/auth/refresh | Refresh expired access token |
| POST | /api/auth/logout | Invalidate session |
| GET | /api/servers | List configured VPS instances |
| POST | /api/servers | Add a new VPS instance |
| PUT | /api/servers/:id | Update server configuration |
| DELETE | /api/servers/:id | Remove a server profile |
| GET | /api/servers/:id/containers | List Docker containers and projects |
| POST | /api/servers/:id/containers/action | Trigger container/project lifecycle action |
| GET | /api/servers/:id/ssl | Retrieve active SSL certificates |
| POST | /api/servers/:id/ssl/provision | Provision new Let's Encrypt certificate |
| DELETE | /api/servers/:id/ssl/:certName | Revoke and delete SSL certificate |
| POST | /api/servers/:id/git/pull | Trigger git pull on project root |

WebSocket channels:

| Path | Protocol | Purpose |
| :--- | :--- | :--- |
| /ws/telemetry/:serverId | WSS | Stream real-time CPU, RAM, and Disk metrics |
| /ws/terminal/:serverId | WSS | Bi-directional interactive SSH terminal session |
| /ws/logs/:actionId | WSS | Stream deployment and command execution logs |

## Configuration

Application settings are configured via environment variables.

| Variable | Component | Purpose |
| :--- | :--- | :--- |
| PORT | Backend | HTTP server listening port |
| CLIENT_URL | Backend | Allowed CORS origin for frontend client |
| DATABASE_URL | Backend | PostgreSQL connection string |
| JWT_SECRET | Backend | Secret used to sign authentication access tokens |
| REFRESH_TOKEN_SECRET | Backend | Secret used to sign session refresh tokens |
| ENCRYPTION_KEY | Backend | 256-bit symmetric key for encrypting stored SSH credentials |
| VITE_API_URL | Frontend | Backend HTTP base URL |
| VITE_WS_URL | Frontend | Backend WebSocket base URL |

Never store production secrets or private keys in source control.

## Development Workflow

Each backlog item is developed on a separate branch and merged through a pull request.

1. Update the local main branch: `git pull origin main`
2. Create a feature branch: `git switch -c feature/ticket-name`
3. Implement and test changes locally.
4. Run code formatting and tests.
5. Commit using descriptive commit messages.
6. Push the branch and open a Pull Request.

Example:

```bash
git switch main
git pull origin main
git switch -c feature/auth-jwt-implementation

# Implement changes

git add .
git commit -m "feat(auth): implement JWT login and token refresh middleware"
git push -u origin feature/auth-jwt-implementation
```

## Project Structure

```text
vps-manager/
├── client/                     Frontend application (React + Vite)
│   ├── src/
│   │   ├── api/                API client and endpoint definitions
│   │   ├── components/         Reusable UI components
│   │   ├── context/            Auth and global application state
│   │   ├── hooks/              WebSocket and telemetry hooks
│   │   ├── pages/              Dashboard, Server View, Terminal, Login
│   │   ├── App.jsx             Root component and routing
│   │   └── main.jsx            Application entry point
│   ├── index.html
│   └── package.json
│
├── server/                     Backend application (Express + WebSockets)
│   ├── src/
│   │   ├── controllers/        Route handlers (auth, servers, docker, ssl)
│   │   ├── middleware/         JWT verification, error handling
│   │   ├── models/             Database queries and schemas
│   │   ├── routes/             Express route definitions
│   │   ├── services/           SSH client, Docker runner, Certbot engine
│   │   ├── websocket/          WebSocket terminal and metric handlers
│   │   └── index.js            Server bootstrap
│   └── package.json
│
├── .gitignore
└── README.md
```

## Roadmap

### Epic 1 — Authentication and Server Registry
- [ ] User authentication with JWT and bcrypt
- [ ] PostgreSQL schema for users and servers
- [ ] AES-256-GCM encryption for stored SSH credentials
- [ ] CRUD endpoints for VPS server profiles

### Epic 2 — SSH Bridge and Real-Time Telemetry
- [ ] SSH connection pool with keepalive
- [ ] Periodic metric collection (CPU, memory, disk usage)
- [ ] WebSocket streaming server for telemetry data
- [ ] React dashboard metric widgets

### Epic 3 — Docker Orchestration
- [ ] Container discovery and Compose project grouping
- [ ] Container lifecycle actions (start, stop, restart, delete)
- [ ] Streaming project rebuilds (`docker compose up -d --build`)
- [ ] Port forwarding and public service link detection

### Epic 4 — SSL and Proxy Automation
- [ ] Nginx container inspection and mount resolution
- [ ] Automated HTTP ACME challenge configuration
- [ ] Certbot certificate issuance pipeline
- [ ] HTTPS server block injection and zero-downtime Nginx reload
- [ ] Certificate inventory with expiration tracking and safe deletion

### Epic 5 — Git Sync and Cloudflare Tunnels
- [ ] Workspace Git repository detection
- [ ] Remote branch tracking and drift comparison
- [ ] One-click git pull with streaming log output
- [ ] Ephemeral Cloudflare Tunnel integration for container demo links

### Epic 6 — Browser Terminal and Hardening
- [ ] xterm.js integration with WebSocket terminal handler
- [ ] ANSI escape code parsing and terminal resizing support
- [ ] Role-based access control (Admin, Operator)
- [ ] Audit log persistence in PostgreSQL

## Security Notes

- Do not commit `.env` files, database credentials, or private SSH keys.
- Server connection passwords and private keys must be encrypted at rest using AES-256-GCM.
- Use HttpOnly, Secure, and SameSite cookies for storing session tokens to mitigate XSS vulnerabilities.
- Enforce strict parameterization on remote shell executions to prevent command injection.
- Restrict inbound ports on target VPS instances to SSH (22), HTTP (80), and HTTPS (443).
- Avoid running backend Node.js processes as the root user.

## License

This project is developed as a personal portfolio project. No license has been selected yet. Unless a license is added, the project is not explicitly licensed for redistribution or reuse.
