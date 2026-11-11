# App 1: NestJS Backend Requirements

Location in monorepo: `apps/api-nestjs/`
Build order: 1 of 3 (first backend, sets the reference behavior)
Suggested time-box: 6 to 8 weeks

## 1. Purpose

Build the reference implementation of the School SaaS Platform API in NestJS and TypeScript. This app defines the behavior that the Java and Rust backends must match. It must implement 100% of `contracts/openapi.yaml`.

## 2. Learning goals

- Structure a production NestJS app with modules, providers, guards, interceptors and pipes.
- Combine MongoDB (documents) and PostgreSQL (relational) in one service.
- Build the full delivery pipeline once: tests, Docker, CI, Cloud Run, load test.
- Produce the baseline numbers that App 3 and App 2 are compared against.

## 3. Tech stack

| Concern | Choice |
| --- | --- |
| Runtime | Node.js 20 LTS |
| Language | TypeScript with `strict: true` |
| Framework | NestJS 10 |
| MongoDB | Mongoose via `@nestjs/mongoose` |
| PostgreSQL | TypeORM with `synchronize: false`, mapped onto `db/postgres/schema.sql` |
| Auth | `@nestjs/passport`, `passport-jwt`, Argon2 for password hashing |
| Validation | `class-validator` and `class-transformer` |
| Queue | BullMQ on Redis |
| Email | Nodemailer with HTML templates |
| Rate limiting | `@nestjs/throttler` with Redis storage |
| API docs | `@nestjs/swagger`, checked against `contracts/openapi.yaml` |
| Health | `@nestjs/terminus` |
| Logging | Pino via `nestjs-pino` |
| Testing | Jest, Supertest, Testcontainers |
| Lint and format | ESLint, Prettier |

## 4. Functional requirements

| ID | Module | Requirement | Priority |
| --- | --- | --- | --- |
| FR-01 | Auth | Signup and login with email and password | Must |
| FR-02 | Auth | JWT access token (15 min) and refresh token (7 days) with rotation | Must |
| FR-03 | RBAC | Role checks on every endpoint (platform admin, school admin, staff, parent, student) | Must |
| FR-04 | Tenancy | Resolve tenant from subdomain or header; no cross-tenant reads | Must |
| FR-05 | Admissions | Parent submits an application; staff move it through draft, submitted, under review, accepted, rejected | Must |
| FR-06 | Enrollment | Accepted applications convert to an enrollment for a class and academic year | Must |
| FR-07 | Analytics | Enrollment counts by class, month and referral source | Must |
| FR-08 | Referral | Single referral-source field with staff verification | Must |
| FR-09 | Notifications | Email on application submitted, status change and enrollment confirmed | Must |
| FR-10 | Notifications | Queue with 3 retries and delivery status tracking | Must |
| FR-11 | Master data | Read classes, academic years and fee categories from Strapi | Must |
| FR-12 | Master data | Handle Strapi webhooks to refresh cached master data | Must |
| FR-13 | Common | Pagination, filtering and sorting on all list endpoints | Must |
| FR-14 | Common | Consistent error format and request validation | Must |
| FR-15 | Common | Rate limiting per tenant and per IP | Must |
| FR-16 | Files | Upload documents to Google Cloud Storage with signed URLs | Must |
| FR-17 | Audit | Record who changed what and when on admissions and enrollments | Must |

Because this is the reference app, every item is a Must here.

## 5. Non-functional requirements

| ID | Area | Requirement |
| --- | --- | --- |
| NFR-01 | Performance | p95 latency under 300 ms for read endpoints at 200 requests/s on one Cloud Run instance |
| NFR-02 | Performance | List endpoints return in under 500 ms with 100,000 seeded applications |
| NFR-03 | Security | Secrets in Secret Manager; none in the repo |
| NFR-04 | Security | All input validated; helmet enabled; OWASP top-10 checklist reviewed once |
| NFR-05 | Reliability | Graceful shutdown, health and readiness endpoints, failed jobs retried 3 times |
| NFR-06 | Observability | JSON logs with request ID and tenant ID; Sentry enabled |
| NFR-07 | Testing | At least 70% line coverage; shared e2e suite passes |
| NFR-08 | Portability | Builds to a Docker image and runs from the shared Compose file |
| NFR-09 | Maintainability | ESLint and Prettier checks block merges in CI |

## 6. Suggested internal structure

```text
apps/api-nestjs/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── common/          # guards, interceptors, filters, pipes, decorators
│   ├── config/          # env validation
│   ├── auth/
│   ├── tenants/
│   ├── users/
│   ├── admissions/
│   ├── enrollments/
│   ├── analytics/
│   ├── notifications/   # queue producers and workers
│   ├── master-data/     # Strapi client and webhook handler
│   ├── files/
│   ├── audit/
│   └── health/
├── test/                # e2e tests
├── package.json
├── Dockerfile
└── README.md
```

## 7. Language-specific requirements

- [ ] `tenant_id` is injected by a request-scoped interceptor, never passed by hand to each query
- [ ] DTOs are the only objects crossing the controller boundary; entities never leak out
- [ ] A global exception filter produces the shared error format
- [ ] Config is validated at startup and the app refuses to boot on bad config
- [ ] Swagger output is diffed against `contracts/openapi.yaml` in CI

## 8. Out of scope for this app

Payments, timetables, attendance, mobile apps and real-time chat.

## 9. Acceptance criteria

- [ ] Every FR and NFR above works and is covered by a test
- [ ] Every endpoint in `contracts/openapi.yaml` is implemented and matches its schema
- [ ] The shared e2e suite in `tests/e2e` passes against the deployed URL
- [ ] Runs from `docker-compose.yml` with one command
- [ ] CI is green and the image is deployed to Cloud Run
- [ ] k6 results are recorded in `docs/comparison.md`
- [ ] `README.md` explains setup, architecture and trade-offs
- [ ] `docs/learnings/nestjs.md` is written
- [ ] Release tagged `nestjs-v1.0.0`
