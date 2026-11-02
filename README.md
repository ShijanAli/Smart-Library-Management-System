# Smart Library Management System

A web-based library system for a university. Students can search and borrow books and reserve unavailable ones online. Librarians manage books, approve borrow requests and record returns. Admins manage users and view reports. Late-return fines are calculated automatically, and students can pay fines online through the SSLCommerz sandbox.

This project is built as a team project for the System Design course.

## Features

- **Authentication:** register and login with role-based access (Student, Librarian, Admin)
- **Book management:** add, edit and delete books, authors, categories and physical copies
- **Catalogue:** search by title, author, category or ISBN, with filters and availability
- **Borrowing:** student sends a request, librarian approves or rejects, return is recorded
- **Reservation:** reserve a book when no copy is free, cancel a reservation, view the queue
- **Fines:** automatic fine for late return (5 BDT per day), mark as paid in cash or pay online
- **Online payment:** SSLCommerz sandbox with success, fail, cancel and IPN handling
- **Dashboard:** total books, borrowed, overdue and fine summary

### Business rules

- A student can borrow at most 3 books at a time.
- The borrow period is 14 days.
- Late fine is 5 BDT per day after the due date.
- A student with an unpaid fine cannot borrow a new book.
- Reservations follow first come, first served.

## Tech Stack

| Layer    | Technology                                      |
| -------- | ----------------------------------------------- |
| Frontend | React (Vite), Tailwind CSS, React Router, Axios |
| Backend  | Node.js, Express                                |
| Database | MySQL                                           |
| ORM      | Prisma                                          |
| Auth     | JWT, bcrypt                                     |
| Payment  | SSLCommerz (sandbox)                            |

## Project Structure

```
smart-library/
├── client/                  # React frontend
│   └── src/
│       ├── pages/           # One folder per module (books, borrow, reservation, ...)
│       ├── components/      # Shared UI (Navbar, Table, Modal, Button, ...)
│       └── lib/             # Axios instance and helpers
├── server/                  # Express backend
│   ├── prisma/
│   │   └── schema.prisma    # Database schema (from the ER diagram)
│   └── src/
│       ├── modules/         # One folder per module: routes, controller, service
│       ├── middleware/      # Auth and role checks, error handler
│       └── app.js
└── docs/                    # SRS, use case and ER diagrams, presentation slides
```

## Getting Started

### Prerequisites

- Node.js 18 or later
- MySQL 8
- Git

### 1. Clone the repository

```bash
git clone <repository-url>
cd smart-library
```

### 2. Backend setup

```bash
cd server
npm install
cp .env.example .env     # then fill in the values
npx prisma migrate dev
npx prisma db seed       # optional: sample users, books and categories
npm run dev
```

### 3. Frontend setup

```bash
cd client
npm install
cp .env.example .env
npm run dev
```

The frontend runs on `http://localhost:5173` and the backend on `http://localhost:5000` by default.

## Environment Variables

### `server/.env`

```env
DATABASE_URL="mysql://USER:PASSWORD@localhost:3306/smart_library"
JWT_SECRET="change-this-secret"
PORT=5000
CLIENT_URL="http://localhost:5173"

# SSLCommerz sandbox credentials (never commit real values)
SSLC_STORE_ID="your-sandbox-store-id"
SSLC_STORE_PASSWD="your-sandbox-store-password"
SSLC_IS_LIVE=false
```

### `client/.env`

```env
VITE_API_URL="http://localhost:5000/api"
```

Never commit `.env` files. Only commit the `.env.example` files.

## Payment (SSLCommerz Sandbox)

1. Create a sandbox store at the SSLCommerz developer site and put the store ID and password in `server/.env`.
2. Keep `SSLC_IS_LIVE=false` for development.
3. Payment flow:
   - `POST /api/payments/init` creates a pending payment and returns the gateway URL.
   - The student completes payment on the SSLCommerz page.
   - SSLCommerz calls `/api/payments/success`, `/fail`, `/cancel` and `/ipn`.
   - The server validates the transaction with the `val_id` before it marks the payment and the fine as paid.
4. Test cards and wallet numbers are listed in the SSLCommerz sandbox documentation.

## API Overview

| Module        | Base path           | Main endpoints                                                         |
| ------------- | ------------------- | ---------------------------------------------------------------------- |
| Auth          | `/api/auth`         | `POST /register`, `POST /login`                                        |
| Profile       | `/api/me`           | `GET`, `PUT`, `PUT /password`                                          |
| Users (admin) | `/api/users`        | `GET`, `PATCH /:id/status`, `PATCH /:id/role`                          |
| Categories    | `/api/categories`   | CRUD                                                                   |
| Authors       | `/api/authors`      | CRUD                                                                   |
| Books         | `/api/books`        | CRUD, `GET/POST /:id/copies`, `PATCH /api/copies/:id`                  |
| Catalogue     | `/api/catalogue`    | `GET`, `GET /:id`                                                      |
| Reservations  | `/api/reservations` | `POST`, `DELETE /:id`, `GET /me`, `GET`                                |
| Borrows       | `/api/borrows`      | `POST`, `GET /me`, `GET`, `PATCH /:id/approve`, `/reject`, `/return`   |
| Fines         | `/api/fines`        | `GET`, `GET /me`, `PATCH /:id/pay`                                     |
| Payments      | `/api/payments`     | `POST /init`, `/success`, `/fail`, `/cancel`, `/ipn`, `GET /me`, `GET` |
| Dashboard     | `/api/dashboard`    | `GET`                                                                  |

## Team Workflow

### Work split

Each feature is split into a Backend task and a Frontend task. The task list, with owners, dependencies and status, is kept in the shared Google Sheet (**Tasks** tab). Each member picks their own tasks there.

### Git branches

- `main` is protected. Never push directly to it.
- Create one branch per task, for example `feature/B-03-book-api` or `feature/B-04-book-form`.
- Open a pull request to `main` and ask one teammate to review it.
- Keep commits small and write clear messages, for example `feat(books): add book copy API`.

### Commit message format

```
<type>(<module>): <short description>

types: feat, fix, docs, refactor, test, chore
```

### Requirements

Requirements are collected in the **Requirements** tab of the Google Sheet. The **SRS Merged** tab combines them automatically.

## Documentation

- `docs/SRS.md`: software requirements specification
- `docs/usecase/`: use case diagram and descriptions
- `docs/er/`: ER diagram and schema notes

## Team

| Name          | Role                                         |
| ------------- | -------------------------------------------- |
| Md Shijan Ali | Team Lead (setup, auth, integration, review) |
| Member 2      | Book Management                              |
| Member 3      | Search & Reservation                         |
| Member 4      | Borrow, Return & Fine                        |

## License

This project is for educational purposes.
