## Bug 1: Server Startup Crash (Environment Incompatibility)
- **Location:** `backend/routes/auth.js` (Line 2) and `backend/scripts/init-db.js` (Line 2)
- **Issue:** The project used the `bcrypt` library, which requires native C++ compilation (via node-gyp). This failed on the local Windows environment with Node v22, causing the server to crash immediately on startup with a "Module not found" or compilation error.
- **Fix:** Replaced the library with `bcryptjs`, a pure JavaScript implementation. This ensures the application is portable and runs correctly across different OS environments without requiring a C++ compiler.

## Bug 2: Login Fails / Missing Database Tables
- **Location:** `database.sqlite` (Missing tables)
- **Issue:** The application showed "Login failed" even with correct credentials. The backend logs showed `SqliteError: no such table: users`. This happened because the initialization script (`init-db.js`) crashed during the previous environment error (Bug 1) and never actually created the tables or the manager user.
- **Fix:** Manually re-ran `npm run init-db` after fixing the dependencies. This successfully created the `users`, `clients`, and `checkins` tables and seeded the default data.

## Bug 3: Employee Dashboard Crash (SQL Syntax Error)
- **Location:** `backend/routes/dashboard.js` (Line 78)
- **Issue:** The Employee Dashboard failed to load with `SqliteError: near "7": syntax error`. This was caused by using MySQL-specific syntax (`DATE_SUB(NOW(), INTERVAL 7 DAY)`) which is incompatible with the SQLite database used in this project.
- **Fix:** Replaced the invalid syntax with the SQLite equivalent `datetime('now', '-7 days')` to correctly filter check-ins from the last week.

## Bug 4: Checkout Failed (SQL Syntax Error)
- **Location:** `backend/routes/checkin.js` (Line 97)
- **Issue:** The checkout operation failed with a server error. The SQL query used double quotes for the string literal `"checked_out"`, which SQLite interprets as a column name, causing a "no such column" error. Additionally, there was a syntax error (double commas `,,`) at the end of the line.
- **Fix:** Changed double quotes to single quotes (`'checked_out'`) and removed the extra comma.

## Bug 5: Check-in Creation Failed (SQL Syntax Error)
- **Location:** `backend/routes/checkin.js` (Line 66)
- **Issue:** Creating a new check-in failed with a server error. The SQL query checking for existing active check-ins used double quotes for `"checked_in"`, which SQLite interprets as a column name instead of a string value.
- **Fix:** Changed the double quotes to single quotes (`'checked_in'`) to correctly query the status column.


## Bug 6: History Page Crash (Frontend State Initialization)
- **Location:** `frontend/src/pages/History.jsx` (Lines 5 & 65)
- **Issue:** The History page crashed with a "White Screen of Death" (TypeError: Cannot read properties of null). This happened because the `checkins` state was initialized as `null`, but the component tried to run the `.reduce()` method on it immediately before data was fetched.
- **Fix:** Changed the state initialization to an empty array `useState([])` and added a safety check `(checkins || []).reduce(...)` to ensure the component never attempts to process a null value.

## Bug 7: Manager Client List Empty (Logic Error)
- **Location:** `backend/routes/checkin.js` (Line 7)
- **Issue:** The manager account (Amit) saw an empty client dropdown when trying to check in. The backend was restricting *all* users to see only "assigned" clients. Since the database initialization script did not explicitly assign clients to the manager, the list returned empty.
- **Fix:** Updated the `GET /clients` route to check the user's role. If the user is a 'manager', the query now returns all clients; otherwise, it restricts the list to assigned clients only.

## Bug 8: Manager Auto-Logout on Check-In (Role Permission Error)
- **Location:** `backend/routes/checkin.js` (Line 60) and Frontend Security Logic
- **Issue:** When a manager attempted to check in, they were immediately logged out. This happened because the backend returned a `403 Forbidden` error (since managers aren't assigned to specific clients). The frontend intercepted this 403 error, assumed the session was invalid/unauthorized, and forced a logout for security.
- **Fix:** Updated the check-in logic to bypass the "client assignment" check for users with the 'manager' role, allowing them to check in at any location without triggering the 403 error.