# App 3: Java (Spring Boot) Backend Requirements

Location in monorepo: `apps/api-java/`
Build order: 2 of 3 (built after the NestJS app)
Suggested time-box: 5 to 6 weeks

## 1. Purpose

Reimplement the School SaaS Platform API in Java with Spring Boot, matching the behavior of the NestJS reference app. It must pass the same shared e2e suite and implement the same `contracts/openapi.yaml`. No product decisions are made here; the only variable is the language and framework.

## 2. Learning goals

- Learn core Java and the Spring ecosystem: dependency injection, beans, configuration, the request lifecycle.
- Understand JPA and Hibernate: entity mapping, lazy loading, transactions and the N+1 problem.
- Use modern Java (21): records, streams, `Optional`, pattern matching, virtual threads.
- Compare idioms against NestJS and write the differences down for interviews.

## 3. Tech stack

| Concern | Choice |
| --- | --- |
| Language | Java 21 |
| Framework | Spring Boot 3.x |
| Build | Maven |
| MongoDB | Spring Data MongoDB |
| PostgreSQL | Spring Data JPA with Hibernate, schema applied via Flyway from `db/postgres/schema.sql` |
| Auth | Spring Security with `jjwt`, Argon2 password encoder |
| Validation | Jakarta Bean Validation |
| Queue | Spring Data Redis with a worker and scheduled retries |
| Email | Spring Mail with HTML templates |
| Rate limiting | Bucket4j with Redis |
| API docs | `springdoc-openapi`, checked against `contracts/openapi.yaml` |
| Health | Spring Boot Actuator |
| Logging | Logback with `logstash-logback-encoder` (JSON) |
| Testing | JUnit 5, Mockito, Spring Boot Test, Testcontainers |
| Lint and format | Checkstyle and Spotless |

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
| FR-08 | Referral | Single referral-source field with staff verification | Must |
| FR-09 | Notifications | Email on submitted, status change and enrollment confirmed | Should |
| FR-10 | Notifications | Queue with 3 retries and delivery status | Should |
| FR-11 | Master data | Read classes, academic years and fee categories from Strapi | Must |
| FR-12 | Master data | Handle Strapi webhooks to refresh cached master data | Should |
| FR-13 | Common | Pagination, filtering and sorting on all list endpoints | Must |
| FR-14 | Common | Consistent error format and request validation | Must |
| FR-15 | Common | Rate limiting per tenant and per IP | Should |
| FR-16 | Files | Upload documents to Google Cloud Storage | Could |
| FR-17 | Audit | Record changes on admissions and enrollments | Could |

Should and Could items may be cut if time runs short. Every Must item must match the NestJS app exactly.

## 5. Non-functional requirements

| ID | Area | Requirement |
| --- | --- | --- |
| NFR-01 | Performance | Hold the same p95 target as App 1: under 300 ms at 200 requests/s on one Cloud Run instance |
| NFR-02 | Performance | List endpoints return in under 500 ms with 100,000 seeded applications |
| NFR-03 | Security | Secrets in Secret Manager; none in the repo |
| NFR-04 | Security | All input validated; Spring Security defaults reviewed |
| NFR-05 | Reliability | Graceful shutdown, Actuator health and readiness probes |
| NFR-06 | Observability | JSON logs with request ID and tenant ID |
| NFR-07 | Testing | At least 70% line coverage (JaCoCo); shared e2e suite passes |
| NFR-08 | Portability | Builds to a Docker image and runs from the shared Compose file |
| NFR-09 | Maintainability | Checkstyle and Spotless block merges in CI |

## 6. Suggested internal structure

```text
apps/api-java/
├── src/main/java/com/schoolplatform/
│   ├── Application.java
│   ├── config/          # security, mongo, redis, openapi config
│   ├── common/          # exception handler, tenant filter, error model
│   ├── auth/
│   ├── tenant/
│   ├── user/
│   ├── admission/
│   ├── enrollment/
│   ├── analytics/
│   ├── notification/
│   ├── masterdata/
│   ├── file/
│   └── audit/
├── src/main/resources/
│   ├── application.yml
│   ├── db/migration/    # Flyway baseline from db/postgres/schema.sql
│   └── templates/       # email templates
├── src/test/java/
├── pom.xml
├── Dockerfile
└── README.md
```

## 7. Language-specific requirements

- [ ] Tenant ID is held in a request-scoped holder set by a servlet filter and applied to every repository query
- [ ] Request and response bodies use records as DTOs; JPA entities never leave the service layer
- [ ] A `@RestControllerAdvice` produces the shared error format
- [ ] `spring.jpa.hibernate.ddl-auto` is `validate`, so the shared schema stays the source of truth
- [ ] At least one place where you find and fix an N+1 query, recorded in `docs/learnings/java.md`
- [ ] Virtual threads enabled, with the effect on k6 numbers noted in `docs/comparison.md`

## 8. Out of scope for this app

Everything out of scope for App 1, plus any feature not in the shared contract.

## 9. Acceptance criteria

- [ ] Every Must requirement above works and is covered by a test
- [ ] Every endpoint in `contracts/openapi.yaml` is implemented and matches its schema
- [ ] The shared e2e suite in `tests/e2e` passes with no test changes
- [ ] Runs from `docker-compose.yml` with one command
- [ ] CI is green and the image is deployed to Cloud Run
- [ ] k6 results are recorded in `docs/comparison.md` using the shared scripts
- [ ] `README.md` explains setup, architecture and trade-offs
- [ ] `docs/learnings/java.md` is written, including NestJS vs Spring differences
- [ ] Release tagged `java-v1.0.0`
