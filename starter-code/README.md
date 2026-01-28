# Unolo Field Force Tracker

A full-stack web application for tracking field employee check-ins at client locations. This project allows employees to check in with location tracking and provides managers with a live dashboard to oversee team activity.

## Tech Stack

- **Frontend:** React 18, Vite, Tailwind CSS, React Router
- **Backend:** Node.js, Express.js, SQLite
- **Authentication:** JWT (JSON Web Tokens)
- **Database:** SQLite (better-sqlite3)
- **Password Hashing:** bcryptjs

## Key Features

✅ Employee Check-in/Check-out tracking  
✅ Location-based tracking with GPS  
✅ Manager Dashboard with team analytics  
✅ Employee Dashboard with assigned clients  
✅ Check-in History with date filtering  
✅ Client assignment management  
✅ Secure JWT authentication  
✅ Role-Based Access Control (Manager/Employee)  
✅ Responsive Tailwind CSS design  
✅ Real-time activity tracking  

---

## Quick Start

### Prerequisites

- Node.js v22+ and npm
- Python 3.8+ (for better-sqlite3)

### 1. Backend Setup

```bash
cd starter-code/backend
npm install
npm run init-db
npm run dev
```

Server runs on: **http://localhost:3001**

### 2. Frontend Setup

```bash
cd starter-code/frontend
npm install
npm run dev
```

Frontend runs on: **http://localhost:5173**

---

## Test Credentials

| Role     | Email                  | Password    | Permissions                           |
|----------|------------------------|-------------|---------------------------------------|
| Manager  | manager@unolo.com      | password123 | View all team activity, dashboard     |
| Employee | rahul@unolo.com        | password123 | Check-in, view assigned clients       |
| Employee | priya@unolo.com        | password123 | Check-in, view assigned clients       |
| Employee | vikram@unolo.com       | password123 | Check-in, view assigned clients       |

---

## API Documentation

### Authentication
- `POST /api/auth/login` - Authenticate user and receive JWT
- `GET /api/auth/me` - Retrieve current user profile

### Check-in Operations
- `GET /api/checkin/clients` - Fetch assigned clients
- `POST /api/checkin` - Create a new check-in
- `PUT /api/checkin/checkout` - Check out from current location
- `GET /api/checkin/active` - Get active check-in status
- `GET /api/checkin/history` - Retrieve check-in history with date filtering

### Dashboard
- `GET /api/dashboard/stats` - Manager dashboard statistics
- `GET /api/dashboard/employee` - Employee dashboard data

---

## Project Structure

```
starter-code/
├── backend/
│   ├── config/
│   │   └── database.js
│   ├── middleware/
│   │   └── auth.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── checkin.js
│   │   └── dashboard.js
│   ├── scripts/
│   │   └── init-db.js
│   ├── package.json
│   ├── server.js
│   └── .env
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── utils/
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── package.json
│   ├── vite.config.js
│   └── index.html
└── README.md
```

---

## Architecture & Decisions

### Database (SQLite)
- Chosen for portability and ease of setup
- All SQL queries optimized for SQLite compatibility
- Uses `better-sqlite3` v11.8.1+ for Windows Node.js v22 support

### Security
- Replaced `bcrypt` with `bcryptjs` for cross-platform compatibility
- Strict role-based access control in middleware
- JWT tokens expire after 24 hours
- Password hashing with bcryptjs salt rounds: 10

### API Design
- Role-based filtering for client endpoints
- Data isolation between employees and managers
- Consistent JSON response format across all endpoints

---

## Bug Fixes Applied

✅ Fixed bcrypt compilation error (replaced with bcryptjs)  
✅ Fixed SQLite syntax errors (NOW() → CURRENT_TIMESTAMP)  
✅ Fixed date calculation queries (SQLite compatible)  
✅ Corrected authentication flow  
✅ Fixed async/await function declarations  

---

## Environment Variables

Create a `.env` file in the `backend` folder:

```
JWT_SECRET=your-secret-key-here
PORT=3001
NODE_ENV=development
```

---

## Troubleshooting

**Database not initializing?**
```bash
npm run init-db
```

**Port already in use?**
Change the port in `.env` file or kill the process using the port.

**Frontend can't connect to backend?**
Ensure backend is running on `http://localhost:3001`

**Fresh database reset?**
```bash
rm database.sqlite
npm run init-db
```

---

## Notes

- The application uses GPS coordinates. For testing, default locations (Gurugram coordinates) are provided
- All timestamps are in UTC
- Check-in history can be filtered by date range
- Manager can view all team members' activities in real-time
