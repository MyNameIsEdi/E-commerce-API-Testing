Here is a comprehensive and structured `README.md` file tailored for your GitHub repository based on the Postman collection you provided.

---

# Postman API Testing Project

This repository contains an automated API testing collection built with Postman. It is designed to test a comprehensive backend system handling user authentication, content recommendations, user comments, admin settings, and shopping cart functionalities.

The collection is modularized into distinct folders (representing logical teams or domains) and includes automated environment variable chaining and QA test output logging.

## 🚀 Features

* **End-to-End Workflows:** Automatically passes dynamically generated data (like auth tokens and resource IDs) from one request to the next.
* **Built-in Assertions:** Every request includes tests to validate successful HTTP status codes and acceptable response times.
* **Console Logging:** Custom QA Test Outputs are generated in the Postman Console for easy debugging and reporting.
* **Organized Structure:** Endpoints are neatly separated into designated "Team" folders for modular execution.

## 📋 Folder Structure & API Workflow

The collection is divided into the following functional areas:

* **Team 1: Authentication & User Profile**
* Registers a new user, handles login to retrieve Bearer tokens, and tests the password recovery/reset flows.


* **Team 2: Recommendations CRUD**
* Creates, fetches, and updates recommendations. Automatically captures the newly created recommendation ID (`rec_id`) for downstream use.


* **Team 3: Comments System**
* Retrieves comments for a recommendation and posts new comments, capturing the comment ID (`bad_c_id`) for moderation testing.


* **Team 4: Admin & Moderation**
* Authenticates an admin user, tests user banning/unbanning endpoints, and manages global platform settings (like toggling recommendations on/off).


* **Team 5: Shopping Cart**
* Tests fetching, adding items, checking, and clearing the user's shopping cart.


* **Delete: Cleanup Phase**
* Teardown requests to maintain an idempotent environment. Deletes the generated comments, recommendations, and the test user profile.



## 🛠️ Environment Variables Setup

To run this collection successfully, you need to set up a Postman Environment. Some variables must be provided by you initially, while others are automatically populated by the collection's test scripts during execution.

### Initial Variables to Provide

| Variable | Description |
| --- | --- |
| `BaseURL` | The root URL of the API (e.g., `[https://sv-students-recommend.onrender.com](https://sv-students-recommend.onrender.com)`) |
| `name` | Test user's display name |
| `email` | Test user's email address |
| `password` | Test user's password |
| `adminuser` | Email address of an admin account |
| `adminpass` | Password for the admin account |

### Auto-Generated Variables (Do Not Set)

*The collection will automatically set and update these variables as it runs:*

* `user_id`
* `testuser_token`
* `rec_id`
* `admin_token`
* `bad_c_id`

## 🔬 Automated Tests

Every endpoint in this collection includes standard Postman tests written in JavaScript:

1. **Status Code Validation:** Ensures the API returns a `2xx` Success status code.
2. **Performance/Latency Validation:** Ensures the API response time is below a specific threshold (e.g., 1000ms or 10000ms).

Additionally, the collection utilizes the `pm.test` and `console.log` functions to print an organized **QA TEST OUTPUT** block in the Postman console detailing:

* Request Name & URL
* HTTP Method
* Status Code
* Response Time
* Parsed JSON Payload

## 💻 How to Run Locally

### Using the Postman App

1. Download and open **Postman**.
2. Click **Import** and upload the `collection.json` file.
3. Create a new Environment and add the **Initial Variables** listed above.
4. Select the environment from the top-right dropdown.
5. Click on the Collection name, go to the **Run** tab, and click **Run Collection**.

### Using Newman (CLI)

You can run this project from your terminal using Postman's command-line collection runner, Newman.

1. Install Newman globally via npm:
```bash
npm install -g newman

```


2. Run the collection, passing in your environment file:

```bash
   newman run API_testing_project.postman_collection.json -e your_environment.postman_environment.json

```

```

```
