# Technical Questions & Answers

### 1. If this app had 10,000 employees checking in simultaneously, what would break first? How would you fix it?
**The Breaking Point:**
The **Database** would likely be the first bottleneck.
- **SQLite Limitations:** Currently, the app uses SQLite, which is a file-based database. It handles concurrent reads well but can only handle one write operation at a time (locking the file). 10,000 simultaneous check-ins (writes) would cause a massive queue, leading to timeouts (`SQLITE_BUSY` errors) and server crashes.
- **Connection Limits:** Even if migrated to a server-based DB like MySQL, the default connection pool size (often 10-100 connections) would be instantly exhausted, causing requests to hang and fail.

**The Fix:**
1.  **Migrate Database:** Switch from SQLite to a robust RDBMS like **PostgreSQL** or **MySQL** designed for high concurrency.
2.  **Connection Pooling:** Implement a connection pooler (like PgBouncer) to manage and reuse database connections efficiently, preventing the "too many connections" error.
3.  **Queueing System:** Introduce a message queue (like RabbitMQ or Redis). When a user checks in, the API immediately accepts the request and pushes it to a queue. A background worker process then reads from the queue and inserts data into the database at a controlled pace, smoothing out the traffic spike.

---

### 2. The current JWT implementation has a security issue. What is it and how would you improve it?
**The Issue:**
The current implementation likely stores the JWT in the browser's `localStorage`.
- **XSS Vulnerability:** `localStorage` is accessible by any JavaScript running on the page. If the site is vulnerable to Cross-Site Scripting (XSS)—for example, if a malicious script is injected via a third-party library or user input—an attacker can easily read the token from local storage and impersonate the user.
- **No Expiration Strategy:** If the token lacks a short expiration time or a refresh mechanism, a stolen token remains valid indefinitely.

**The Improvement:**
1.  **HttpOnly Cookies:** Store the JWT in an `HttpOnly`, `Secure` cookie. These cookies are sent automatically with requests but cannot be accessed by client-side JavaScript, effectively neutralizing XSS attacks against the token.
2.  **Short-Lived Tokens:** Set a short expiration time for the Access Token (e.g., 15 minutes) and use a separate Refresh Token (stored securely) to obtain new access tokens. This limits the damage window if a token is compromised.

---

### 3. How would you implement offline check-in support? (Employee has no internet, checks in, syncs later)
**Implementation Strategy:**
I would use a **"Store and Forward"** pattern utilizing the browser's Service Workers and local storage.

1.  **Detection:** The frontend code detects network status using `navigator.onLine`.
2.  **Local Storage (IndexedDB):** If offline, instead of calling the API, the check-in data (Timestamp, GPS Coordinates, Client ID) is saved locally to `IndexedDB` (which is more robust than localStorage).
3.  **Background Sync:** Register a Service Worker with the **Background Sync API**. When the device regains connectivity, the Service Worker wakes up (even if the tab is closed) and attempts to send the queued requests.
4.  **Fallback:** As a backup, add an event listener for `window.addEventListener('online', ...)` to trigger a "sync" function that loops through stored check-ins and uploads them to the backend when the internet returns.

---

# Theory/Research Questions

### 4. Explain the difference between SQL and NoSQL databases. For this Field Force Tracker application, which would you recommend and why?
**Differences:**
- **SQL (Relational):** Stores data in tables with strict schemas and relationships (rows/columns). It guarantees ACID properties (Atomicity, Consistency, Isolation, Durability) and excels at complex queries using JOINs.
- **NoSQL (Non-Relational):** Stores data in flexible formats (documents, key-value pairs, graphs). It is easier to scale horizontally but often trades immediate consistency for performance.

**Recommendation: SQL**
I would strongly recommend a **SQL database** (like PostgreSQL or MySQL) for this application.
- **Structured Data:** The data model is highly relational: *Managers* manage *Employees*, who visit *Clients* and create *Check-ins*. SQL handles these relationships natively.
- **Data Integrity:** For attendance and payroll-related data, accuracy is critical. SQL's ACID compliance ensures that a check-in isn't half-saved or lost during a crash.
- **Reporting:** Generating reports (e.g., "Total hours worked by Employee X at Client Y") requires complex JOINs and aggregations, which SQL is optimized to perform efficiently.

---

### 5. What is the difference between authentication and authorization? Identify where each is implemented in this codebase.
**Authentication (AuthN):**
*Definition:* Verifying **who** the user is.
*Codebase Location:* Implemented in the **Login Route** (`POST /auth/login`). Here, the system compares the provided password with the stored hash to confirm identity and issues a token.

**Authorization (AuthZ):**
*Definition:* Verifying **what** the user is allowed to do.
*Codebase Location:* Implemented in the **Middleware** (`authenticateToken` in `middleware/auth.js`). It checks the valid token to grant access. It is also found in specific route logic, such as:
- **Role Check:** `if (req.user.role === 'manager')` (Checking if user has manager privileges).
- **Assignment Check:** Checking `employee_clients` table to see if an employee is authorized to visit a specific client.

---

### 6. Explain what a race condition is. Can you identify any potential race conditions in this codebase? How would you prevent them?
**Definition:**
A race condition happens when the system's behavior depends on the sequence or timing of uncontrollable events, such as two signals arriving at the same time.

**Potential Race Condition:**
**"Double Check-in":** If a user clicks the "Check In" button twice very quickly (or uses two devices simultaneously), two requests hit the server at the same time.
1. Request A queries DB: "Is there an active check-in?" -> DB says "No".
2. Request B queries DB: "Is there an active check-in?" -> DB says "No" (Request A hasn't finished writing yet).
3. Request A inserts check-in.
4. Request B inserts check-in.
*Result:* The employee has two active check-ins open simultaneously, breaking the data logic.

**Prevention:**
1.  **Database Constraints:** Add a **Unique Partial Index** in the database: `CREATE UNIQUE INDEX unique_active_checkin ON checkins (employee_id) WHERE status = 'checked_in';`. The database will reject the second insert with an error.
2.  **Frontend Locking:** Disable the "Check In" button immediately after the first click (`submitting` state) to prevent double submission.