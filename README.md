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

---
