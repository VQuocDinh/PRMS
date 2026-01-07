
# PRMS — Patient Record & Recognition Management System

PRMS is a two-part application (backend + frontend) for managing patient records and performing simple face-based recognition against stored patient avatars.

This repository contains:
- backend: Express-based API + utilities and a Python face-recognition helper
- frontend: React + Vite single-page application

---

## Table of contents

- About
- Architecture
- Prerequisites
- Quick start
  - Backend
  - Frontend
  - Face recognition script
- Configuration / Environment variables
- Database notes
- Development notes
- Security & maintenance notes
- License
- Contact

---

## About

PRMS provides a web UI to manage patient data and a facial recognition helper that compares a provided image to stored patient avatar images to identify a matching patient.

Key components found in the repo:
- backend/server.js — Express entry point
- backend/package.json — backend dependencies and scripts
- backend/face_recognition.py — Python script that reads an image, detects a face and compares it against patient avatar images stored in the database
- backend/model.h5 — trained model file (large binary)
- frontend/ — React + Vite application

---

## Architecture

- Frontend: React + Vite (see `frontend/`)
- Backend: Node.js + Express (see `backend/`)
  - Uses Sequelize (and sequelize-auto tooling present) and mysql/mysql2 packages for DB access
  - Server bootstrap is in `backend/server.js` which calls `src/routes/index.js` to register routes
- Face recognition helper: Python + OpenCV script (`backend/face_recognition.py`) that connects to MySQL to read patient avatar file paths and compares histograms for simple similarity checks.

---

## Prerequisites

- Node.js (>= 18 recommended)
- npm
- Python 3.x
  - OpenCV for Python (e.g. pip install opencv-python)
  - numpy
  - mysql-connector-python (or mysql-connector)
- MySQL server (or compatible)
- (Optional) git-lfs if you choose to manage the large model file with LFS

---

## Quick start

Clone repository and switch to project root:

```bash
git clone https://github.com/VQuocDinh/PRMS.git
cd PRMS
```

### Backend

1. Install dependencies:

```bash
cd backend
npm install
```

2. Configure environment variables (see the next section).

3. Start development server (uses nodemon + babel-node):

```bash
npm run server
```

Or start production-like server:

```bash
npm start
```

The server entrypoint is `backend/server.js` and it will listen on the PORT set in environment vars.

### Frontend

1. Install dependencies:

```bash
cd frontend
npm install
```

2. Start dev server:

```bash
npm run dev
```

Open the frontend at the address shown by Vite (usually http://localhost:5173).

### Face recognition script

The repository includes a Python helper which expects a path to an image. From repository root:

```bash
python3 backend/face_recognition.py path/to/photo.jpg
```

- The script will attempt to detect a face and compare it against avatars stored in the database (`patients` table).
- If a match is found (simple histogram correlation threshold in the script), the script prints the `patient_id`.

---

## Configuration / Environment variables

Create a `.env` file for the backend (example):

```
PORT=5000
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_db_password
DB_NAME=PRMS
# add any other env variables your local routes/config expect (JWT secrets, etc.)
```

For frontend, a `.env` file may hold API base URL used by the React app, for example:

```
VITE_API_BASE_URL=http://localhost:5000
```

Notes:
- The Python face recognition script currently contains a hardcoded DB connection snippet. For production, move DB credentials into environment variables and modify the script to read them instead of hardcoding.

---

## Database notes

- The Python helper queries: `SELECT patient_id, avatar FROM patients` — therefore the `patients` table must exist and include at least:
  - `patient_id` (primary key / identifier)
  - `avatar` (string path or URL to the avatar image file readable by the backend/Python environment)

- The backend has Sequelize dependencies and `sequelize-cli` included in devDependencies. If the project uses Sequelize migrations, use the CLI to run them (check `backend/src/migrations` or Sequelize setup).

Example minimal SQL to create the required table (adjust types to your preference):

```sql
CREATE TABLE patients (
  patient_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255),
  avatar VARCHAR(512)
);
```

---

## Development notes

- Backend routes and controllers are under `backend/src/`. Look at `backend/src/routes/index.js` to discover available endpoints.
- The backend uses `nodemon --exec babel-node server.js` for live-reload development; production should use a compiled bundle or `node` with appropriate transpilation.
- The face recognition approach currently uses OpenCV histogram comparison and Haar cascade face detection. This is a basic approach and may produce false positives/negatives — consider using more robust embeddings-based methods (e.g., face embeddings + cosine similarity) for better accuracy.

---

## Security & maintenance notes

- model.h5 is large (~80+ MB). Consider:
  - Moving binary large artifacts to Git LFS or an external storage if you plan to keep it in the project.
- The Python script currently uses hardcoded DB credentials — move these to environment variables and restrict their usage.
- Avoid committing secret keys, DB passwords or other secrets into the repository. Use environment variables or a secret store.
- Sanitize and validate uploaded avatar files to avoid malicious uploads.

---

## Known limitations & TODOs

- The face-matching algorithm is simplistic (histogram correlation). Accuracy may be poor in many real-world scenarios.
- No explicit API documentation is included (OpenAPI/Swagger). Consider adding an API spec for easier frontend/backend integration.
- Database connectivity in the Python script should be refactored to use parameterized config (env-based).
- Consider adding test coverage and CI workflow.

---

## Contributing

1. Fork the repo
2. Create a feature branch
3. Make changes and add tests
4. Open a pull request with a clear description

Please ensure environment secrets are not added to commits.

---

## License

This repository does not contain a license file in the root. Add a LICENSE file (e.g., MIT) if you want to permit reuse. If you want me to add a license, tell me which license to use and I can create one.

---

## Contact

Repository owner: VQuocDinh

If you'd like, I can:
- Draft a smaller README focused only on setup commands,
- Create an example `.env.example` file,
- Extract the DB credentials from the Python helper and refactor it to use environment variables and a shared configuration module.
