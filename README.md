
# 🚀 Comprehensive API Automation & Testing Suite

![Postman](https://img.shields.io/badge/Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white)
![Newman](https://img.shields.io/badge/Newman-Automated_Testing-4CAF50?style=for-the-badge&logo=postman)
![JSON](https://img.shields.io/badge/Format-JSON-blue?style=for-the-badge)

A fully automated, end-to-end Postman collection designed to validate a complex backend ecosystem. This project covers everything from secure user authentication and administrative controls to content recommendation CRUD operations and shopping cart management.

## 📑 Table of Contents
- [Features](#-features)
- [Collection Structure](#-collection-structure)
- [Prerequisites & Setup](#-prerequisites--setup)
- [Environment Variables](#-environment-variables)
- [Automated Testing Strategy](#-automated-testing-strategy)
- [Executing the Tests](#-executing-the-tests)

---

## ✨ Features

* **Data-Driven Workflows:** Utilizes automated environment variable setting/chaining to pass state (e.g., auth tokens, generated IDs) between requests seamlessly.
* **Idempotent Execution:** Includes a dedicated "Teardown" phase that deletes generated test data, ensuring the environment remains clean for subsequent test runs.
* **Granular QA Output:** Every endpoint logs highly readable `QA TEST OUTPUT` directly to the Postman Console, detailing request parameters, latency, and JSON payloads for rapid debugging.
* **Comprehensive Coverage:** Validates standard HTTP methods (`GET`, `POST`, `PUT`, `DELETE`) across varied authorization scopes (Public, User, Admin).

---

## 🗂 Collection Structure

The collection is modularized into feature-specific domains:

```text
📦 API Testing Project
├── 📁 Team 1: Auth & User Profile
│   ├── POST /auth/register (Registers test user)
│   ├── POST /auth/login (Extracts Bearer token)
│   └── PUT  /api/profile/password (Tests password resets)
├── 📁 Team 2: Recommendations
│   ├── POST /api/recommendations (Creates entity & extracts rec_id)
│   ├── GET  /api/recommendations/{rec_id}
│   └── PUT  /api/recommendations/{rec_id}
├── 📁 Team 3: Comments
│   ├── POST /api/recommendations/{rec_id}/comments (Captures bad_c_id)
│   └── GET  /api/recommendations/{rec_id}/comments
├── 📁 Team 4: Admin Roles & Settings
│   ├── POST /auth/login (Admin scope)
│   ├── POST /api/admin/users/{user_id}/ban & /unban
│   └── PUT  /api/admin/settings/recommendations_enabled
├── 📁 Team 5: Shopping Cart
│   ├── GET  /api/cart
│   └── PUT  /api/cart (Add/Clear items)
└── 📁 Delete: Teardown / Cleanup
    ├── DELETE /api/recommendations/{rec_id}/comments/{bad_c_id}
    ├── DELETE /api/recommendations/{rec_id}
    └── DELETE /api/profile/me (Deletes test user)

```

---

## ⚙️ Prerequisites & Setup

1. Download and install [Postman](https://www.postman.com/downloads/).
2. Clone this repository to your local machine.
3. Open Postman, click **Import**, and select the `API_testing_project.postman_collection.json` file.
4. Create a new **Postman Environment** and add the required initial variables (see below).

---

## 🔐 Environment Variables

To execute the tests successfully, define the following variables in your Postman Environment prior to running the collection:

### ✍️ User-Defined Variables (Required)

| Variable | Description | Example |
| --- | --- | --- |
| `BaseURL` | The root endpoint of the API | `https://sv-students-recommend.onrender.com` |
| `name` | Display name for the test account | `Test User` |
| `email` | Email for the standard test account | `testuser@example.com` |
| `password` | Password for the standard test account | `SecurePass123!` |
| `adminuser` | Existing Admin account email | `admin@example.com` |
| `adminpass` | Existing Admin account password | `AdminPass123!` |

### 🤖 Auto-Populated Variables (Do not set manually)

The scripts will automatically handle the assignment and lifecycle of these variables during execution:
`user_id`, `testuser_token`, `rec_id`, `admin_token`, `bad_c_id`

---

## 🧪 Automated Testing Strategy

Every request is accompanied by robust JavaScript assertions in the **Tests** tab. The standard baseline tests include:

1. **HTTP Status Code Verification:** `pm.response.to.be.success;` ensures endpoints are reachable and resolve successfully (2xx status codes).
2. **Performance Assertions:** `pm.expect(pm.response.responseTime).to.be.below(10000);` guarantees latency thresholds are met.
3. **Data Integrity Checks:** Post-execution scripts validate JSON schemas and verify that newly created resources return the expected properties.

---

## 💻 Executing the Tests

### Option 1: Postman GUI

Select your configured environment in the top right corner. Click on the collection folder name, navigate to the **Run** tab, ensure all requests are checked, and click **Run Collection**.

### Option 2: Newman CLI (For CI/CD)

You can run this suite directly from your terminal, which is ideal for integrating into GitHub Actions, Jenkins, or GitLab CI.

```bash
# Install Newman globally
npm install -g newman

# Run the collection with your environment variables
newman run API_testing_project.postman_collection.json -e your_env.postman_environment.json

```

```

***

Would you like me to also write a `github-actions.yml` workflow file so you can show potential employers that this collection runs automatically in CI/CD whenever you push code?

```
