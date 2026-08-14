---

# 📄 Esrahley (اشرحلي) — Project Foundation Document

## Phase 0 — Problem Statement

Freelancers struggle to find consistent, trustworthy micro-teaching/coaching work, and clients struggle to find affordable, subject-specific help (e.g., Chemistry, Programming) with real trust in the freelancer's ability — since YouTube lacks interaction, friends aren't sustainable, and general platforms like Upwork are too broad and expensive for this kind of need.

Esrahley solves this by letting clients post a **"Teach" or "Coach"** request in a specific subject. Freelancers respond with a proposal — optionally including a short video demonstrating their explanation skill — so clients can hire with confidence. Once hired, price is agreed between both parties, and they communicate via in-platform chat to complete the session/work. Payment is handled securely through the platform (Stripe test mode for v1).

**Out of scope for v1:** live video/calls, dispute resolution system, AI-based matching, multi-currency support.

---

## Phase 1a — Functional Requirements

**Actors:** Client, Freelancer (same account, two views), Admin

**Auth**
- Register / Login
- Duplicate-account check on signup
- Forgot password → email code/link → reset password

**Dashboard**
- Overview of projects, earnings, basic stats

**Client capabilities**
- Post a project: title, details, resources (files/PDFs/images), budget range, duration
- View list of applicants per project
- Chat with any freelancer who applied
- Accept a freelancer (project becomes active); can revert to pending and choose another if needed
- Mark project as finished → releases payment to freelancer (minus platform fee)
- Balance must be ≥ project budget before posting

**Freelancer capabilities**
- Browse open projects
- Apply to a project: proposal text, optional video, proposed price (validated against client's budget range), proposed duration
- Max 8 applications per 3 days
- View own applications
- Withdraw earnings

**Wallet / Payments**
- Balance top-up via Stripe (test mode for v1)
- Withdrawal (simplified for v1)
- Platform takes a percentage fee on completed projects

**Reviews**
- Rating + comment, bidirectional (client→freelancer and freelancer→client), after project completion

**Admin**
- Exists in v1; exact permissions to be defined later

---

## Phase 1b — Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Security** | Hashed passwords; users can only access their own data; basic rate-limiting on login/signup |
| **Performance** | Clean, efficient code; no premature scaling optimization |
| **Availability/Reliability** | Proper error handling and logging from the start |
| **Usability** | Consistent, clear API responses and error messages |
| **Scalability** | Modular, clean design; no scaling infrastructure investment yet |
| **Maintainability** | Layered architecture (Controller → Service → Repository), clean naming, production-quality coding habits |

**Philosophy:** Production-grade *practices*, not production-scale *infrastructure*.

---

## Phase 2: Actors & Use Cases

### Actors

- **Client** — posts teaching/coaching requests, hires freelancers
- **Freelancer** — applies to requests, delivers teaching/coaching
- **Admin** — manages and oversees the platform

*(Note: Client and Freelancer are two views of the same account, not separate account types.)*

---

### Actor: Client

- Register/Sign-up
- Login
- Forgot Password (reset via email code)
- Edit account
- View dashboard (projects, earnings, stats)
- Top-up wallet balance
- Post a project (title, details, budget range, duration)
- Attach resources to a project (files/PDFs/images)
- View list of applicants per project
- Chat with any freelancer who applied
- Accept a freelancer
- Revert to pending / choose another freelancer
- Mark project as finished
- Rate and review freelancer

---

### Actor: Freelancer

- Register/Sign-up
- Login
- Forgot Password (reset via email code)
- Edit account
- View dashboard (projects, earnings, stats)
- Browse open projects
- Apply to a project (proposal text, optional video, proposed price, proposed duration)
- View own applications (with status: pending / accepted / rejected)
- Withdraw/Cancel an application
- Chat with client (after applying)
- Withdraw earnings
- Rate and review client

---

### Actor: Admin

- User management — view/suspend/ban users who violate rules
- Content moderation — remove inappropriate projects/proposals/reviews if reported
- Financial oversight — view transactions, handle stuck/failed payments
- Platform monitoring — view basic stats (total users, total projects, revenue)
- View system logs / activity

---

## Phase 3: Domain Model

### Entities & Attributes

**User**
- id, name, email, password (hashed), phone (optional), profile_bio, created_at

**Project**
- id, client_id (→ User), title, description, budget_min, budget_max, duration, type (Teach/Coach), subject, status (open/in_progress/finished), created_at

**Proposal**
- id, project_id (→ Project), freelancer_id (→ User), proposal_text, video_url (optional), proposed_price, proposed_duration, status (pending/accepted/rejected), created_at

**Resource**
- id, project_id (→ Project), file_url, file_type, uploaded_at

**Wallet**
- id, user_id (→ User), balance, updated_at

**Transaction**
- id, wallet_id (→ Wallet), type (top-up/payment/withdrawal/fee), amount, related_project_id (optional), created_at

**Chat**
- id, project_id (→ Project), created_at

**Message**
- id, chat_id (→ Chat), sender_id (→ User), text, sent_at, read_status

**Rating_Review**
- id, project_id (→ Project), reviewer_id (→ User), reviewee_id (→ User), rating (1–5), comment, created_at

---

### Relationships

| Relationship | Cardinality | Meaning |
|---|---|---|
| User ↔ Project | 1:N | One user (as client) posts many projects |
| User ↔ Proposal | 1:N | One user (as freelancer) submits many proposals |
| Project ↔ Proposal | 1:N | One project receives many proposals |
| Project ↔ Resource | 1:N | One project can have many attached files |
| User ↔ Wallet | 1:1 | Each user has exactly one wallet |
| Wallet ↔ Transaction | 1:N | One wallet has many transaction records |
| Project ↔ Chat | 1:1 | Each project has one chat thread (once freelancer accepted) |
| Chat ↔ Message | 1:N | One chat contains many messages |
| User ↔ Message | 1:N | One user sends many messages (as sender) |
| Project ↔ Rating_Review | 1:N | One project can have reviews (up to 2 — one per direction) |
| User ↔ Rating_Review (as reviewer) | 1:N | One user writes many reviews |
| User ↔ Rating_Review (as reviewee) | 1:N | One user receives many reviews |

---

Good — trust noted, and I'll use it responsibly: I'll flag it clearly whenever something is genuinely better learned "later" vs. "now," like I just did.

## Phase 4: System Architecture

### Layered Architecture
```
Client (Postman / Frontend / Mobile)
        ↓
[Auth Filter — verifies JWT token, identifies the user]
        ↓
Controller Layer — receives requests, delegates, returns responses (no logic, no error handling)
        ↓
Service Layer — business rules, authorization checks (ownership), calls Repository and/or external APIs
        ↓
Repository Layer (JPA) ──→ Database
Service Layer also ──→ External API Clients (e.g., Stripe)
        ↑
[Global Exception Handler — catches thrown exceptions from anywhere, returns clean HTTP error responses]
```

### Key Architectural Decisions
- **Authentication**: JWT-based, verified by a security filter before requests reach Controllers. Login issues a signed token; every subsequent request carries it.
- **Authorization**: enforced manually in the Service layer (e.g., "does this wallet belong to the requesting user?") — separate concern from Authentication.
- **Error Handling**: centralized via a Global Exception Handler (`@RestControllerAdvice`) — Controllers never contain try/catch blocks.
- **Chat**: 
  - v1 (initial working version): simple REST polling (`GET /chats/{id}/messages`) — same Controller→Service→Repository pattern as everything else.
  - v2 (planned upgrade, learned as a dedicated step in Phase 9): real-time via WebSockets/STOMP, once core system is functional.
- **Payments (Stripe)**: Service layer coordinates between Repository (own DB: Wallet, Transaction) and an external Stripe API client (test mode for v1).

---

## Phase 5: Database Design

```sql
User
------------------------
id              BIGINT (PK, auto-increment)
name            VARCHAR(100)
email           VARCHAR(150)   UNIQUE
password        VARCHAR(255)      -- stored hashed
phone           VARCHAR(20)    NULL
profile_bio     TEXT           NULL
created_at      TIMESTAMP

Project
------------------------
id              BIGINT (PK, auto-increment)
client_id       BIGINT (FK → User.id)
title           VARCHAR(150)
description     TEXT
budget_min      DECIMAL(10,2)
budget_max      DECIMAL(10,2)
duration        VARCHAR(50)
type            VARCHAR(20)     -- 'TEACH' or 'COACH'
subject         VARCHAR(100)
status          VARCHAR(20)     -- 'OPEN' / 'IN_PROGRESS' / 'FINISHED'
created_at      TIMESTAMP

Proposal
------------------------
id                  BIGINT (PK, auto-increment)
project_id          BIGINT (FK → Project.id)
freelancer_id       BIGINT (FK → User.id)
proposal_text       TEXT
video_url           VARCHAR(255)   NULL
proposed_price      DECIMAL(10,2)
proposed_duration   VARCHAR(50)
status              VARCHAR(20)    -- 'PENDING' / 'ACCEPTED' / 'REJECTED'
created_at          TIMESTAMP

Resource
------------------------
id              BIGINT (PK, auto-increment)
project_id      BIGINT (FK → Project.id)
file_url        VARCHAR(255)
file_type       VARCHAR(50)
uploaded_at     TIMESTAMP

Wallet
------------------------
id              BIGINT (PK, auto-increment)
user_id         BIGINT (FK → User.id)   UNIQUE   -- enforces 1:1
balance         DECIMAL(10,2)   DEFAULT 0
updated_at      TIMESTAMP

Transaction
------------------------
id                  BIGINT (PK, auto-increment)
wallet_id           BIGINT (FK → Wallet.id)
type                VARCHAR(20)     -- 'TOPUP' / 'PAYMENT' / 'WITHDRAWAL' / 'FEE'
amount              DECIMAL(10,2)
related_project_id  BIGINT (FK → Project.id)   NULL
created_at          TIMESTAMP

Chat
------------------------
id              BIGINT (PK, auto-increment)
project_id      BIGINT (FK → Project.id)   UNIQUE   -- enforces 1:1
created_at      TIMESTAMP

Message
------------------------
id              BIGINT (PK, auto-increment)
chat_id         BIGINT (FK → Chat.id)
sender_id       BIGINT (FK → User.id)
text            TEXT
sent_at         TIMESTAMP
read_status     BOOLEAN   DEFAULT FALSE

Rating_Review
------------------------
id              BIGINT (PK, auto-increment)
project_id      BIGINT (FK → Project.id)
reviewer_id     BIGINT (FK → User.id)     -- who wrote it
reviewee_id     BIGINT (FK → User.id)     -- who it's about
rating          INT                       -- 1 to 5
comment         TEXT     NULL
created_at      TIMESTAMP
```

---

### A few technical notes worth reading carefully:

1. **`Wallet.user_id` and `Chat.project_id` are marked `UNIQUE`** — this is exactly how a database *enforces* a 1:1 relationship (without it, nothing stops someone from creating two wallets for one user, which would break your "balance" logic).

2. **Status fields (`Project.status`, `Proposal.status`) are stored as `VARCHAR` here for readability** — in actual JPA code (Phase 9), these become **Enums** (a fixed list of allowed values), which is safer than free text. Just flagging it now so it's not a surprise later.

3. **`Rating_Review` has two separate FKs to `User`** (`reviewer_id`, `reviewee_id`) — this is the "same entity, two relationships, two meanings" pattern we discussed in Phase 3, now made concrete in the table.

4. **Nothing here is M:N** — confirming what I mentioned earlier: your Phase 3 decision to make `Proposal` its own entity already resolved the one relationship that looked M:N at first glance.

---

---
