# Deployment Guide: Vercel (Frontend) & Render (Backend)

This repository ([Road-Pothole-Detection-And-Smart-Lane-Analysis](https://github.com/Haariiiii/Road-Pothole-Detection-And-Smart-Lane-Analysis)) is pre-configured for multi-platform cloud deployment using **Vercel** for the user interface and **Render** for the Python Flask backend with YOLO model inference.

---

## 🏗️ Architecture Overview

```
 ┌────────────────────────┐                   ┌─────────────────────────────┐
 │    Vercel Frontend     │  /api/* Proxy /   │       Render Backend        │
 │  (HTML, CSS, JS UI)    │ ────────────────> │  (Flask + Gunicorn + YOLO)  │
 │  https://your.vercel.app│    CORS Fetch     │  https://your-app.onrender  │
 └────────────────────────┘                   └─────────────────────────────┘
```

---

## Part 1: Commit & Push Changes to GitHub

Before creating services on Render and Vercel, push all updated configuration files (`requirements.txt`, `app.py`, `render.yaml`, `Procfile`, `Dockerfile`, `vercel.json`, `index.html`, `static/script.js`) to your GitHub repository:

```bash
git add .
git commit -m "Configure Vercel and Render deployment files"
git push origin main
```

---

## Part 2: Deploy Backend on Render

1. Sign in to **[Render](https://render.com/)**.
2. Click **New +** -> **Web Service**.
3. Connect your GitHub repository: `Haariiiii/Road-Pothole-Detection-And-Smart-Lane-Analysis`.
4. Configure the Web Service details:
   - **Name**: `road-pothole-backend` (or your choice)
   - **Region**: Select nearest region (e.g. Singapore, Frankfurt, Oregon)
   - **Branch**: `main`
   - **Runtime**: `Python 3` (or `Docker` if using containerized build)
   - **Build Command**: `pip install --upgrade pip && pip install -r requirements.txt`
   - **Start Command**: `gunicorn app:app --bind 0.0.0.0:$PORT --workers 1 --threads 4 --timeout 120`
   - **Instance Type**: Free / Starter
5. Click **Create Web Service**.
6. Wait for Render to build the service. Once finished, copy your Render Web Service URL (e.g., `https://road-pothole-backend.onrender.com`).

---

## Part 3: Deploy Frontend on Vercel

1. Sign in to **[Vercel](https://vercel.com/)**.
2. Click **Add New...** -> **Project**.
3. Import your GitHub repository: `Haariiiii/Road-Pothole-Detection-And-Smart-Lane-Analysis`.
4. Configure Project Settings:
   - **Framework Preset**: `Other` / `Static HTML`
   - **Root Directory**: `./`
   - **Build Command**: *(Leave empty)*
   - **Output Directory**: *(Leave empty)*
5. Update `vercel.json` rewrite URL:
   - In `vercel.json`, set the `destination` rule under `/api/:path*` to your Render URL:
     ```json
     {
       "source": "/api/:path*",
       "destination": "https://<YOUR-RENDER-SERVICE-NAME>.onrender.com/api/:path*"
     }
     ```
6. Click **Deploy**.
7. Vercel will build and host your site (e.g. `https://road-pothole-detection.vercel.app`).

---

## Part 4: Testing & Verification

1. Open your Vercel URL in your browser.
2. Select **Upload Video** mode and upload a test dashcam video (`.mp4` / `.avi`).
3. Click **Start Detection**.
4. Verify that:
   - Video frame streaming displays real-time detection boxes (Pothole bounding boxes & Lane boundaries).
   - Stats (Potholes count, Road status, Lane offset) update live.
   - Stop button and Session Summary modal function properly.

---

## 🛠️ Troubleshooting & Notes

- **Render Cold Starts**: On Render's Free tier, Web Services spin down after 15 minutes of inactivity. Initial request may take 30-50 seconds to start.
- **SSE Connection Timeout**: The Gunicorn command uses `--threads 4` so Server-Sent Events (SSE) stream threads don't block incoming HTTP POST requests.
- **Cross-Origin (CORS)**: `app.py` uses `flask-cors` to allow cross-domain requests between Vercel and Render domains.
