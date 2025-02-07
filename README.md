# Payment Processing API with Quarkus

Welcome to the **Payment Processing API Challenge**! This challenge is designed to help you dive deep into building a robust, production-like payment processing service using [Quarkus](https://quarkus.io). In this challenge, you'll develop a microservice that simulates a banking/payments system with full account management, transaction processing, and real-time notifications using modern reactive paradigms.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Use Cases](#use-cases)
- [Technology Stack & Quarkus Extensions](#technology-stack--quarkus-extensions)
- [API Endpoints](#api-endpoints)
- [Architecture & Design Guidelines](#architecture--design-guidelines)
- [Getting Started](#getting-started)
- [Testing & Native Compilation](#testing--native-compilation)
- [License](#license)

---

## Overview

In this challenge, you'll build a **Payment Processing API** that enables the following functionalities:

- **Account Management:** Create, retrieve, update, and delete bank accounts.
- **Transaction Processing:** Handle deposits, withdrawals, and money transfers between accounts.
- **Transaction History:** Query detailed transaction records with filtering and pagination.
- **Reactive Notifications:** Emit and process events for transaction alerts (e.g., low balance or high-value transactions).
- **Observability:** Integrate health checks, metrics, and auto-generated OpenAPI documentation.
- **Optional Native Compilation:** Explore Quarkus’ native image capabilities for fast startup and reduced memory footprint.

This project simulates a real-world payment system, providing an opportunity to work with non-blocking REST endpoints, reactive messaging, and persistence using Hibernate ORM with Panache.

---

## Features

1. **Account Management**
   - **CRUD Operations:** Create, read, update, and delete bank accounts.
   - **Validation:** Ensure valid account creation with initial deposit rules.
   - **Caching:** Optimize read operations for account details.

2. **Transaction Processing**
   - **Deposits & Withdrawals:** Process simple deposit and withdrawal transactions.
   - **Money Transfers:** Transfer funds between accounts with proper validations (e.g., sufficient balance).
   - **Event Emission:** Emit events for each transaction to simulate downstream processing (e.g., fraud detection, notifications).

3. **Transaction History**
   - **Detailed Logs:** Maintain a history of all transactions.
   - **Filtering & Pagination:** Allow querying transactions based on account, date, or transaction type.

4. **Real-Time Notifications**
   - **Reactive Messaging:** Use reactive messaging (e.g., Kafka, AMQP) to notify other systems or trigger alerts when:
     - A transaction exceeds a predefined threshold.
     - An account balance falls below a minimum limit.
   - **Optional Streaming:** Implement Server-Sent Events (SSE) or WebSockets for real-time updates to clients.

5. **Observability & Documentation**
   - **Health Checks:** Expose endpoints (e.g., `/health`, `/health/ready`) to monitor service health.
   - **Metrics:** Publish runtime metrics at `/metrics` for integration with Prometheus or similar systems.
   - **OpenAPI Documentation:** Auto-generate API docs via Swagger UI for ease of integration.

---

## Use Cases

### Use Case 1: Account Management

- **Actor:** Bank Administrator / Customer Onboarding Service
- **Scenario:**  
  An administrator creates a new bank account, retrieves account details, updates account information, or deletes an account.
  
- **Flow Details:**
  - **Create Account:**
    - **Endpoint:** `POST /api/accounts`
    - **Input:** JSON payload with fields like `accountHolderName`, `initialDeposit`, `accountType` (e.g., savings, checking).
    - **Validation:** Ensure the `initialDeposit` is non-negative and meets any minimum requirements.
    - **Outcome:** A new account is created and stored in the database; details may be cached for quick retrieval.
    
  - **Read Account(s):**
    - **Endpoint:** `GET /api/accounts` and `GET /api/accounts/{accountId}`
    - **Details:**  
      - Support pagination and filtering (e.g., by account type).
      - Retrieve account details from cache when available.
      
  - **Update Account:**
    - **Endpoint:** `PUT /api/accounts/{accountId}`
    - **Flow:** Update account information such as contact details.
    
  - **Delete Account:**
    - **Endpoint:** `DELETE /api/accounts/{accountId}`
    - **Flow:** Remove the account from the database and invalidate relevant caches.

### Use Case 2: Transaction Processing (Deposits, Withdrawals, Transfers)

- **Actor:** Account Holder / Banking Service
- **Scenario:**  
  A customer initiates a financial transaction, which may be a deposit, withdrawal, or transfer between accounts.
  
- **Flow Details:**
  - **Initiate Transaction:**
    - **Endpoint:** `POST /api/transactions`
    - **Input:** JSON payload containing:
      - `accountId` (for deposits or withdrawals) or `sourceAccountId` and `targetAccountId` (for transfers)
      - `transactionType` (e.g., DEPOSIT, WITHDRAWAL, TRANSFER)
      - `amount`
    - **Validation:**  
      - For withdrawals and transfers, ensure the source account has sufficient funds.
      - Validate that the transaction amount is positive.
    - **Process:**  
      - Update account balances accordingly.
      - Persist the transaction details using Panache ORM.
      - Emit an event on a messaging channel for further processing (e.g., fraud analysis, notifications).
    - **Response:**  
      - Return a confirmation with a unique `transactionId` and the transaction status.

### Use Case 3: Transaction History Query

- **Actor:** Account Holder / Bank Auditor
- **Scenario:**  
  Retrieve a detailed history of transactions for an account.
  
- **Flow Details:**
  - **Endpoint:** `GET /api/accounts/{accountId}/transactions`
  - **Details:**  
    - Fetch and paginate transaction records.
    - Support filters such as date range and transaction type.
  
### Use Case 4: Fraud Detection & Notifications

- **Actor:** Automated Monitoring Service
- **Scenario:**  
  Monitor transactions to detect suspicious activity or alert on low account balances.
  
- **Flow Details:**
  - **Event Emission:**  
    - After each transaction, evaluate if the transaction meets criteria for a fraud alert (e.g., unusually high amount) or if an account balance falls below a defined threshold.
    - Emit relevant events to a messaging system.
  - **Notification Handling:**  
    - A downstream service (or a simulated consumer) processes these events and triggers alerts (via email, SMS, or dashboard updates).

### Use Case 5: Health, Metrics, and API Documentation

- **Actor:** DevOps / API Consumers
- **Scenario:**  
  Ensure the payment service is healthy, observable, and well-documented.
  
- **Flow Details:**
  - **Health Checks:**
    - **Endpoints:** `/health`, `/health/ready`
    - **Implementation:** Use Quarkus Health extensions to verify database connectivity and messaging system availability.
    
  - **Metrics:**
    - **Endpoint:** `/metrics`
    - **Details:**  
      - Expose runtime metrics (e.g., request counts, error rates, response times) for monitoring and analysis.
      
  - **API Documentation:**
    - **Endpoint:** `/openapi` or via Swagger UI.
    - **Details:**  
      - Auto-generate comprehensive API documentation using Quarkus’ OpenAPI extension.

---

## Technology Stack & Quarkus Extensions

- **RESTEasy Reactive:** For building non-blocking REST endpoints.
- **Hibernate ORM with Panache:** Simplifies CRUD operations and database interactions.
- **Reactive Messaging:** To integrate with messaging platforms (e.g., Kafka, AMQP) for event-driven communication.
- **Caching:** To improve read performance for account and transaction data.
- **Health Checks & Metrics:** For production-readiness and system observability.
- **OpenAPI/Swagger:** For auto-generated API documentation.
- **Native Compilation (Optional):** Use Quarkus native image capabilities for fast startup and low memory footprint.

---

## API Endpoints

Below is an overview of key API endpoints:

### Accounts
- `POST /api/accounts`  
  *Create a new bank account.*

- `GET /api/accounts`  
  *List all bank accounts with optional pagination and filtering.*

- `GET /api/accounts/{accountId}`  
  *Retrieve details of a specific bank account.*

- `PUT /api/accounts/{accountId}`  
  *Update account details.*

- `DELETE /api/accounts/{accountId}`  
  *Delete an account.*

### Transactions
- `POST /api/transactions`  
  *Initiate a transaction (deposit, withdrawal, or transfer).*

- `GET /api/accounts/{accountId}/transactions`  
  *Retrieve transaction history for an account.*

### Observability
- `GET /health`  
  *General health check of the service.*

- `GET /health/ready`  
  *Readiness probe indicating if the service is ready to handle traffic.*

- `GET /metrics`  
  *Expose service metrics.*

- `GET /openapi` or access via Swagger UI  
  *View auto-generated API documentation.*

---

## Architecture & Design Guidelines

- **Modular Structure:**  
  Organize your codebase into packages/modules for **accounts**, **transactions**, **messaging**, and **infrastructure**.
  
- **Reactive & Non-blocking:**  
  Use RESTEasy Reactive for building high-performance endpoints and reactive messaging for event-driven processing.
  
- **Data Persistence:**  
  Leverage Hibernate ORM with Panache to manage CRUD operations with minimal boilerplate code.
  
- **Error Handling:**  
  Implement centralized exception mappers to provide consistent error responses.
  
- **Testing:**  
  Develop comprehensive unit and integration tests using Quarkus testing frameworks.
  
- **Documentation & Observability:**  
  Ensure endpoints are documented with OpenAPI annotations and integrate health and metrics endpoints for monitoring.

---

## Getting Started

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/yourusername/payment-processing-api.git
   cd payment-processing-api
   ```
2. **Build the Project:**
  ```bash
  ./mvnw clean package
  ```
3. **Run the Application in JVM Mode:**
  ```bash
  ./mvnw quarkus:dev
  Access the API Documentation:
  ```
Open your browser and navigate to http://localhost:8080/swagger-ui

## Testing & Native Compilation
- Testing:
  Run unit and integration tests with:
  ```bash
  ./mvnw test
  ```
- Native Compilation (Optional):
  Build a native executable to experience Quarkus' fast startup times and low memory usage:
  ```bash
  ./mvnw package -Pnative
  ./target/payment-processing-api-1.0.0-runner
  ```

## License
This project is provided for educational purposes under the MIT License.
