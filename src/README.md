# Photo Motion AI

Full-stack starter for an Image-to-Video AI mobile product.

## Stack

- Mobile: Expo + React Native + TypeScript + Zustand
- Backend: FastAPI + Python
- Architecture: Thin routes, service-layer business logic, provider abstraction for pluggable AI vendors

## Project Structure

```text
photo-motion-ai/
  mobile/
  backend/
```

## Backend Setup

1. Go to backend folder:

```bash
cd backend
```

2. Create and activate virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Create env file:

```bash
cp .env.example .env
```

5. Run API:

```bash
uvicorn main:app --reload --port 8000
```

Production-style start command (Render-compatible):

```bash
uvicorn main:app --host 0.0.0.0 --port ${PORT:-8000}
```

Health endpoint:

```bash
GET http://127.0.0.1:8000/health
```

## Mobile Setup

1. Go to mobile folder:

```bash
cd mobile
```

2. Install dependencies:

```bash
npm install
```

3. Run Expo app:

```bash
npm run start
```

4. Open iOS simulator, Android emulator, or Expo Go.

## API Endpoints

- `POST /upload-image`
- `POST /generate-video`
- `GET /jobs/{job_id}`
- `GET /videos`
- `GET /videos/{video_id}`

## Backend Tests

Run backend tests from CI or any non-local environment:

```bash
cd backend
pip install -r requirements.txt
pytest -q
```

Included test coverage:

- health endpoints (`/` and `/health`)
- upload image endpoint
- generation lifecycle (queued -> processing -> completed)
- list and fetch generated videos
- invalid image id error path

## Render Deployment

Backend is configured for Render deployment via [src/backend/render.yaml](src/backend/render.yaml).

1. Push repository to GitHub.
2. In Render, create a new Blueprint service (or Web Service).
3. If using Web Service manually, set:
- Root Directory: `src/backend`
- Build Command: `pip install -r requirements.txt`
- Start Command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
4. Set environment variables:
- `APP_ENV=production`
- `CORS_ORIGINS=https://your-mobile-web-domain.example`
- `STORAGE_DIR=/tmp/photo-motion-ai-data`
- `MOCK_PROVIDER_DELAY_SECONDS=2`
- `LOG_LEVEL=info`
5. Deploy and verify:
- `GET /health`
- `GET /`

Notes for Render:

- `/tmp` is used for writable ephemeral storage in the sample config.
- Current storage is in-memory + local temp files. For production persistence, switch to object storage and a database.

## Example Flow

### 1) Upload image

Request (`multipart/form-data`):
- field name: `file`
- file type: jpg/png/webp

Response:

```json
{
  "image_id": "img_9ce6e8505aef",
  "filename": "portrait.jpg",
  "content_type": "image/jpeg",
  "image_url": "/absolute/path/to/backend/data/uploads/img_9ce6e8505aef.jpg",
  "created_at": "2026-04-01T08:00:00.000000Z"
}
```

### 2) Create generation job

Request:

```json
{
  "image_id": "img_9ce6e8505aef",
  "prompt": "Make hair move softly with cinematic camera push-in",
  "duration_seconds": 4,
  "aspect_ratio": "9:16"
}
```

Response:

```json
{
  "job": {
    "id": "job_3558ef283611",
    "status": "queued",
    "prompt": "Make hair move softly with cinematic camera push-in",
    "duration_seconds": 4,
    "aspect_ratio": "9:16",
    "image_id": "img_9ce6e8505aef",
    "created_at": "2026-04-01T08:00:05.000000Z",
    "updated_at": "2026-04-01T08:00:05.000000Z",
    "video_id": null,
    "error": null
  }
}
```

### 3) Poll job status

Response while running:

```json
{
  "id": "job_3558ef283611",
  "status": "processing",
  "prompt": "Make hair move softly with cinematic camera push-in",
  "duration_seconds": 4,
  "aspect_ratio": "9:16",
  "image_id": "img_9ce6e8505aef",
  "created_at": "2026-04-01T08:00:05.000000Z",
  "updated_at": "2026-04-01T08:00:07.000000Z",
  "video_id": null,
  "error": null
}
```

Response when completed:

```json
{
  "id": "job_3558ef283611",
  "status": "completed",
  "prompt": "Make hair move softly with cinematic camera push-in",
  "duration_seconds": 4,
  "aspect_ratio": "9:16",
  "image_id": "img_9ce6e8505aef",
  "created_at": "2026-04-01T08:00:05.000000Z",
  "updated_at": "2026-04-01T08:00:10.000000Z",
  "video_id": "vid_68ca6f01ff4a",
  "error": null
}
```

### 4) Fetch generated video

```json
{
  "id": "vid_68ca6f01ff4a",
  "source_image_id": "img_9ce6e8505aef",
  "prompt": "Make hair move softly with cinematic camera push-in",
  "duration_seconds": 4,
  "aspect_ratio": "9:16",
  "video_url": "https://storage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4",
  "created_at": "2026-04-01T08:00:10.000000Z"
}
```

## Mock Provider Design

- Provider contract: `services/provider_base.py`
- Local mock provider: `services/provider_mock.py`
- Orchestration service: `services/video_service.py`

## TODO Hooks For Real Providers

- Replace `MockVideoProvider.generate_video` with real SDK/API calls (Runway, fal, Replicate)
- Add webhook callback route to mark jobs completed instead of polling-only approach
- Move storage from in-memory/local files to PostgreSQL + object storage (S3/GCS)
- Add auth and credits middleware before generation endpoints

## Product Screens Included

- HomeScreen
- CreateVideoScreen
- ProgressScreen
- PreviewScreen
- MyVideosScreen
- SettingsScreen

## Notes

- The mobile app uses `extra.apiBaseUrl` from `app.json`. For physical devices, replace `127.0.0.1` with your machine LAN IP.
- Generated videos are mock outputs so app behavior can be tested immediately.
