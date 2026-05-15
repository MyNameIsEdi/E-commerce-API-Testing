# SV Students Recommend — API Test Suite

<div align="center">

![Postman](https://img.shields.io/badge/Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white)
![Newman](https://img.shields.io/badge/Newman-CLI_Runner-4CAF50?style=for-the-badge&logo=node.js&logoColor=white)
![OAS 3.1](https://img.shields.io/badge/OpenAPI-3.1-6BA539?style=for-the-badge&logo=openapiinitiative&logoColor=white)
![Status](https://img.shields.io/badge/Tests-Passing-brightgreen?style=for-the-badge)

**A fully automated, end-to-end Postman collection for the SV College practice API.**  
Covers authentication, recommendations, comments, admin controls, cart management — and a clean teardown.

[Live API →](https://sv-students-recommend.onrender.com) · [Swagger Docs →](https://sv-students-recommend.onrender.com/docs) · [OpenAPI JSON →](https://sv-students-recommend.onrender.com/openapi.json)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Collection Structure](#collection-structure)
- [Bugs Found](#bugs-found)
- [Environment Variables](#environment-variables)
- [Running the Tests](#running-the-tests)
- [Test Strategy](#test-strategy)
- [Clean Slate Principle](#clean-slate-principle)
- [Team Assignments](#team-assignments)

---

## Overview

This collection was built directly from the live OpenAPI 3.1 spec (`/openapi.json`) and validates every documented endpoint across all authorization scopes — public, authenticated user, and admin.

Key design decisions:

- **No hardcoded values.** Every ID, token, and dynamic value is captured from prior responses and stored as an environment variable.
- **Schema-accurate tests.** Each assertion is written against the actual OpenAPI schemas (`UserOut`, `RecommendationOut`, `CommentOut`, etc.), not assumed structures.
- **Negative tests included.** Every team folder contains at least one negative test case (wrong credentials, missing auth, out-of-range values, invalid enums) — as required by the project spec.
- **Clean teardown.** The final folder deletes every resource created during the run and clears all auto-populated environment variables, leaving zero pollution for the next run.

---

## Collection Structure

```
📦 SV Students Recommend — API Test Suite v3
│
├── 📁 Team 1 — Auth & Profile
│   ├── ✅ Register New User                  POST  /auth/register
│   ├── ❌ Register – Duplicate Email          POST  /auth/register        [negative]
│   ├── ❌ Register – Missing Required Field   POST  /auth/register        [negative]
│   ├── ✅ Login – Get User Token              POST  /auth/login
│   ├── ❌ Login – Wrong Password              POST  /auth/login           [negative]
│   ├── ✅ Get My Profile                      GET   /api/profile/me
│   ├── ✅ Get Bearer Token                    GET   /api/profile/token
│   ├── ✅ Password Reset – Request Email      POST  /auth/recover
│   ├── ✅ Change Password                     PUT   /api/profile/password
│   └── ✅ Login After Password Change         POST  /auth/login
│
├── 📁 Team 4 — Admin  ← run before Teams 2 & 3 to get admin_token
│   ├── ✅ Login – Get Admin Token             POST  /auth/login
│   ├── ✅ Get All Users                       GET   /api/admin/users
│   ├── ❌ Get All Users – Regular Token       GET   /api/admin/users      [negative]
│   ├── ✅ Ban User                            POST  /api/admin/users/{id}/ban
│   ├── ✅ Unban User                          POST  /api/admin/users/{id}/unban
│   ├── ✅ Get Admin Settings                  GET   /api/admin/settings
│   ├── ✅ Turn Off Recommendations            PUT   /api/admin/settings/recommendations_enabled
│   └── ✅ Turn On Recommendations             PUT   /api/admin/settings/recommendations_enabled
│
├── 📁 Team 2 — Recommendations
│   ├── ✅ Get All Recommendations             GET   /api/recommendations
│   ├── ✅ Get All – Filter by Category        GET   /api/recommendations?category=Movie
│   ├── ✅ Create Recommendation               POST  /api/recommendations
│   ├── ❌ Create – No Token                   POST  /api/recommendations  [negative]
│   ├── ✅ Get Recommendation by ID            GET   /api/recommendations/{id}
│   ├── ❌ Get – Non-Existent ID               GET   /api/recommendations/{id} [negative]
│   ├── ✅ Update Recommendation               PUT   /api/recommendations/{id}
│   └── ❌ Update – Invalid Category Enum      PUT   /api/recommendations/{id} [negative]
│
├── 📁 Team 3 — Comments & Blacklist
│   ├── ✅ Get Comments for Recommendation     GET   /api/recommendations/{id}/comments
│   ├── ✅ Add Comment                         POST  /api/recommendations/{id}/comments
│   ├── ❌ Add Comment – Rating Out of Range   POST  /api/recommendations/{id}/comments [negative]
│   ├── ❌ Add Comment – No Auth               POST  /api/recommendations/{id}/comments [negative]
│   ├── ✅ Get Blacklist                        GET   /api/admin/blacklist
│   ├── ✅ Add Email to Blacklist              POST  /api/admin/blacklist
│   └── ❌ Add to Blacklist – Regular Token    POST  /api/admin/blacklist  [negative]
│
├── 📁 Team 5 — Cart
│   ├── ✅ Get Cart                            GET   /api/cart
│   ├── ✅ Add Item to Cart                    PUT   /api/cart
│   ├── ✅ Verify Cart Has Item                GET   /api/cart
│   ├── ❌ Get Cart – No Auth                  GET   /api/cart             [negative]
│   ├── ✅ Clear Cart                          PUT   /api/cart
│   └── ✅ Verify Cart Is Empty                GET   /api/cart
│
└── 📁 Teardown — Cleanup
    ├── 🧹 Delete Comment                      DELETE /api/recommendations/{id}/comments/{id}
    ├── 🧹 Delete Blacklist Entry              DELETE /api/admin/blacklist/{entry_id}
    ├── 🧹 Delete Recommendation               DELETE /api/recommendations/{id}
    └── 🧹 Delete User Account                 DELETE /api/profile/me
```

> **Run order:** Team 1 → Team 4 → Team 2 → Team 3 → Team 5 → Teardown

---

## Bugs Found

These bugs were discovered by cross-referencing the Postman collection against the live OpenAPI 3.1 spec (`/openapi.json`).

| # | Severity | Endpoint | Bug | Expected | Actual / In Original Collection |
|---|----------|----------|-----|----------|---------------------------------|
| 1 | 🔴 High | `PUT /api/profile/password` | Extra field `email` sent in request body | `PasswordChange` schema contains only `new_password` | Collection sent `{ email, new_password }` — `email` is not a valid field |
| 2 | 🔴 High | All DELETE endpoints | Wrong expected status code | `204 No Content` (documented) | Collection tested for generic `2xx` — a `200` would also pass, masking regressions |
| 3 | 🔴 High | `POST /api/recommendations/{id}/comments` | `comment_id` captured from wrong field | `CommentOut.id` is the comment's ID | Collection read `commenter_id` — that's the *user's* ID, not the comment's |
| 4 | 🟡 Medium | `PUT /api/admin/settings/{key}` | Wrong expected status code | `200 OK` with body (documented) | Collection used `.to.be.success` which accepts 200/201/204 equally |
| 5 | 🟡 Medium | `PUT /api/recommendations/{id}` | `category` field is an enum, not a free string | Only `Book`, `Movie`, `Series`, `Activity`, `Other` are valid | No validation tested; any string would pass |
| 6 | 🟡 Medium | `POST /api/recommendations/{id}/comments` | `rating` has min/max constraints | Integer `1–5` only (schema: `minimum: 1.0, maximum: 5.0`) | No out-of-range test existed |
| 7 | 🟡 Medium | `POST /api/recommendations` | `website_link` sent as literal string `"string"` | A real URL like `https://example.com` | Typo from Postman's auto-generated placeholder value |
| 8 | 🟢 Low | `GET /api/profile/me`, `GET /api/profile/token`, `GET /api/admin/users`, `GET /api/admin/blacklist`, `GET /api/admin/settings`, `GET /api/cart`, `PUT /api/cart` | Response schema is `{}` in OpenAPI | Documented response body structure | All 7 endpoints declare an empty `{}` schema — structural assertions are impossible without a real schema |
| 9 | 🟢 Low | `GET /api/recommendations/{id}` | `404` response not documented | When a non-existent UUID is queried, a `404` should be returned and documented | OpenAPI only documents `200` and `422` for this endpoint |

---

## Environment Variables

### Required before running (set manually)

| Variable | Description | Example |
|----------|-------------|---------|
| `BaseURL` | Root URL of the API | `https://sv-students-recommend.onrender.com` |
| `name` | Display name for the test account | `Test Student` |
| `email` | Email for the test account (must be unique per run) | `testuser@sv-test.dev` |
| `password` | Password for the test account (min 4 chars) | `TestPass1234!` |
| `adminuser` | Admin account email | `hagai@svcollege.co.il` |
| `adminpass` | Admin account password | `test1234` |

### Auto-populated during the run (do not set manually)

| Variable | Set by | Cleared by |
|----------|--------|------------|
| `user_id` | Register New User | 🧹 Delete User Account |
| `testuser_token` | Login – Get User Token | 🧹 Delete User Account |
| `admin_token` | Login – Get Admin Token | *(persists — admin account is not deleted)* |
| `rec_id` | Create Recommendation | 🧹 Delete Recommendation |
| `comment_id` | Add Comment | 🧹 Delete Comment |
| `blacklist_id` | Add Email to Blacklist | 🧹 Delete Blacklist Entry |

---

## Running the Tests

### Option 1 — Postman GUI

1. Click **Import** and select both files:
   - `SV_Students_Recommend_Collection_v3_FINAL.json`
   - `SV_Students_Recommend_Environment_FIXED.json`
2. Select the `SV Students Recommend – Environment` from the environment dropdown (top right).
3. Fill in the six required variables (`BaseURL`, `name`, `email`, `password`, `adminuser`, `adminpass`).
4. Right-click the collection → **Run collection**.
5. Ensure **Run folder by folder in order** is enabled and click **Run**.

### Option 2 — Newman CLI

Ideal for CI/CD pipelines (GitHub Actions, Jenkins, GitLab CI).

```bash
# Install Newman
npm install -g newman

# Run the full suite
newman run SV_Students_Recommend_Collection_v3_FINAL.json \
  -e SV_Students_Recommend_Environment_FIXED.json \
  --env-var "email=testuser-$(date +%s)@sv-test.dev" \
  --reporters cli,junit \
  --reporter-junit-export results/report.xml
```

> **Tip:** Use `$(date +%s)` or a UUID in the email to guarantee uniqueness across runs.

### Option 3 — GitHub Actions (CI/CD)

```yaml
name: API Test Suite

on:
  push:
    branches: [main]
  pull_request:

jobs:
  api-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Newman
        run: npm install -g newman

      - name: Run Postman Collection
        run: |
          newman run SV_Students_Recommend_Collection_v3_FINAL.json \
            -e SV_Students_Recommend_Environment_FIXED.json \
            --env-var "email=ci-$(date +%s)@sv-test.dev" \
            --env-var "BaseURL=${{ secrets.API_BASE_URL }}" \
            --env-var "adminuser=${{ secrets.ADMIN_EMAIL }}" \
            --env-var "adminpass=${{ secrets.ADMIN_PASS }}" \
            --reporters cli,junit \
            --reporter-junit-export results/report.xml

      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: newman-report
          path: results/report.xml
```

Store sensitive values (`adminuser`, `adminpass`, etc.) as **GitHub Secrets** — never commit them to the repository.

---

## Test Strategy

Every request includes a minimum of **4 assertions** in the Tests tab:

```
1. HTTP status code — exact match against the OpenAPI documented code
2. Response time    — must be below 5 000 ms (accounts for cold-starts on Render's free tier)
3. Schema check     — verifies that required fields exist and have the correct type
4. Data integrity   — verifies that returned values match what was sent (e.g. returned email = sent email)
```

Negative tests additionally assert:
- That error status codes fall in the documented range (e.g. `422` for validation errors, `401` for missing auth, `403` for insufficient privileges)
- That `access_token` is **absent** from error responses
- That `detail` is present in `HTTPValidationError` bodies

All console output follows the format:
```
[Request Name] STATUS_CODE | RESPONSE_TIME ms
```

---

## Clean Slate Principle

A test run that does not clean up after itself is a failed test run — it pollutes the environment and causes false failures in subsequent runs.

This collection enforces clean slate via the **Teardown** folder, which:

1. Deletes the comment (`DELETE /api/recommendations/{id}/comments/{comment_id}`)
2. Deletes the blacklist entry (`DELETE /api/admin/blacklist/{blacklist_id}`)
3. Deletes the recommendation (`DELETE /api/recommendations/{rec_id}`)
4. Deletes the test user account (`DELETE /api/profile/me`)

After each deletion the corresponding environment variable is unset via `pm.environment.unset(...)`.  
On a clean run, the final console message is:

```
✅ ====== CLEANUP COMPLETE – ENVIRONMENT IS CLEAN ====== ✅
```

---

## Team Assignments

| Team | Domain | Members | Endpoints Covered |
|------|--------|---------|-------------------|
| Team 1 | Auth & Profile | אנסטסיה, רון, שגב | `/auth/register`, `/auth/login`, `/auth/recover`, `/api/profile/me`, `/api/profile/token`, `/api/profile/password` |
| Team 2 | Recommendations | אדי, קסנייה, אביעד | `/api/recommendations` (GET/POST/PUT/DELETE), filter by category |
| Team 3 | Comments & Blacklist | סיון, יונתן ובמן, כרמית | `/api/recommendations/{id}/comments`, `/api/admin/blacklist` |
| Team 4 | Admin | אלרן, יעקב, יהונתן כהן | `/api/admin/users`, `/api/admin/users/{id}/ban`, `/api/admin/settings` |
| Team 5 | Cart & Cleanup | איליה, אנדריי | `/api/cart` (GET/PUT), teardown |

> **Note for Team 4:** Admin credentials require special permissions. Contact the course instructor to receive access before running the collection.
