# 🌾 Agri-Tech: Smart Farming Management System

A full-stack MERN web application that gives farmers digital tools for better decisions: weather forecasts, soil health analysis, crop tracking and pest alerts. The interface is responsive and available in **English and Hindi**.

## Features

- **User authentication**: register and login with JWT; roles for `farmer` and `admin`
- **Farm profiles**: create, view and delete farms with location (lat/lon), area, soil type and irrigation
- **Crop monitoring**: track crops per farm with variety, sowing date and status
- **Soil health analysis**: enter pH, N, P, K and organic carbon to get a health score and recommendations
- **Weather forecast**: current conditions and 5-day forecast from OpenWeatherMap (cached for 30 minutes)
- **Pest alerts**: admin-published alerts matched to the farm's region and crops, plus automatic weather-based warnings
- **Multilingual UI**: English and Hindi, switchable from the navbar

## Tech Stack

| Layer     | Technology                                                    |
|-----------|---------------------------------------------------------------|
| Frontend  | React 18, React Router, Bootstrap 5, Axios, react-i18next     |
| Backend   | Node.js, Express.js, express-validator, Helmet, Morgan        |
| Database  | MongoDB (Atlas or local) with Mongoose                        |
| Auth      | JSON Web Tokens (JWT), bcryptjs                               |
| External  | OpenWeatherMap API                                            |

## Project Structure

```
agri-tech/
├── backend/
│   ├── config/db.js            # MongoDB connection
│   ├── middleware/auth.js      # JWT protect + adminOnly
│   ├── models/                 # User, Farm, Crop, SoilReport, WeatherData,
│   │                           # PestAlert, Order, ActivityLog
│   ├── routes/                 # auth, farms, crops, soil, weather, pests
│   ├── .env.example
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── public/index.html
│   └── src/
│       ├── components/AppNavbar.js
│       ├── pages/              # Login, Register, Dashboard, FarmDetail
│       ├── AuthContext.js
│       ├── ProtectedRoute.js
│       ├── api.js              # Axios instance with token interceptor
│       ├── i18n.js             # English + Hindi translations
│       ├── App.js
│       └── index.js
├── .gitignore
└── README.md
```

## Prerequisites

- **Node.js** v16 or higher (LTS recommended) from https://nodejs.org
- **MongoDB**: either a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster or a local MongoDB install
- **OpenWeatherMap API key** (free) from https://openweathermap.org/api
- **VS Code** (optional) and **Git**

Check your setup:

```bash
node -v
npm -v
```

## Setup

### 1. Get the code

```bash
git clone https://github.com/<your-username>/agri-tech.git
cd agri-tech
```

If you downloaded a ZIP, **extract it first** (right click → Extract All) and open the extracted folder in VS Code. Do not run the project from inside the ZIP.

### 2. Backend

```bash
cd backend
npm install
```

Create a `.env` file (copy from `.env.example`) and fill it in:

```env
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/agritech?retryWrites=true&w=majority
JWT_SECRET=use_a_long_random_string_here
WEATHER_API_KEY=your_openweathermap_key
CLIENT_URL=http://localhost:3000
```

For a local MongoDB, use `MONGO_URI=mongodb://127.0.0.1:27017/agritech`.

Start the server:

```bash
npm run dev
```

You should see:

```
Server running on port 5000
MongoDB connected
```

Health check: http://localhost:5000/api/health

### 3. Frontend

Open a **second terminal**:

```bash
cd frontend
npm install
npm start
```

The app opens at http://localhost:3000. Both terminals must stay running.

### 4. MongoDB Atlas quick guide

1. Create a free **M0** cluster (AWS, Mumbai region works well for India).
2. **Database Access**: create a user with a simple password (letters and numbers only).
3. **Network Access**: add `0.0.0.0/0` (do not mark it temporary). Fine for development; restrict it in production.
4. **Database → Connect → Drivers**: copy the connection string into `MONGO_URI`. Replace `<password>` with your real password and add `/agritech` after `.net`.

## Creating an Admin

Users always register as `farmer`. To make someone an admin, register normally, then in `mongosh` or Atlas (Browse Collections → `users`):

```js
use agritech
db.users.updateOne({ email: "you@example.com" }, { $set: { role: "admin" } })
```

Admins can publish pest alerts through the API (see below).

## API Reference

All routes except register and login need the header `Authorization: Bearer <token>`.

| Method | Endpoint                   | Description                                  |
|--------|----------------------------|----------------------------------------------|
| POST   | `/api/auth/register`       | Create account, returns token                |
| POST   | `/api/auth/login`          | Login, returns token                         |
| GET    | `/api/auth/me`             | Current user                                 |
| GET    | `/api/farms`               | List own farms                               |
| POST   | `/api/farms`               | Create farm                                  |
| GET    | `/api/farms/:id`           | Get one farm                                 |
| PUT    | `/api/farms/:id`           | Update farm                                  |
| DELETE | `/api/farms/:id`           | Delete farm                                  |
| GET    | `/api/crops?farm=<id>`     | List crops (optionally by farm)              |
| POST   | `/api/crops`               | Add crop                                     |
| PUT    | `/api/crops/:id`           | Update crop                                  |
| DELETE | `/api/crops/:id`           | Delete crop                                  |
| POST   | `/api/soil`                | Add soil report (score and tips computed)    |
| GET    | `/api/soil/farm/:farmId`   | Soil reports for a farm                      |
| GET    | `/api/weather/:farmId`     | Current weather and 5-day forecast           |
| GET    | `/api/pests/farm/:farmId`  | Alerts and weather warnings for a farm       |
| POST   | `/api/pests`               | **Admin**: publish a pest alert              |
| DELETE | `/api/pests/:id`           | **Admin**: deactivate an alert               |

Example, publishing a pest alert as admin:

```bash
curl -X POST http://localhost:5000/api/pests \
  -H "Authorization: Bearer <admin-token>" -H "Content-Type: application/json" \
  -d '{"crop":"wheat","pest":"Yellow rust","severity":"high","state":"Madhya Pradesh","message":"Spray recommended fungicide at first sign of yellow stripes."}'
```

## Database Design

```
User 1 ─── ∞ Farm 1 ─── ∞ Crop
                 ├────── ∞ SoilReport
                 └────── ∞ WeatherData
User 1 ─── ∞ Order
User 1 ─── ∞ ActivityLog
PestAlert (published by admins, matched by crop + region)
```

Collections are linked with `ObjectId` references. Every farmer query is filtered by `owner`, so users only see their own data.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `npm is not recognized` | Install Node.js LTS, then fully close and reopen VS Code |
| `Cannot find path ...\backend\backend` | The terminal is already inside `backend`. Skip `cd backend` |
| Terminal fails to start in a Temp folder | You opened the project from inside the ZIP. Extract it and open the real folder |
| `MongoDB connection error: ECONNREFUSED 127.0.0.1` | No local MongoDB running. Start it or use Atlas |
| `querySrv EREFUSED` | DNS issue. Add the two lines below at the top of `server.js`, or change your DNS to 8.8.8.8 |
| `Weather service unavailable` | Missing or not-yet-active `WEATHER_API_KEY` (new keys can take up to a couple of hours) |
| `.env` changes not applied | Restart the backend server |
| Atlas connection suddenly stops | Check that your Network Access entry is still active and not temporary |

DNS fix for `querySrv EREFUSED`, placed at the very top of `backend/server.js`:

```js
const dns = require('dns');
dns.setServers(['8.8.8.8', '1.1.1.1']);
```

## Security Notes

- Never commit `.env`. It is already listed in `.gitignore`.
- Use a long random `JWT_SECRET` in production.
- The soil scoring thresholds are a simple starting point. Replace them with values from ICAR or your state agriculture department before real-world use.
- Before deploying: restrict `CLIENT_URL` (CORS), add rate limiting on `/api/auth` (for example `express-rate-limit`), and limit Atlas access to your server's IP.
- `npm audit` warnings in the frontend come from Create React App's older tooling. Do not run `npm audit fix --force`.

## Roadmap

- Admin panel for publishing pest alerts from the UI
- Orders and marketplace module for seeds, fertilizers and equipment
- More regional languages
- SMS or push notifications for severe alerts
- Expert consultation and chat
- Migration from Create React App to Vite

## License

For educational use. Add a license of your choice before publishing.
