# QuickEMS — Full-Stack Employee Management System

QuickEMS is a full-stack Employee Management System built for organizations that need a clean, role-aware platform to manage their workforce. It provides two distinct portals — one for administrators and one for employees — each tailored to the actions that role can perform.

---

## Project Overview

QuickEMS separates concerns clearly: admins control the organization's data (employees, departments, payroll, and attendance records), while employees interact with their own profile, submit leave requests, mark attendance, and download payslips. All of this is backed by a real-time background-workflow engine (Inngest) that handles automated check-outs, leave reminders, and daily attendance nudges via email — without any manual scheduling.

---

## Key Features

### 🛡️ Admin Portal

- **Employee Lifecycle Management** — Create, update, search, filter by department, and soft-delete employee records. Each employee is given a linked user account automatically.
- **Department Configuration** — 10 built-in departments: Engineering, Human Resources, Marketing, Sales, Finance, Operations, IT Support, Customer Success, Product Management, and Design.
- **Payroll Processing** — Generate payslips per employee per month, specifying base salary, allowances, and deductions. Net salary is computed and stored automatically.
- **Dashboard Analytics** — At-a-glance metrics: total headcount, number of departments, today's attendance count, and pending leave applications.
- **Leave Review** — Approve or reject employee leave applications; an automated email reminder is sent to the admin if any application sits pending for more than 24 hours.

### 👤 Employee Portal

- **Attendance Tracking** — One-click check-in/check-out per day. Late arrivals (after 09:00) are flagged automatically. Working hours and day-type (Full Day / Three Quarter / Half Day / Short Day) are computed on check-out.
- **Leave Application Management** — Submit Sick, Casual, or Annual leave requests with start/end dates and a reason. Employees can track the status (Pending → Approved / Rejected) of each application.
- **Payslip Access & Printing** — View the full history of issued payslips and open a print-ready view for any individual slip.

### 🔒 Authentication — Role-Based Access Control (RBAC)

- Separate login pages for Admin (`/login/admin`) and Employee (`/login/employee`) portals.
- Passwords are hashed with **bcrypt**. On successful login the server issues a signed **JWT**.
- Every API request attaches the token via an `Authorization: Bearer` header; server middleware (`protect` + `protectAdmin`) validates the token and enforces role restrictions on every route.
- Session state is persisted in `localStorage` and re-validated against the server on page load.

### ⚙️ Inngest Background Workflows

| Function                   | Trigger                    | Behaviour                                                                                                                                      |
| -------------------------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `autoCheckOut`             | `employee/check-out` event | Waits 9 hours; if no check-out has been recorded, sends a reminder email, then waits 1 more hour and automatically marks a half-day check-out. |
| `leaveApplicationReminder` | `leave/pending` event      | Waits 24 hours; emails the admin if the application is still pending.                                                                          |
| `attendanceReminderCron`   | Daily cron — 11:30 AM IST  | Identifies active employees who haven't checked in and aren't on approved leave, then emails each one a reminder.                              |

---

## Tech Stack

| Layer               | Technology                                         |
| ------------------- | -------------------------------------------------- |
| **Frontend**        | React 19, Tailwind CSS 4, React Router DOM 7       |
| **HTTP Client**     | Axios (with JWT interceptor)                       |
| **UI Utilities**    | Lucide React (icons), React Hot Toast, date-fns    |
| **Backend**         | Node.js, Express 5                                 |
| **Database**        | MongoDB                                            |
| **Auth**            | JSON Web Tokens (`jsonwebtoken`), bcrypt           |
| **Background Jobs** | Inngest 4                                          |
| **Email**           | Nodemailer — SMTP via Brevo (smtp-relay.brevo.com) |
| **Dev tooling**     | Nodemon, ESLint, Vite HMR                          |

---

## Data Models

| Model                | Key Fields                                                                                                                                            |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **User**             | `email`, `password` (hashed), `role` (`ADMIN` \| `EMPLOYEE`)                                                                                          |
| **Employee**         | `firstName`, `lastName`, `email`, `phone`, `position`, `department`, `basicSalary`, `allowances`, `deductions`, `employmentStatus`, `joinDate`, `bio` |
| **Attendance**       | `employeeId`, `date`, `checkIn`, `checkOut`, `status` (`PRESENT` \| `LATE` \| `ABSENT`), `workingHours`, `dayType`                                    |
| **LeaveApplication** | `employeeId`, `type` (`SICK` \| `CASUAL` \| `ANNUAL`), `startDate`, `endDate`, `reason`, `status` (`PENDING` \| `APPROVED` \| `REJECTED`)             |
| **Payslip**          | `employeeId`, `month`, `year`, `basicSalary`, `allowances`, `deductions`, `netSalary`                                                                 |

---

## Getting Started

### Prerequisites

- **Node.js** v18 or later
- A **MongoDB** connection string (local or Atlas)
- A **Brevo** (or any SMTP) account for transactional email
- An **Inngest** account (or run the local dev server via `npx inngest-cli@latest dev`)

---

### 1. Clone the repository

```bash
git clone <repo-url>
cd FullStack-EMS
```

---

### 2. Configure environment variables

#### `server/.env`

```env
PORT=4000
MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/quickems
JWT_SECRET=your_jwt_secret_key

# Admin seed account
ADMIN_EMAIL=admin@yourcompany.com

# Brevo SMTP
SMTP_USER=your_brevo_smtp_user
SMTP_PASS=your_brevo_smtp_password
SENDER_EMAIL=noreply@yourcompany.com

# Inngest
INNGEST_EVENT_KEY=your_inngest_event_key
INNGEST_SIGNING_KEY=your_inngest_signing_key
```

#### `client/.env`

```env
VITE_BASE_URL=http://localhost:4000
```

---

### 3. Seed the admin account

The seed script creates the first admin user (password: `admin123`). **Change the password immediately after first login.**

```bash
cd server
npm run seed
```

---

### 4. Run the development servers

**Backend** — from the `server/` directory:

```bash
cd server
npm run server      # or: npm run dev
```

The API will be available at `http://localhost:4000`.

**Frontend** — from the `client/` directory:

```bash
cd client
npm run dev
```

The app will be available at `http://localhost:5173`.

---

### 5. (Optional) Run Inngest locally

To test background workflows in development, start the Inngest Dev Server and point it at your running backend:

```bash
npx inngest-cli@latest dev -u http://localhost:4000/api/inngest
```

---

## API Routes

| Method    | Path                 | Access   | Description               |
| --------- | -------------------- | -------- | ------------------------- |
| `POST`    | `/api/auth/login`    | Public   | Login and receive a JWT   |
| `GET`     | `/api/auth/session`  | Auth     | Validate current token    |
| `GET`     | `/api/dashboard`     | Auth     | Role-aware dashboard data |
| `GET`     | `/api/employees`     | Admin    | List all employees        |
| `POST`    | `/api/employees`     | Admin    | Create a new employee     |
| `PUT`     | `/api/employees/:id` | Admin    | Update an employee        |
| `DELETE`  | `/api/employees/:id` | Admin    | Soft-delete an employee   |
| `GET/PUT` | `/api/profile`       | Auth     | View / update own profile |
| `POST`    | `/api/attendance`    | Employee | Clock in or clock out     |
| `GET`     | `/api/attendance`    | Auth     | Fetch attendance records  |
| `POST`    | `/api/leave`         | Employee | Submit a leave request    |
| `GET`     | `/api/leave`         | Auth     | List leave applications   |
| `PATCH`   | `/api/leave/:id`     | Admin    | Approve or reject a leave |
| `POST`    | `/api/payslips`      | Admin    | Generate a payslip        |
| `GET`     | `/api/payslips`      | Auth     | Fetch payslips            |

---

## Project Structure

```
FullStack-EMS/
├── client/          # React + Vite frontend
│   └── src/
│       ├── pages/   # Dashboard, Employees, Attendance, Leave, Payslips, Settings
│       ├── components/
│       ├── context/ # AuthContext (JWT session)
│       └── api/     # Axios instance
└── server/          # Node.js + Express backend
    ├── models/      # User, Employee, Attendance, LeaveApplication, Payslip
    ├── controllers/
    ├── routes/
    ├── middleware/  # protect, protectAdmin
    ├── inngest/     # Background workflow functions
    ├── config/      # MongoDB connection, Nodemailer transport
    └── seed.js      # Creates the initial admin account
```
