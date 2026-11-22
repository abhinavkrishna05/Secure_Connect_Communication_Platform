# Secure Connect & Communication Platform

> A backend platform that combines a JWT-secured REST API with real-time communication: user authentication, role-based access control, a private Notes API, live notifications and room-based chat.

![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.x-000000?style=flat-square&logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white)
![Socket.IO](https://img.shields.io/badge/Socket.IO-4.x-010101?style=flat-square&logo=socketdotio&logoColor=white)
![JWT](https://img.shields.io/badge/Auth-JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white)
![Course](https://img.shields.io/badge/Course-Web%20Dev%20III-blue?style=flat-square)

**Repository:** [github.com/abhinavkrishna05/&lt;repository-name&gt;](https://github.com/abhinavkrishna05)

---

## Table of Contents

1. [About the Project](#about-the-project)
2. [Features](#features)
3. [Tech Stack](#tech-stack)
4. [Project Structure](#project-structure)
5. [REST API Reference](#rest-api-reference)
6. [Real-Time Events](#real-time-events)
7. [Getting Started](#getting-started)
8. [Testing Guide](#testing-guide)
9. [Security Notes](#security-notes)
10. [Learning Outcomes](#learning-outcomes)
11. [Author](#author)

---

## About the Project

This project was built for **Web Development III (Unit 4, Lab Assignment 4)**. It shows how a backend can protect its resources with token-based authentication while still supporting bidirectional, event-driven communication between connected clients.

Users can sign up, log in, access protected routes, manage their own notes, and chat in real time inside Socket.IO rooms. Internal actions such as signup, login, note creation and chat messages are also handled through Node.js `EventEmitter`.

## Features

| Module | Highlights |
|:---|:---|
| **Authentication** | User signup, password hashing, login that returns a JWT |
| **Authorization** | Protected routes and role-based access control (RBAC) for admin-only endpoints |
| **Notes API** | Create, list, fetch, update and delete notes, scoped to the authenticated user |
| **Live notifications** | `note:created` is broadcast to connected Socket.IO clients when a note is saved |
| **Room chat** | Join a room and exchange messages with everyone in that room |
| **Event-driven design** | `EventEmitter` handles `user:signup`, `user:login`, `note:created` and `chat:message` |
| **Configuration** | Secrets and connection details kept in `.env` through dotenv |

## Tech Stack

| Technology | Purpose |
|:---|:---|
| Node.js | Runtime |
| Express.js | Routing and middleware |
| MongoDB (official driver) | Data persistence |
| jsonwebtoken | Issuing and verifying JWTs |
| bcrypt | Password hashing |
| Socket.IO | Real-time communication |
| EventEmitter | Server-side event handling |
| dotenv | Environment variables |
| Postman | API and Socket.IO testing |

## Project Structure

```text
secure-connect-platform/
├── src/
│   ├── config/
│   │   └── database.js         # shared database handle
│   ├── middleware/
│   │   └── authMiddleware.js   # authenticateToken, requireRole
│   └── routes/
│       ├── authRoutes.js       # signup, login
│       └── notesRoutes.js      # notes CRUD
├── .env.example
├── .gitignore
├── package.json
└── server.js                   # Express + Socket.IO + MongoDB bootstrap
```

## REST API Reference

Base URL: `http://localhost:5000`

| Method | Endpoint | Auth | Description |
|:---:|:---|:---:|:---|
| `GET` | `/` | None | Health check |
| `POST` | `/api/auth/signup` | None | Register a new user |
| `POST` | `/api/auth/login` | None | Log in and receive a JWT |
| `GET` | `/api/protected` | JWT | Sample protected resource |
| `GET` | `/api/admin` | JWT + `admin` role | Admin-only resource |
| `POST` | `/api/notes` | JWT | Create a note |
| `GET` | `/api/notes` | JWT | List the user's notes |
| `GET` | `/api/notes/:id` | JWT | Get one note |
| `PUT` | `/api/notes/:id` | JWT | Update a note |
| `DELETE` | `/api/notes/:id` | JWT | Delete a note |

**Signup body**

```json
{
  "username": "abhinav",
  "email": "abhinav@example.com",
  "password": "your_password"
}
```

**Login body**

```json
{
  "email": "abhinav@example.com",
  "password": "your_password"
}
```

**Note body**

```json
{
  "title": "My Note",
  "content": "Note content goes here."
}
```

### Authentication flow

1. Register with `POST /api/auth/signup`.
2. Log in with `POST /api/auth/login` and copy the returned token.
3. In Postman, choose **Authorization → Bearer Token** and paste the token.
4. Call any protected endpoint. Requests without a valid token are rejected.

## Real-Time Events

The Socket.IO server listens on the same port as the REST API.

| Event | Direction | Description |
|:---|:---:|:---|
| `note:created` | Server → all clients | Sent with `{ userId, title }` after a note is created |
| `join_room` | Client → server | Join a room, e.g. `{ "room": "general" }` |
| `room_joined` | Server → client | Confirms the room was joined |
| `chat_message` | Client ⇄ server | Send a message; the server relays it to everyone in the room |
| `chat_error` | Server → client | Sent when `room`, `username` or `message` is missing |

**Sending a chat message**

```json
{
  "room": "general",
  "username": "Abhinav",
  "message": "Hello! Testing real-time chat."
}
```

The server adds a `sentAt` timestamp before broadcasting it to the room.

**Flow**

```text
Client  →  Socket.IO server  →  EventEmitter  →  Clients in the room
```

## Getting Started

### Prerequisites

- Node.js 18 or later
- A MongoDB instance (local or Atlas)
- Postman (REST and Socket.IO testing)

### Installation

```bash
git clone https://github.com/abhinavkrishna05/<repository-name>.git
cd <repository-name>
npm install
```

### Configuration

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

```env
PORT=5000
MONGODB_URI=your_mongodb_connection_string
DB_NAME=your_database_name
JWT_SECRET=your_long_random_secret
```

The server refuses to start if `MONGODB_URI`, `DB_NAME` or `JWT_SECRET` is missing.

### Run

```bash
npm start       # node server.js
npm run dev     # node --watch server.js (auto-restart)
```

You should see `Server listening on http://localhost:5000` in the terminal.

## Testing Guide

**REST (Postman)**

1. Signup → expect `201 Created`.
2. Login → expect `200 OK` and a token.
3. `GET /api/notes` without a token → expect an authorization error.
4. Add the Bearer token, then create, list, update and delete a note.
5. `GET /api/admin` with a normal user → expect `403 Forbidden`.

**Socket.IO (Postman or any Socket.IO client)**

1. Connect to `http://localhost:5000`.
2. Listen for `note:created`, then create a note over REST and watch the event arrive.
3. Emit `join_room` with `{ "room": "general" }`.
4. Emit `chat_message` from two connected clients and confirm both receive it.
5. Check the server terminal for the connection, room and event logs.

## Security Notes

- Passwords are hashed and never stored in plain text.
- Protected routes require a valid JWT; admin routes also check the user's role.
- Notes are tied to the authenticated user.
- Chat payloads are validated before they are broadcast.
- Secrets live in `.env`, which is excluded from version control. Never commit real credentials or your `JWT_SECRET`.
- The Socket.IO CORS policy is open (`*`) for development. Restrict it to trusted origins in production.

## Learning Outcomes

- Building REST APIs with Express.js
- Implementing authentication and authorization with JWT and RBAC
- Working with MongoDB collections and indexes
- Handling real-time events and rooms with Socket.IO
- Using `EventEmitter` for event-driven features
- Managing configuration securely with environment variables

## Author

**Abhinav Krishna**
GitHub: [@abhinavkrishna05](https://github.com/abhinavkrishna05)

*Web Development III | Lab Assignment 4*
