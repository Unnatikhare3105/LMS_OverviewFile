# EduCore LMS

**AI-powered Learning Management Platform** — layered backend architecture, real-time systems, and dual-mode (text + video) intelligent search.

[![Live Overview](https://img.shields.io/badge/overview-live-blue)](https://unnatikhare3105.github.io/LMS_OverviewFile/)
![Node.js](https://img.shields.io/badge/Node.js-Express-339933)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178C6)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248)
![Redis](https://img.shields.io/badge/Redis-caching-DC382D)
![Socket.io](https://img.shields.io/badge/Socket.io-realtime-010101)
![Next.js](https://img.shields.io/badge/Next.js-14-000000)

**[Full architecture & UI walkthrough →](https://unnatikhare3105.github.io/LMS_OverviewFile/)**

> Source code is private (contains environment credentials). This document describes the system design, API surface, and engineering decisions in detail. Happy to walk through the implementation live on request.

---

## Overview

EduCore is a full-stack LMS that combines structured course content with AI-generated assessment, real-time engagement features (live leaderboard, activity tracking), and unified search across text and video content types. Built as a production-oriented system rather than a CRUD tutorial project — the design choices below reflect that.

**Scale of the system:**

| | |
|---|---|
| API endpoints | 12+ |
| Core modules | 12+ (auth, search, quiz, bookmarks, leaderboard, analytics) |
| Content types unified under one search/bookmark model | 2 (text, video) |
| Real-time transport | WebSocket (Socket.io) |

---

## My Role

Designed and built the system end-to-end — API design, database schema, service-layer architecture, real-time infrastructure, and the AI integration for quiz generation. Solo-built; no boilerplate/starter template used.

---

## Tech Stack & Why

| Layer | Choice | Reasoning |
|---|---|---|
| Frontend | Next.js 14 (App Router), Redux Toolkit, TypeScript | SSR for initial load performance, predictable global state for leaderboard/session data |
| Backend | Node.js, Express, TypeScript | Type safety shared across the stack; async I/O fits a read-heavy, real-time workload |
| Database | MongoDB + Mongoose | Flexible schema for heterogeneous content (text/video) without repeated migrations |
| Cache | Redis | Reduces DB load for frequently-read data (leaderboard, session lookups) under concurrent access |
| Real-time | Socket.io | Event-driven leaderboard/notification updates without client polling |
| AI | Google GenAI | Contextual quiz generation instead of a static question bank |
| Auth | JWT + bcrypt | Stateless auth suited to horizontal scaling; hashed credentials, HTTP-only cookies |

---

## System Architecture

```
Client            Next.js App Router · Redux Store · TypeScript
      │  REST + WebSocket
API Layer         Express Router · Auth Middleware · Rate Limiter · Error Handler
      │
Service Layer     Quiz Service · AI Service · User Service · Search Service · Email Service
      │
Data Layer        MongoDB · Redis · Socket.io · Google GenAI
```

Layering follows **Repository → Service → Controller** separation:
- **Repositories** isolate all data access — swapping MongoDB for another store touches only this layer
- **Services** hold business logic, independent of HTTP concerns — directly unit-testable
- **Controllers** stay thin — request parsing and response shaping only

```
backend/src/
├── config/          # env, constants
├── db/              # connection handling
├── models/          # User, Quiz, Bookmark, Syllabus, DailyChallenge
├── repositories/     # data access layer
├── services/          # ai.service.ts, redis.service.ts, email.service.ts
├── controllers/        # route logic
├── routers/             # route definitions
├── middlewares/          # auth, rate limiting, error handling
└── utils/                 # logger, socket setup
```

---

## API Design

RESTful, consistently typed, uniform error-response schema across all routes.

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/user/register` | Create account | Public |
| POST | `/api/user/login` | Authenticate, issue JWT | Public |
| GET | `/api/user/profile` | Profile, stats, streaks | JWT |
| GET | `/api/user/activity` | Heatmap activity data | JWT |
| GET | `/api/syllabus/search` | Search across text + video | JWT |
| GET | `/api/syllabus/:id` | Topic detail & content | JWT |
| POST | `/api/quiz/generate` | AI-generate quiz for a topic | JWT |
| POST | `/api/quiz/submit` | Submit answers, update rank | JWT |
| GET | `/api/bookmarks` | List saved content | JWT |
| POST | `/api/bookmarks` | Add bookmark | JWT |
| DELETE | `/api/bookmarks/:id` | Remove bookmark | JWT |
| GET | `/api/daily-challenge` | Today's challenge | JWT |

---

## Engineering Decisions & Trade-offs

**Unified search across content types**
Text articles and video content share one search index and one bookmark model instead of two parallel systems. Cost: a slightly more complex schema up front. Benefit: no duplicated search/bookmark logic, and new content types can be added without a new subsystem.

**Redis for leaderboard reads, not writes**
Leaderboard reads are cached; writes (score submission) go straight to MongoDB and invalidate the relevant cache keys. Avoids stale-write bugs that a write-through cache would need extra handling for.

**AI-generated quizzes over a static question bank**
Removes the ongoing content-authoring bottleneck, at the cost of needing guardrails (prompt structure, response validation) to keep question quality consistent — handled in the AI service layer rather than the controller.

**JWT over session store**
Chosen for statelessness (no shared session store needed across instances), with the standard trade-off of needing short expiry + refresh flow for revocation, rather than instant server-side invalidation.

---

## What I'd Improve With More Time

- Move quiz-generation validation into a schema-enforced (Zod) response contract with the AI service, rather than manual checks
- Add integration tests around the search endpoint's ranking logic
- Introduce a message queue for email/notification dispatch instead of inline calls from the service layer

---

## Links

- **Live architecture & UI overview:** https://unnatikhare3105.github.io/LMS_OverviewFile/