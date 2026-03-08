# 📚 Books API — Automated API Testing with Postman & Newman

![API Testing](https://img.shields.io/badge/API%20Testing-Postman-orange)
![Newman](https://img.shields.io/badge/CLI-Newman%206.2.2-green)
![Requests](https://img.shields.io/badge/Requests-14-blue)
![Assertions](https://img.shields.io/badge/Assertions-61%20Passing-brightgreen)
![Status](https://img.shields.io/badge/Build-Passing-brightgreen)
![Language](https://img.shields.io/badge/Scripts-JavaScript-yellow)

A complete API test automation project built using **Postman** and **Newman**, covering the full lifecycle of the [Simple Books API](https://simple-books-api.click). This project demonstrates real-world QA Engineering skills including request chaining, environment variables, automated token management, negative testing, and CI/CD-ready test execution.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Tools & Technologies](#tools--technologies)
- [API Under Test](#api-under-test)
- [Test Collection Structure](#test-collection-structure)
- [Test Scenarios](#test-scenarios)
- [Key Features](#key-features)
- [Environment Setup](#environment-setup)
- [How to Run Tests](#how-to-run-tests)
- [Test Results](#test-results)
- [Lessons Learned](#lessons-learned)

---

## 📌 Project Overview

This project tests the **Simple Books API** — a RESTful API that allows users to:
- Check API availability
- Browse and retrieve books
- Submit, manage, and delete orders

The goal was to automate the full API workflow from authentication to order management, covering positive and negative test scenarios, verifying that all endpoints return correct responses and that data flows correctly between requests.

---

## 🛠 Tools & Technologies

| Tool | Version | Purpose |
|---|---|---|
| Postman | Latest | Building and managing API requests |
| Newman | 6.2.2 | Running collections from command line |
| newman-reporter-htmlextra | Latest | Generating HTML test reports |
| JavaScript | ES6 | Writing test scripts and assertions |
| Node.js / npm | Latest | Installing Newman and plugins |
| PowerShell | Windows | Running Newman CLI commands |

---

## 🌐 API Under Test

**Base URL:** `https://simple-books-api.click`

| Endpoint | Method | Description | Auth Required |
|---|---|---|---|
| `/api-clients/` | POST | Register and get access token | ❌ No |
| `/status` | GET | Check API health status | ❌ No |
| `/books` | GET | Get list of all books | ❌ No |
| `/books/:id` | GET | Get a single book by ID | ❌ No |
| `/orders` | POST | Submit a new order | ✅ Yes |
| `/orders` | GET | Get all orders | ✅ Yes |
| `/orders/:id` | GET | Get a single order | ✅ Yes |
| `/orders/:id` | PATCH | Update an order | ✅ Yes |
| `/orders/:id` | DELETE | Delete an order | ✅ Yes |

---

## 📁 Test Collection Structure

```
Books API Collection (14 Requests)
│
├── 📬 Create Token                          → POST /api-clients/
│   ├── Pre-request: Auto-generates random email
│   └── Post-response: Auto-saves access_token
│
├── 📗 Get Book Status                       → GET /status
├── 📗 List of Books                         → GET /books
├── 📗 Get Single Book                       → GET /books/1
├── ❌ Get Single Book - Invalid ID          → GET /books/99999 (Negative Test)
│
├── 📮 Submit Order                          → POST /orders
│   └── Post-response: Auto-saves orderId
│
├── ❌ Submit Order - Invalid Book ID        → POST /orders (Negative Test)
├── 📗 Get All Orders                        → GET /orders
├── 📗 Get Single Order                      → GET /orders/{{orderId}}
├── ❌ Get Single Order - Invalid ID         → GET /orders/invalid-id (Negative Test)
├── 📝 Update Order                          → PATCH /orders/{{orderId}}
├── 🔍 Verify Order Updated                  → GET /orders/{{orderId}}
├── 🗑️  Delete Order                          → DELETE /orders/{{orderId}}
└── ❌ Verify Order Deleted                  → GET /orders/{{orderId}} (Negative Test)
```

---

## 🧪 Test Scenarios

### ✅ Verify Response Status Codes
Every request validates the correct HTTP status code:
```
201 Created    → Create Token, Submit Order
200 OK         → Get Status, List Books, Get Single Book, Get Orders
204 No Content → Update Order, Delete Order
400 Bad Request → Submit Order with invalid book ID
404 Not Found  → Invalid book ID, Invalid order ID, Deleted order
```

### ✅ Validate Response Time
Every request checks performance:
```javascript
pm.test("Response time is under 2000ms", () => {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

### ✅ Validate Response Body Structure
Checks that all required fields exist and are the correct data type:
```javascript
pm.test("Each book has required fields", () => {
    response.forEach((book) => {
        pm.expect(book).to.have.property("id");
        pm.expect(book).to.have.property("name");
        pm.expect(book).to.have.property("type");
        pm.expect(book).to.have.property("available");
    });
});
```

### ✅ Verify Order Creation
Validates the full order creation flow:
```javascript
pm.test("Order created is true", () => {
    pm.expect(response.created).to.be.true;
});
pm.test("Response contains orderId", () => {
    pm.expect(response.orderId).to.not.be.empty;
});
```

### ✅ Verify Order Retrieval
Confirms retrieved data matches what was created:
```javascript
pm.test("Order ID matches saved ID", () => {
    pm.expect(response.id).to.eql(pm.environment.get("orderId"));
});
pm.test("Response contains customerName", () => {
    pm.expect(response.customerName).to.eql("John Doe");
});
```

### ✅ Verify Order Update
Confirms update was applied correctly:
```javascript
pm.test("Customer name was updated correctly", () => {
    pm.expect(response.customerName).to.eql("Lola Updated");
});
pm.test("Book ID remains unchanged after update", () => {
    pm.expect(response.bookId).to.eql(1);
});
```

### ✅ Verify Order Deletion
Confirms deleted order returns 404:
```javascript
pm.test("Status is 404 Not Found after deletion", () => {
    pm.response.to.have.status(404);
});
pm.test("Error message confirms order not found", () => {
    pm.expect(response.error).to.not.be.empty;
});
```

### ✅ Negative Testing with Invalid IDs
Three dedicated negative test scenarios:
- `GET /books/99999` → expects 404
- `POST /orders` with bookId 99999 → expects 400
- `GET /orders/invalid-order-id-99999` → expects 404
- `GET /orders/{{orderId}}` after deletion → expects 404

---

## ⭐ Key Features

### 1. Automatic Token Management
Pre-request script generates a unique email on every run:
```javascript
const randomEmail = "lolatest" + Date.now() + "@gmail.com";
pm.environment.set("clientEmail", randomEmail);
```
Post-response script saves the token automatically:
```javascript
if (pm.response.code === 201) {
    pm.environment.set("access_token", response.accessToken);
}
```

### 2. Request Chaining
Order ID flows automatically from Submit Order to all subsequent requests:
```javascript
if (pm.response.code === 201) {
    pm.environment.set("orderId", response.orderId);
}
```

### 3. Zero Manual Setup Required
Every run is fully automated:
```
Run Newman → fresh email generated → token saved → 
order created → orderId saved → all requests execute → 
HTML report generated
```

### 4. Full CRUD Coverage
```
CREATE  → POST   /orders        (Submit Order)
READ    → GET    /orders        (Get All Orders)
READ    → GET    /orders/:id    (Get Single Order)
UPDATE  → PATCH  /orders/:id    (Update Order)
DELETE  → DELETE /orders/:id    (Delete Order)
```

---

## ⚙️ Environment Setup

### Prerequisites
- [Node.js](https://nodejs.org/) installed
- [Postman](https://www.postman.com/downloads/) installed
- Newman installed globally

### Step 1 — Install Newman
```powershell
npm install -g newman
```

### Step 2 — Install HTML Reporter
```powershell
npm install -g newman-reporter-htmlextra
```

### Step 3 — Verify Installation
```powershell
newman --version
# Expected output: 6.2.2
```

### Step 4 — Clone or Download This Repository
Download `BooksAPI.json` and `QA_Environment.json` to your Desktop.

> **Note:** No manual configuration needed. The `access_token`, `orderId`, and `clientEmail` variables are all populated automatically when the collection runs.

---

## ▶️ How to Run Tests

### Basic Run (Terminal Output Only)
```powershell
newman run "C:\Users\LENOVO\Desktop\BooksAPI.json" -e "C:\Users\LENOVO\Desktop\QA_Environment.json"
```

### Run with HTML Report
```powershell
newman run "C:\Users\LENOVO\Desktop\BooksAPI.json" -e "C:\Users\LENOVO\Desktop\QA_Environment.json" -r htmlextra --reporter-htmlextra-export "C:\Users\LENOVO\Desktop\TestReport.html"
```

### Open the HTML Report
```powershell
Start-Process "C:\Users\LENOVO\Desktop\TestReport.html"
```

> **Note:** A new unique email is automatically generated on every run — no manual changes needed.

---

## ✅ Test Results

### Final Newman Run Summary

| Metric | Result |
|---|---|
| Total Requests | 14 |
| Passed Requests | ✅ 14 |
| Failed Requests | ❌ 0 |
| Total Assertions | 61 |
| Passed Assertions | ✅ 61 |
| Failed Assertions | ❌ 0 |
| Total Duration | 5.9s |
| Average Response Time | 342ms |
| Min Response Time | 156ms |
| Max Response Time | 807ms |

### Individual Request Results

| Request | Method | Status | Assertions |
|---|---|---|---|
| Create Token | POST | 201 Created ✅ | 4 |
| Get Book Status | GET | 200 OK ✅ | 4 |
| List of Books | GET | 200 OK ✅ | 7 |
| Get Single Book | GET | 200 OK ✅ | 8 |
| Get Single Book - Invalid ID | GET | 404 Not Found ✅ | 4 |
| Submit Order | POST | 201 Created ✅ | 5 |
| Submit Order - Invalid Book ID | POST | 400 Bad Request ✅ | 3 |
| Get All Orders | GET | 200 OK ✅ | 5 |
| Get Single Order | GET | 200 OK ✅ | 6 |
| Get Single Order - Invalid ID | GET | 404 Not Found ✅ | 3 |
| Update Order | PATCH | 204 No Content ✅ | 3 |
| Verify Order Updated | GET | 200 OK ✅ | 3 |
| Delete Order | DELETE | 204 No Content ✅ | 3 |
| Verify Order Deleted | GET | 404 Not Found ✅ | 3 |
| **TOTAL** | | | **61 ✅** |

---

## 💡 Lessons Learned

### Technical Challenges Overcome

**1. CloudFront Stripping Authorization Headers**
The `.glitch.me` domain was fronted by AWS CloudFront which stripped Authorization headers before they reached the API. Resolved by switching to the correct domain `simple-books-api.click`.

**2. Environment Variables Not Resolving in Newman**
Early runs failed because `{{baseUrl}}` had no protocol definition in the exported JSON. Fixed by using hardcoded full HTTPS URLs with proper protocol definitions.

**3. Duplicate Authorization Headers**
During debugging, multiple Authorization headers accumulated causing conflicts. Resolved by cleaning the collection JSON directly and using only the Bearer Token auth type.

**4. PowerShell Execution Policy**
Initial Newman installation was blocked by Windows PowerShell execution policy. Fixed using:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**5. Email Already Registered Error**
The API only allows one registration per email. Solved by implementing a pre-request script that generates a unique timestamped email on every run.

### Key QA Skills Demonstrated
- API test design and execution
- Positive and negative test scenario design
- Request chaining and data passing between requests
- Environment and variable management
- Dynamic test data generation
- CLI test execution with Newman
- Debugging real API authentication issues
- Reading and interpreting HTTP status codes
- Professional test reporting with HTMLExtra
- GitHub project documentation

---

## 👩‍💻 Author

**Lola**
QA Engineer
- Specializing in API Test Automation
- Tools: Postman, Newman, JavaScript, Git

---

## 📄 License

This project is open source and available for learning and reference purposes.
*Last updated: March 2026*
*Jenkins CI/CD pipeline configured and running*

