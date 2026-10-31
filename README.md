# AI-Driven Course Registration System

Spring Boot REST backend for the AI course registration and academic planning project. The application
provides JWT-authenticated student, faculty, and administrator workflows, with a MySQL deployment option
and Gemini-powered course-planning recommendations.

## Requirements

- Java 21 or later
- Maven 3.9+
- Node.js 20 or later and npm (for the React frontend)
- Docker Compose (optional, for MySQL)

## Run locally

The default profile uses an in-memory H2 database:

```bash
mvn spring-boot:run
```

Set an administrator account before launch so you can create course catalog entries:

```bash
ADMIN_EMAIL=admin@example.edu ADMIN_PASSWORD='change-this-password' mvn spring-boot:run
```

In a second terminal, run the React frontend:

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`. The frontend uses `http://localhost:8080` by default; set
`VITE_API_BASE_URL` in `frontend/.env` to use another backend URL. The backend's allowed frontend
origin can be changed with `FRONTEND_ORIGIN`.

To use MySQL instead, start the database and point the application to it:

```bash
docker compose up -d mysql
DATABASE_URL='jdbc:mysql://localhost:3306/course_registration' \
DATABASE_USERNAME=course_app DATABASE_PASSWORD=course_app_local \
ADMIN_EMAIL=admin@example.edu ADMIN_PASSWORD='change-this-password' \
mvn spring-boot:run
```

For personalized AI recommendations, set `GEMINI_API_KEY`. The planning endpoint returns HTTP 503 when
the key is not configured; provider errors are returned as errors rather than silently replaced with
non-AI results.

Before deployment, set a unique, randomly generated `JWT_SECRET` of at least 32 bytes, change database
credentials, and set `DDL_AUTO=validate` after creating a schema migration. The default JWT secret and
database credentials are for local development only.

## REST API

All request and response bodies use JSON. Except for registration/login, endpoints require
`Authorization: Bearer <token>`. New self-registered accounts always receive the `STUDENT` role.

| Method | Path | Access | Purpose |
|---|---|---|---|
| POST | `/api/auth/register` | Public | Create a student account |
| POST | `/api/auth/login` | Public | Sign in and receive a JWT |
| GET | `/api/courses` | Authenticated | List the course catalog |
| GET | `/api/courses/{code}` | Authenticated | Get course details |
| POST | `/api/courses` | ADMIN | Create a course |
| PUT | `/api/courses/{code}` | ADMIN | Update a course |
| POST | `/api/admin/faculty` | ADMIN | Create a faculty account |
| POST | `/api/registrations` | STUDENT | Register for a course |
| GET | `/api/registrations/me` | STUDENT | View your registrations and completed courses |
| GET | `/api/registrations` | FACULTY or ADMIN | View registration records |
| DELETE | `/api/registrations/{id}` | Registration owner | Drop your registration |
| PUT | `/api/registrations/{id}/complete` | FACULTY or ADMIN | Mark a registration completed |
| POST | `/api/planning/recommendations` | STUDENT | Generate an AI-assisted academic plan |

Course request example:

```json
{
  "code": "CS301",
  "title": "Data Structures",
  "credits": 3,
  "capacity": 40,
  "dayOfWeek": "MONDAY",
  "startTime": "09:00",
  "endTime": "10:30",
  "prerequisites": ["CS201"]
}
```

Registration checks course capacity, completed prerequisites, duplicate enrollment, and overlapping
weekly class times. User registration does not accept a role field, preventing public account creation
with faculty or administrator privileges.
