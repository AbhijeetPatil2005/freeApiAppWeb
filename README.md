# Doctor Appointment Website

This repository contains a full-stack Doctor Appointment application built with the following parts:

- backend — Express + Node.js + MongoDB (API & file uploads via Cloudinary)
- frontend — React + Vite (patient-facing website)
- admin — React + Vite (admin dashboard)

The project enables users to register/login, view doctors, book and cancel appointments. Admins can add doctors, view appointments and manage availability.

## Table of contents

- Project structure
- Tech stack
- Quick start (development)
- Environment variables
- API reference (summary)
- Database models (key fields)
- Frontend / Admin notes
- Troubleshooting
- Contributing


## Project structure (important folders)

Root folders:

- `backend/` — Express API, controllers, models, and middleware
- `frontend/` — React (patient-facing) app
- `admin/` — React (admin) app

Backend highlights (important files):

- `backend/server.js` — main Express server and route registration
- `backend/config/mongodb.js` — MongoDB connection (uses `MONGODB_URI`)
- `backend/config/cloudinary.js` — Cloudinary configuration (uses Cloudinary env vars)
- `backend/routes/` — API routes: `adminRoute.js`, `doctorRoute.js`, `userRoute.js`
- `backend/controllers/` — route handlers for admin/doctor/user functionality
- `backend/models/` — Mongoose models: `doctorModel.js`, `userModel.js`, `appointmentModel.js`


## Tech stack

- Backend: Node.js, Express, MongoDB (Mongoose)
- Frontend / Admin: React (Vite), Axios, React Router, TailwindCSS
- Auth: JSON Web Tokens (JWT)
- File storage: Cloudinary


## Prerequisites

- Node.js (16+ recommended)
- npm
- A MongoDB instance (Atlas cluster or local)
- A Cloudinary account (for images)


## Environment variables

Create a `.env` file in the `backend/` folder with the following variables:

- `MONGODB_URI` — the base MongoDB connection string (without the DB name). The code appends `/prescripto` to it. Example:
	- MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.abcd.mongodb.net
- `CLOUDINARY_NAME` — your Cloudinary cloud name
- `CLOUDINARY_API_KEY` — your Cloudinary API key
- `CLOUDINARY_SECRET_KEY` — your Cloudinary API secret
- `JWT_SECRET` — secret string for signing JWTs
- `ADMIN_EMAIL` — admin login email (used by admin login endpoint)
- `ADMIN_PASSWORD` — admin login password
- `PORT` — optional server port (default 4000)

Example `backend/.env` (DO NOT commit credentials):

```powershell
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net
CLOUDINARY_NAME=your_cloud_name
CLOUDINARY_API_KEY=1234567890
CLOUDINARY_SECRET_KEY=abcdefg
JWT_SECRET=someVerySecretString
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=adminpassword
PORT=4000
```

Frontend and Admin applications use Vite environment variables. Add a `.env` file in each of `frontend/` and `admin/` with:

```powershell
VITE_BACKEND_URL=http://localhost:4000
```

(Change the backend URL to your production API when deploying.)


## Quick start (development)

Open three terminals (or use a process manager) and run the backend, frontend and admin apps:

1) Backend

```powershell
cd backend
npm install
npm run server    # nodemon server.js (development)
```

This starts the API (default port 4000). Visit http://localhost:4000 to check it responds.

2) Frontend (patient site)

```powershell
cd frontend
npm install
npm run dev
```

By default Vite opens at http://localhost:5173 (check terminal output).

3) Admin (dashboard)

```powershell
cd admin
npm install
npm run dev
```

By default Vite opens at http://localhost:5174 (check terminal output).


## API summary (quick reference)

Base URL: {VITE_BACKEND_URL} or http://localhost:4000

Headers
- User authenticated endpoints expect header `token: <JWT>` (sent after user login)
- Admin authenticated endpoints expect header `atoken: <JWT>` (sent after admin login)

Routes (important ones)

Admin routes (`/api/admin`):
- POST `/login` — body: { email, password } -> returns { success, token } (store as `aToken`)
- POST `/add-doctor` — headers: { atoken }, multipart form data: image, plus fields: name, email, password, speciality, degree, experience, about, fees, address (address is JSON string). Adds doctor (uploads image to Cloudinary).
- POST `/all-doctors` — headers: { atoken } -> returns doctor list (admin view)
- POST `/change-availability` — headers: { atoken }, body: { docId } -> toggles doctor's availability
- GET `/appointments` — headers: { atoken } -> list all appointments
- POST `/cancel-appointment` — headers: { atoken }, body: { appointmentId } -> cancel appointment (admin)
- GET `/dashboard` — headers: { atoken } -> admin dashboard counts

Doctor routes (`/api/doctor`):
- GET `/list` — public -> returns list of doctors (frontend uses this)

User routes (`/api/user`):
- POST `/register` — body: { name, email, password } -> registers user and returns token
- POST `/login` — body: { email, password } -> returns token
- GET `/get-profile` — headers: { token } -> returns user profile (expects body.userId set by middleware)
- POST `/update-profile` — headers: { token }, optionally image file + fields: userId, name, phone, address(JSON string), dob, gender -> updates profile
- POST `/book-appointment` — headers: { token }, body: { userId, docId, slotDate, slotTime } -> books appointment
- GET `/appointments` — headers: { token } -> list user's appointments (expects userId in body via middleware)
- POST `/cancel-appointment` — headers: { token }, body: { userId, appointmentId } -> cancels appointment (user must match appointment)

Notes:
- Admin login verifies credentials against `ADMIN_EMAIL` and `ADMIN_PASSWORD` from the backend `.env` and signs a token containing (email+password) with `JWT_SECRET`. When calling protected admin endpoints include returned token in header `atoken`.
- User login returns JWT signed with `{ id: user._id }`. Include it in header `token` for protected user routes.


## Database models (key fields)

doctorModel (important fields):
- name, email, password, image, speciality, degree, experience, about, available (boolean), fees, address (object), slots_booked (object)

userModel (important fields):
- name, email, password, image, address, gender, dob, phone

appointmentModel (important fields):
- userId, docId, slotDate, slotTime, userData, docData, amount, cancelled (boolean), payment (boolean)


## Frontend / Admin notes

- Both frontends use `import.meta.env.VITE_BACKEND_URL` to build API requests (set `VITE_BACKEND_URL` in each frontend's `.env`).
- The frontend stores the user token in `localStorage` as `token`. Admin stores admin token as `aToken` in `localStorage`.
- File uploads use `multipart/form-data` through the `multer` middleware on the backend and the backend uploads to Cloudinary.


## Troubleshooting & common pitfalls

- MongoDB connection: backend connects to `${process.env.MONGODB_URI}/prescripto`. Make sure your `MONGODB_URI` is correct and that your IP/Network settings allow connections.
- Cloudinary: ensure `CLOUDINARY_NAME`, `CLOUDINARY_API_KEY` and `CLOUDINARY_SECRET_KEY` are valid. Image uploads will fail without them.
- JWT errors: If you get "Not Authorized Login Again" responses, verify you pass `token` or `atoken` header exactly as required and that `JWT_SECRET` matches the one used to sign tokens.
- CORS: the backend enables CORS by default for development. If deploying to production, restrict origins appropriately.


## Tests & quality

There are no automated tests in the repo. Linting scripts exist on frontend/admin (`npm run lint`). Backend has no tests configured.


## Deploying to production (short notes)

- Build frontend and admin:

```powershell
cd frontend
npm run build

cd ../admin
npm run build
```

- Serve the built static files using a static host (Netlify, Vercel), or serve them from a Node/Express static directory. Configure `VITE_BACKEND_URL` in production to point to your deployed API.


## Contributing

If you want to contribute:

1. Fork the repository
2. Create a branch for your feature or bugfix
3. Add tests where appropriate
4. Open a pull request with a clear description


## Contact / Notes

If anything in this README seems incorrect or incomplete, open an issue describing what's missing and attach logs or screenshots when relevant.

---

Generated and updated to reflect the current repository layout and server/client expectations.

