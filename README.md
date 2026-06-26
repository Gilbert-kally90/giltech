# CSRP — Collaborative STEM Reasoning Platform

A full-stack web application for live, collaborative STEM reasoning in CBC classrooms.

## Architecture

```
csrp/
├── backend/          Node.js + Express + WebSockets
│   ├── src/
│   │   ├── db/       lowdb JSON database + seed script
│   │   ├── routes/   REST API (auth, sessions)
│   │   ├── middleware/  JWT auth
│   │   └── websocket.js  Real-time chat & events
│   └── data/         db.json (auto-created on seed)
│
└── frontend/         React + Vite
    └── src/
        ├── pages/    LoginPage, StudentHome, StudentSession, TeacherDashboard
        ├── components/  QuestionCard, ResultCard, GroupChat
        ├── hooks/    useAuth, useWebSocket
        └── lib/      api.js (REST client)
```

## Quick start

### 1. Install dependencies

```bash
cd backend && npm install
cd ../frontend && npm install
```

### 2. Seed the database

```bash
cd backend && node src/db/seed.js
```

Output:
```
✅ Database seeded!

Teacher login:  teacher@csrp.dev / teacher123
Student logins: amara@csrp.dev   / student123
                kofi@csrp.dev    / student123
                zara@csrp.dev    / student123
                jomo@csrp.dev    / student123

Class join code: BIO-2024
Active session:  Photosynthesis & Cell Respiration
```

### 3. Start the backend

```bash
cd backend && node src/index.js
# → http://localhost:3001
# → ws://localhost:3001/ws
```

### 4. Start the frontend (new terminal)

```bash
cd frontend && npx vite
# → http://localhost:5173
```

## API reference

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/login` | – | Login, returns JWT |
| GET | `/api/auth/me` | JWT | Current user |
| GET | `/api/sessions/join/:code` | JWT | Join by class code |
| GET | `/api/sessions/:id/current` | JWT | Current question state |
| POST | `/api/sessions/:id/submit` | JWT | Submit reasoning + answer |
| GET | `/api/sessions/:id/leaderboard` | JWT | Live scores |
| GET | `/api/sessions/:id/chat/:qid` | JWT | Group chat history |

## WebSocket events

Connect: `ws://localhost:3001/ws?sessionId=<id>&token=<jwt>`

| Type | Direction | Description |
|------|-----------|-------------|
| `connected` | server→client | Confirmed connection |
| `chat` | client→server | Send a message `{ type, text, questionId }` |
| `chat` | server→client | Receive a message `{ type, message }` |
| `presence` | server→client | Student joined/left group |
| `session_advance` | server→client | Teacher moved to next question |
| `ping/pong` | both | Keepalive |

## Student session flow

1. Login → `/student`
2. Enter class code `BIO-2024` → join session
3. **Phase 1**: Read question, type reasoning (≥20 chars required), discuss in group chat
4. **Phase 2**: Select answer from MCQ options
5. See result: correct/incorrect, score breakdown, correct answer revealed
6. Leaderboard updates live

## Scoring

| Action | Points |
|--------|--------|
| Correct answer | +8 |
| First correct in class | +2 bonus |
| Reasoning submitted (>20 chars) | +2 |

## Next steps

- [ ] Teacher dashboard: create sessions, upload questions, advance questions live
- [ ] Question bank: reusable questions across sessions
- [ ] AI reasoning evaluator (Anthropic API)
- [ ] CBC competency report export
- [ ] Mobile app (React Native / Flutter)
- [ ] PostgreSQL migration for production
