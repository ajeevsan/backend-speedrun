# Comparison: NestJS vs Java vs Rust

Location in monorepo: `docs/comparison.md`

This file records how the three backends compare on the same API, the same data and the same load. Every result cell starts empty. Fill each one from a real run and never from estimates. Rows marked "TBD" are the work still to do.

Status:

- [ ] App 1: NestJS measured
- [ ] App 3: Java measured
- [ ] App 2: Rust measured
- [ ] Write-up finished

## 1. How to read this document

- **Section 2** says exactly how every number was produced, so someone else could repeat it.
- **Sections 3 to 5** hold the numbers.
- **Sections 6 to 8** hold the judgment: what the numbers mean and what you would choose.
- A number without a run date and a commit hash is not valid. Record both.

## 2. Test method

Keep these identical for all three backends. If you change any of them, re-run all three.

### 2.1 Environment

| Item | Value |
| --- | --- |
| Cloud region | TBD |
| Cloud Run CPU and memory per instance | TBD (use the same for all three) |
| Min and max instances | TBD (use min 0 for cold start tests, min 1 for load tests) |
| Cloud SQL tier | TBD |
| MongoDB host and tier | TBD |
| Redis tier | TBD |
| Load generator machine | TBD (run it in the same region) |
| k6 version | TBD |

### 2.2 Data set

| Item | Value |
| --- | --- |
| Tenants | TBD |
| Users | TBD |
| Admission applications | 100,000 (from `db/` seed scripts) |
| Enrollments | TBD |
| Seed script commit | TBD |

### 2.3 Load scenarios

All scripts live in `tests/load/` and are identical for every backend.

| Scenario | Script | What it does | Load shape |
| --- | --- | --- | --- |
| Read-heavy | `read.js` | Login once, then `GET /admissions` with pagination and filters | 200 requests/s for 5 min after a 1 min ramp |
| Write-heavy | `write.js` | `POST /admissions` and `PATCH /admissions/{id}/status` | 50 requests/s for 5 min |
| Mixed | `mixed.js` | 80% reads, 20% writes | 100 requests/s for 5 min |
| Analytics | `analytics.js` | `GET /analytics/enrollment` | 20 requests/s for 5 min |
| Stress | `stress.js` | Ramp until errors exceed 1% | Ramp 50 requests/s every minute |

### 2.4 Rules

- Warm up each backend for 2 minutes before measuring.
- Run each scenario 3 times and report the median run.
- Use the same Docker CPU and memory limits everywhere.
- Count a request as failed on any non-2xx response or any response above 5 seconds.

## 3. Runtime results

### 3.1 Latency (milliseconds)

| Scenario | Metric | NestJS | Java | Rust |
| --- | --- | --- | --- | --- |
| Read-heavy | p50 | | | |
| Read-heavy | p95 | | | |
| Read-heavy | p99 | | | |
| Write-heavy | p50 | | | |
| Write-heavy | p95 | | | |
| Write-heavy | p99 | | | |
| Mixed | p50 | | | |
| Mixed | p95 | | | |
| Mixed | p99 | | | |
| Analytics | p95 | | | |

### 3.2 Throughput and errors

| Measure | NestJS | Java | Rust |
| --- | --- | --- | --- |
| Max sustained requests/s with error rate under 1% (stress test) | | | |
| Error rate at 200 requests/s (read-heavy) | | | |
| Requests/s at which p95 first exceeds 300 ms | | | |

### 3.3 Resource use under the mixed scenario

| Measure | NestJS | Java | Rust |
| --- | --- | --- | --- |
| Peak memory (MB) | | | |
| Average memory (MB) | | | |
| Average CPU (% of one vCPU) | | | |
| Peak CPU (% of one vCPU) | | | |
| Instances needed to hold 200 requests/s at p95 under 300 ms | | | |

### 3.4 Startup

| Measure | NestJS | Java | Rust |
| --- | --- | --- | --- |
| Cold start to first successful response (seconds, median of 10) | | | |
| Local start to `/ready` returning 200 (seconds, median of 10) | | | |

## 4. Build and delivery results

| Measure | NestJS | Java | Rust |
| --- | --- | --- | --- |
| Docker image size (MB) | | | |
| Clean Docker build time (minutes) | | | |
| Cached Docker build time (minutes) | | | |
| CI pipeline duration, lint to deploy (minutes) | | | |
| Dependency count (direct) | | | |

## 5. Code and effort

| Measure | NestJS | Java | Rust |
| --- | --- | --- | --- |
| Lines of source code (excluding tests, generated code, blank lines) | | | |
| Lines of test code | | | |
| Line coverage (%) | | | |
| Files in `src/` | | | |
| Working days to reach the acceptance criteria | | | |
| Requirements implemented (Must / Should / Could) | | | |
| Compile or build time, clean (seconds) | | | |

Count lines with the same tool on all three, for example `tokei`, and record the command you ran.

## 6. Developer experience

Score each from 1 (poor) to 5 (excellent) after finishing the app, then add one sentence of evidence. A score without evidence is not useful.

| Area | NestJS | Java | Rust |
| --- | --- | --- | --- |
| Time to first working endpoint | | | |
| Quality of compiler and runtime error messages | | | |
| Refactoring confidence | | | |
| Test writing and speed | | | |
| Debugging | | | |
| Library maturity for this project's needs | | | |
| Documentation and community answers | | | |
| Local feedback loop (edit to result) | | | |

## 7. Where each backend struggled

Write one short paragraph per backend, with specifics.

### NestJS

TBD. Cover at least: the hardest bug, one thing the framework made easy, one thing that surprised you.

### Java

TBD. Cover at least: the hardest bug, any N+1 query found and fixed, the effect of virtual threads on the numbers.

### Rust

TBD. Cover at least: the hardest borrow-checker or lifetime error, how error handling compared, which features were cut and why.

## 8. Conclusions

Fill this in last, and only claim what the numbers above support.

- **Fastest at p95 under the mixed scenario:** TBD
- **Lowest memory use:** TBD
- **Smallest image and fastest cold start:** TBD
- **Fastest to build:** TBD
- **Easiest to maintain, in your opinion and why:** TBD
- **What I would pick for a new team project, and why:** TBD
- **What I would pick for a solo project, and why:** TBD
- **What surprised me most:** TBD

## 9. Caveats

State these honestly so the comparison is not misread.

- Results come from one implementation by one developer, who knew the NestJS app best and the Rust app least. Skill level affects the numbers.
- Most time in this system is spent waiting on databases, so language differences may be small on read-heavy scenarios.
- The Rust app implements a smaller feature set, which gives it less work per request on some paths. Note any scenario affected.
- Cloud Run results vary between runs. The median of three runs reduces this but does not remove it.
- Library choices (for example TypeORM vs Prisma, or SQLx vs Diesel) affect results as much as the language does.

## 10. Reproducing the results

```text
make up                          # start Postgres, Mongo, Redis, Strapi
make seed                        # load the shared data set
make run BACKEND=nestjs          # or java, or rust
make load-test SCENARIO=mixed    # runs the matching k6 script
```

Record for every run:

| Field | Value |
| --- | --- |
| Date | |
| Backend and commit hash | |
| Scenario | |
| Command | |
| Raw k6 output file | `tests/load/results/<date>-<backend>-<scenario>.json` |

## 11. Change log

| Date | Change |
| --- | --- |
| | Created from template |
