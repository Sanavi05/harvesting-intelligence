# Harvesting Intelligence

Harvesting Intelligence is an agriculture-focused web app that brings together:

- a Node/Express authentication and upload backend
- a crop rotation recommender API
- a companion-cropping recommender API
- a browser-based frontend for navigating the tools

The UI also references crop-disease services for wheat, rice, and maize on ports `5003`, `5004`, and `5005`, but the server code for those services is not included in this repository.

## Project layout

```text
backend/               Express API for auth and image uploads
companion_cropping/    Flask API for companion planting recommendations
crop_rotation/         Flask API for crop rotation recommendations
frontend/              HTML/CSS/JS frontend
```

## Features

- User registration and login with JWT-based auth
- Image upload endpoint protected by JWT
- Crop rotation recommendations from a CSV dataset
- Companion planting recommendations from a relationship graph
- Frontend landing page with navigation to each feature

## Service map

| Service | Path | Port | Notes |
|---|---|---:|---|
| Backend API | `backend/index.js` | `5000` | `/auth/*`, `/image/upload` |
| Crop rotation API | `crop_rotation/rotation_app.py` | `5001` | `/get_rotation?crop=` |
| Companion cropping API | `companion_cropping/app.py` | `5002` | `/recommend?plant=` |
| Frontend dev server | `frontend/` | `5173` | Vite dev server |
| Crop disease service (referenced) | not present | `5003` | Wheat upload flow |
| Crop disease service (referenced) | not present | `5004` | Maize landing page |
| Crop disease service (referenced) | not present | `5005` | Rice landing page |

## Prerequisites

- Node.js 18+ (Node 22 works with the current dependencies)
- npm
- Python 3.10+
- PostgreSQL

## Environment variables

### Backend

Set these in a `.env` file at the repository root:

- `PORT` (defaults to `5000`)
- `DB_USER`
- `DB_HOST`
- `DB_PORT`
- `DB_PASSWORD`
- `DB_AUTH_NAME`
- `JWT_SECRET`
- `JWT_EXPIRES_IN`

### Frontend

An optional `frontend/.env.local` file is present for local hosting experiments, but the current Vite workflow does not require it.

The app code itself uses direct `localhost` URLs and the Vite dev server runs on `5173`.

## Setup

### 1) Install root dependencies

```bash
npm install
```

### 2) Create the PostgreSQL tables

The backend expects at least:

- `users` table
- `images` table

The code inserts the following fields:

- `users`: `name`, `email`, `password`, `role`
- `images`: `user_id`, `filename`

### 3) Set up the Python environments

Crop rotation:

```bash
cd crop_rotation
python3 -m venv .venv
source .venv/bin/activate
pip install -r rotation_requirements.txt
```

Companion cropping:

```bash
cd ../companion_cropping
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Running the project

Run each service in its own terminal.

### Backend

```bash
cd backend
node index.js
```

### Crop rotation API

```bash
cd crop_rotation
python3 rotation_app.py
```

### Companion cropping API

```bash
cd companion_cropping
python3 app.py
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open the Vite URL shown in the terminal, usually:

```text
http://localhost:5173
```

## Main routes and endpoints

### Frontend pages

- `frontend/index.html` — landing page
- `frontend/login.html` — sign in / sign up page
- `frontend/rotation_frontend/rotation.html` — crop rotation UI
- Companion cropping UI — available from the companion page in the frontend folder

### Backend

- `POST /auth/register`
- `POST /auth/login`
- `POST /image/upload`  
  Requires an Authorization header with a bearer token and a multipart form field named `image`.

### Crop rotation API

- `GET /get_rotation?crop=<crop-name>`

### Companion cropping API

- `GET /recommend?plant=<plant-name>`

## How the flows work

1. A user registers or logs in through the frontend.
2. The backend returns a JWT on successful login.
3. The token is stored in `localStorage`.
4. Protected upload requests send the token in the `Authorization` header.
5. Crop rotation and companion planting pages call their Flask APIs directly.

## Data sources

- `crop_rotation/data/Crop_Rotation_Dataset.csv`
- `companion_cropping/data/companion_plants.csv`

## Notes

- The frontend currently points crop-disease buttons to ports `5003`, `5004`, and `5005`, but those servers are not part of this repository.
- The companion-cropping app serves HTML from a configured static folder; if you move files around, keep that path aligned with the frontend directory layout.
- There are no automated tests defined in the current `package.json` files.
