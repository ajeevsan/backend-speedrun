# App 2: Rust (Axum) Backend Requirements

Location in monorepo: `apps/api-rust/`
Build order: 3 of 3 (built last, after NestJS and Java)
Suggested time-box: 8 to 10 weeks, set a hard stop

## 1. Purpose

Reimplement the core of the School SaaS Platform API in Rust with Axum, matching the behavior of the NestJS and Java apps. It must pass the shared e2e suite for the Must requirements. Because Rust has the steepest learning curve, this app has the smallest scope of the three.

## 2. Learning goals

- Understand ownership, borrowing and lifetimes by writing real handlers and services.
- Learn async Rust with Tokio, and how Axum extractors, state and middleware work.
- Design error handling with `Result`, `thiserror` and `IntoResponse`.
- Measure what Rust changes in latency, memory, image size and cold start, and explain why.

## 3. Tech stack

| Concern | Choice |
| --- | --- |
| Language | Rust (current stable), edition 2021 |
| Framework | Axum on Tokio |
| MongoDB | `mongodb` crate |
| PostgreSQL | SQLx with migrations from `db/postgres/schema.sql` |
| Auth | `jsonwebtoken`, `argon2`, custom Tower middleware |
| Validation | `validator` with `serde` |
| Queue | Tokio background tasks with the `redis` crate |
| Email | `lettre` |
| Rate limiting | `tower_governor` |
| HTTP client (Strapi) | `reqwest` |
| API docs | `utoipa` and `utoipa-swagger-ui`, checked against `contracts/openapi.yaml` |
| Errors | `thiserror` for library code, `anyhow` at the edges |
| Logging | `tracing` and `tracing-subscriber` (JSON) |
| Testing | `cargo test`, Testcontainers |
| Lint and format | `clippy` (deny warnings) and `rustfmt` |

## 4. Functional requirements

| ID | Module | Requirement | Priority |
| --- | --- | --- | --- |
| FR-01 | Auth | Signup and login with email and password | Must |
| FR-02 | Auth | JWT access token (15 min) and refresh token (7 days) with rotation | Must |
| FR-03 | RBAC | Role checks on every endpoint | Must |
| FR-04 | Tenancy | Resolve tenant from subdomain or header; no cross-tenant reads | Must |
| FR-05 | Admissions | Same state machine as the reference app | Must |
| FR-06 | Enrollment | Accepted applications convert to an enrollment | Must |
| FR-07 | Analytics | Enrollment counts by class, month and referral source | Must |
| FR-08 | Referral | Single referral-source field with staff verification | Should |
| FR-09 | Notifications | Email on submitted, status change and enrollment confirmed | Should |
| FR-10 | Notifications | Queue with retries | Could |
| FR-11 | Master data | Read classes, academic years and fee categories from Strapi | Must |
| FR-12 | Master data | Handle Strapi webhooks | Could |
| FR-13 | Common | Pagination, filtering and sorting on all list endpoints | Must |
| FR-14 | Common | Consistent error format and request validation | Must |
| FR-15 | Common | Rate limiting per IP | Could |
| FR-16 | Files | Upload to Google Cloud Storage | Cut |
| FR-17 | Audit | Record changes | Cut |

Only the Must items are required. Should items come next if time allows. Could and Cut items are skipped unless everything else is finished. Document every skipped item in the README so the reduced scope is a stated decision rather than a gap.

## 5. Non-functional requirements

| ID | Area | Requirement |
| --- | --- | --- |
| NFR-01 | Performance | Hold the same p95 target as App 1: under 300 ms at 200 requests/s on one Cloud Run instance |
| NFR-02 | Performance | List endpoints return in under 500 ms with 100,000 seeded applications |
| NFR-03 | Security | Secrets in Secret Manager; none in the repo |
| NFR-04 | Security | All input validated; no `unsafe` blocks, or each one justified in a comment |
| NFR-05 | Reliability | Graceful shutdown on SIGTERM, health and readiness endpoints |
| NFR-06 | Observability | JSON logs with request ID and tenant ID via `tracing` spans |
| NFR-07 | Testing | Shared e2e suite passes; unit tests on services and error mapping |
| NFR-08 | Portability | Multi-stage Docker build to a small final image, runs from the shared Compose file |
| NFR-09 | Maintainability | `cargo fmt --check` and `cargo clippy -- -D warnings` block merges in CI |

## 6. Suggested internal structure

```text
apps/api-rust/
├── src/
│   ├── main.rs
│   ├── config.rs        # env loading and validation
│   ├── state.rs         # AppState shared through Arc
│   ├── error.rs         # AppError and IntoResponse
│   ├── routes/          # one module per resource
│   ├── handlers/
│   ├── services/
│   ├── repositories/    # SQLx and MongoDB access
│   ├── models/          # DTOs and domain types
│   ├── middleware/      # auth, tenant, request ID
│   └── clients/         # Strapi, email
├── tests/               # integration tests
├── migrations/
├── Cargo.toml
├── Dockerfile
└── README.md
```

## 7. Language-specific requirements

- [ ] A single `AppError` enum maps every failure to the shared error format
- [ ] Shared state lives in `Arc<AppState>`; no global mutable state
- [ ] Tenant ID is extracted by a custom Axum extractor so handlers cannot forget it
- [ ] SQLx offline mode is enabled so CI builds without a live database
- [ ] A short note in `docs/learnings/rust.md` explaining one borrow-checker error you hit and how you fixed it
- [ ] Final image size and cold start time are recorded against the other two apps

## 8. Out of scope for this app

Everything out of scope for App 1, plus FR-16 and FR-17.

## 9. Acceptance criteria

- [ ] Every Must requirement above works and is covered by a test
- [ ] Every Must endpoint in `contracts/openapi.yaml` is implemented and matches its schema; skipped endpoints are listed in the README
- [ ] The shared e2e suite passes for the implemented scope
- [ ] Runs from `docker-compose.yml` with one command
- [ ] CI is green and the image is deployed to Cloud Run
- [ ] k6 results are recorded in `docs/comparison.md` using the shared scripts
- [ ] `README.md` explains setup, architecture, trade-offs and the reduced scope
- [ ] `docs/learnings/rust.md` is written
- [ ] Release tagged `rust-v1.0.0`
