# 🧪 Postman API Automation – Newman + Allure + Jenkins

This repository contains an end‑to‑end API automation framework for a Savings Account backend.  It uses a **Postman** collection as the source of truth, runs it via **Newman**, publishes rich reports through **Allure** and `newman-reporter-htmlextra`, and wraps everything in a one‑click **Jenkins** pipeline.

The framework executes a full customer journey — from registration through account application, KYC, transactions and statements — across twelve APIs with assertions.  After each run, it publishes:

- A comprehensive HTML report via the `newman-reporter-htmlextra` plugin
- A JUnit XML file for CI test result aggregation
- Allure raw results that can be visualised as interactive dashboards in Jenkins

## 📂 Project Structure

The root folder is organised to keep configuration, flows and reporting neatly separated:

```text
PostmanAPI_automation/
├─ collections/                # (optional) legacy Postman collections
├─ e2e flows/
│   └─ Postman API Automation.postman_collection.json  # main collection with E2E flow
├─ environments/
│   └─ QA.postman_environment.json    # environment variables (baseUrl, credentials)
├─ allure-results/             # generated Allure raw results (not committed)
├─ newman/
│   ├─ reports/
│   │   ├─ html/               # htmlextra HTML reports
│   │   ├─ json/               # raw JSON run output
│   │   └─ junit/              # qa-results.xml for JUnit
├─ node_modules/               # installed npm dependencies (ignored)
├─ Jenkinsfile                 # declarative Jenkins pipeline
├─ package.json                # npm metadata & test scripts
├─ package-lock.json
└─ .gitignore
```

> **Note:** the `allure-results/`, `newman/` and `node_modules/` folders are generated at runtime and should not be committed to source control.

## 🛠 Tech Stack & Framework Features

### Core technologies

- **Postman collection:** the main E2E flow is defined in `Postman API Automation.postman_collection.json`.
- **Execution engine:** **Newman** — Postman’s CLI runner — drives the tests.
- **Build & dependencies:** managed via **npm** (`package.json`).

### Reporting

- **CLI summary:** standard Newman output for quick feedback.
- **HTML reports:** via `newman-reporter-htmlextra`, giving high‑level summaries and per‑request details.
- **JUnit XML:** for CI systems to track pass/fail trends.
- **Allure:** through `newman-reporter-allure` and the Jenkins Allure plugin, providing interactive dashboards.

### Test data

- Dynamic data and random values via [`@faker-js/faker`](https://www.npmjs.com/package/@faker-js/faker) and `uuid`.
- Environment configuration loaded from `QA.postman_environment.json`.

### CI/CD

A declarative Jenkins pipeline (`Jenkinsfile`) orchestrates the end‑to‑end workflow:

1. **Checkout:** clones the repository.
2. **Node setup:** verifies Node.js and npm versions, then runs `npm ci` to install dependencies.
3. **Clean reports:** removes previous `newman/`, `allure-report/` and `allure-results/` folders.
4. **Run Newman:** executes `npm run test:qa`, generating CLI, HTML, JUnit and Allure results.
5. **Generate Allure report:** Jenkins Allure plugin transforms raw results into a nice dashboard.
6. **Publish HTML:** Jenkins HTML publisher archives the htmlextra report.

### Design principles

The framework applies DRY, KISS and SOLID‑inspired practices familiar from UI automation:

- **DRY:** the base URL and other variables are centralised in environment files; dynamic values (contact numbers, passwords, IDs) are calculated once in pre‑request scripts and reused.
- **KISS:** a single E2E Flow folder tells a clear story from registration to statement; one npm script (`test:qa`) drives the entire run.
- **Separation of concerns:** collections focus on flows, environments on configuration, Jenkinsfile on orchestration and reporters on visualisation.  New APIs or flows can be added without changing the pipeline or core wiring.

### Comparison with a Java + Selenium TAF

| Aspect | Selenium TAF | Postman/Newman TAF |
|---|---|---|
| Tech stack | Java, Selenium WebDriver, TestNG | Node.js ecosystem, Newman CLI |
| Test specification | Cucumber BDD feature files | Postman collections & folders |
| Reporting | Extent Reports, Allure, GitHub Pages | htmlextra HTML, Allure and JUnit in Jenkins |
| Configuration patterns | DriverFactory & ConfigManager | Centralised environment config & CLI wiring |
| Test artifacts | Page Object Model for UI pages | Request‑level scripts & chained API steps |

The same spirit of DRY/KISS/SOLID and **reusable configuration plus clear separation of concerns** is preserved — just mapped to an API context.

## 🌐 Functional Coverage – E2E Flow

The collection executes twelve API requests as a single integrated scenario representing the entire customer journey:

1. **`POST /addRegister`** – create a new customer registration.
2. **`GET /registerByContact`** – fetch the customer by dynamic contact number.
3. **`GET /registers`** – verify the customer appears in the list.
4. **`POST /addApplication`** – open a savings account application.
5. **`GET /applicationByContact/{contact}`** – verify application state.
6. **`GET /applications`** – list all applications.
7. **`POST /kycDataCapture`** – submit KYC details.
8. **`POST /kycVerification`** – approve the application post‑KYC.
9. **`POST /deposit`** – deposit transaction.
10. **`POST /withdraw`** – withdrawal transaction.
11. **`POST /transferFund/{contact}`** – transfer funds.
12. **`GET /displayStatement/{contact}?password=…`** – validate the full statement and its schema.

Each request includes a pre‑request script (to generate dynamic data) and test script assertions on status codes, response structures and business behaviour.

## 📦 Prerequisites

Before running the framework you need:

- **Node.js ≥ 18** (validated on Node 22) and npm.
- The **Allure command‑line** (optional) if you want to view Allure reports locally.
- A **running Savings Account API backend** at `http://localhost:8082/UL_SavingsAccount-API_prototype/`.  
  Update `environments/QA.postman_environment.json` if your backend URL differs.

## 🚀 Setup & Local Execution

1. **Clone the repository**

   ```bash
   git clone https://github.com/<your-username>/javascript_postmanAPI_automation.git
   cd javascript_postmanAPI_automation
   ```

2. **Install dependencies**

   Use `npm ci` for reproducible installs:

   ```bash
   npm ci
   ```

   This installs Newman, `newman-reporter-htmlextra`, `newman-reporter-allure`, `@faker-js/faker`, `uuid` and other dependencies.

3. **Start the backend**

   Start your Savings Account API so it listens at the base URL configured in the QA environment.  For example, if it’s a Spring Boot project:

   ```bash
   mvn spring-boot:run
   ```

   Adjust the command for your actual backend.

4. **Run the QA suite**

   From the project root:

   ```bash
   npm run test:qa
   ```

   This runs the E2E flow using the QA environment and generates:

   - HTML report: `newman/reports/html/qa-run.html`
   - JSON output: `newman/reports/json/qa-run.json`
   - JUnit XML: `newman/reports/junit/qa-results.xml`
   - Allure raw results: `allure-results/`

## 📊 Viewing Reports

1. **Newman HTML (htmlextra)**  
   Open the generated HTML report in a browser:

   ```bash
   start newman/reports/html/qa-run.html   # Windows
   open newman/reports/html/qa-run.html    # macOS
   xdg-open newman/reports/html/qa-run.html  # Linux
   ```

   You will see a high‑level summary (pass/fail, response times) and detailed request/response information.

2. **Allure report (local)**  
   If you have the Allure CLI installed:

   ```bash
   allure serve allure-results
   ```

   This spins up a local web UI with trend graphs, suite/test/step breakdowns and attachments for each request and response.

3. **JUnit in CI**  
   The file `newman/reports/junit/qa-results.xml` can be consumed by Jenkins’ JUnit plugin or other CI tools for test trends and dashboards.

## 🧩 Jenkins Integration

The supplied `Jenkinsfile` wires everything into a clean CI pipeline.  It performs the same steps as local execution and archives the reports.

### Pipeline stages

1. **Checkout** – clones the GitHub repository.
2. **Node setup & dependencies** – verifies Node and npm versions and runs `npm ci`.
3. **Clean old reports** – removes previous `newman/`, `allure-report/` and `allure-results/` folders.
4. **Run Newman (QA)** – executes `npm run test:qa` and generates CLI, HTML, JUnit and Allure results.
5. **Generate Allure report** – Jenkins Allure plugin creates a web dashboard from `allure-results/`, outputting to `allure-report/`.
6. **Publish HTML** – Jenkins HTML Publisher archives `newman/reports/html` as a “Newman HTML Report” on the job page.

### Artifacts & history

Each build archives:

- `allure-report.zip` – zipped Allure report.
- `qa-run.html` – htmlextra report.
- `qa-results.xml` – JUnit results.

Allure history tracks test case trends across builds.  A successful build shows 12/12 API tests passing with both Newman and Allure views available from the Jenkins job page.

## 🐛 Debugging & Troubleshooting

1. **Rerun just the collection locally**  
   If something fails in CI, run:

   ```bash
   npm run test:qa
   ```

   Watch the CLI output for failing assertions and sample request/response payloads.

2. **Run a single folder or request via Newman**  
   You can refine the command (or define another npm script) like:

   ```bash
   newman run "e2e flows/Postman API Automation.postman_collection.json" \
     --folder "Registration" \
     -e environments/QA.postman_environment.json \
     -r cli,htmlextra
   ```

   Alternatively, open the collection in Postman, select a request and hit **Send** to debug manually.

3. **Check Allure attachments**  
   Allure attaches the request body, response body, headers and other details to each step.  Review these attachments to see exactly what the backend returned at the moment of failure.

4. **Common issues**

   - **Backend not running:** all requests fail with connection errors.
   - **Wrong `baseUrl`:** leads to 404s or timeouts — check `QA.postman_environment.json`.
   - **Data dependency:** if an early step (e.g. `addRegister`) fails, later requests will misbehave.  Fix the root cause instead of the last failing test.

## 🔧 Extending the Framework

### Add a new API to the E2E flow

1. Add a new request in `Postman API Automation.postman_collection.json` under the appropriate folder (e.g., “E2E Flow”).
2. Configure the method, URL (use the `{{baseUrl}}` variable), headers and body.
3. Add a pre‑request script for dynamic data if required.
4. Define test scripts in the **Tests** tab (status code, schema/field assertions and business checks).
5. Re‑run `npm run test:qa` – all reports will update automatically.

### Add a new environment (e.g., UAT)

1. Duplicate `QA.postman_environment.json` as `UAT.postman_environment.json`.
2. Change `baseUrl` and any other environment‑specific variables.
3. Optionally add another npm script in `package.json`, for example:

   ```json
   {
     "scripts": {
       "test:qa": "newman run ... -e environments/QA.postman_environment.json ...",
       "test:uat": "newman run ... -e environments/UAT.postman_environment.json ..."
     }
   }
   ```

4. You can then run:

   ```bash
   npm run test:uat
   ```

## ✅ Summary – What This Framework Provides

- **Single‑command execution** of a 12‑step end‑to‑end customer journey.
- **Multiple reports** out of the box: CLI, HTML, JUnit and Allure.
- **DRY, KISS, SOLID‑aligned structure** with clear separation of configuration, flows and CI orchestration.
- **Jenkins‑ready** with proper artifacts and Allure dashboards.
- **Easy extension** for new APIs and environments.

---

With this framework you can automate comprehensive API journeys, obtain detailed insights through multiple reporting formats and integrate seamlessly into your CI/CD pipeline.
