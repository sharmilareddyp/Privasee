# Privasee

**Privasee** scans your Google Photos for images that may expose personally identifiable information (PII), so you can review and secure them before they are shared or leaked.

**Live app:** [https://privasee-67838979048.us-west1.run.app](https://privasee-67838979048.us-west1.run.app)

## Features

- **Google Photos integration** — Sign in with Google OAuth and read your photo library (read-only scope).
- **OCR text extraction** — Uses Google Cloud Vision to pull text from images.
- **PII detection** — Uses Google Gemini to classify extracted text (SSN, phone, email, name, address, credit card, and other PII).
- **Sensitive image tracking** — Marks reviewed images in MongoDB so they are not re-scanned on later visits.
- **Dashboard UI** — React frontend to browse flagged photos, view detected values, and mark images as sensitive or not sensitive.

## Tech stack

| Layer      | Technologies |
|-----------|--------------|
| Frontend  | React 19, React Router, Bootstrap, Axios |
| Backend   | Node.js, Express 5, Mongoose |
| APIs      | Google Photos Library, Google Cloud Vision, Google Gemini, Google OAuth 2.0 |
| Database  | MongoDB |

## Project structure

```
Privasee/
├── client/          # React app (login, OAuth callback, dashboard)
├── server/          # Express API, Vision OCR, Gemini PII checks
│   ├── models/      # Mongoose schemas (SensitiveImage)
│   └── server.js
└── README.md
```

## Prerequisites

- Node.js 18+
- MongoDB instance (local or Atlas)
- Google Cloud project with:
  - OAuth 2.0 credentials (Photos Library + profile scopes)
  - Cloud Vision API enabled
  - Service account JSON for Vision (`GOOGLE_APPLICATION_CREDENTIALS`)
- Google Gemini API key
- Google Photos Library API enabled for your OAuth client

## Environment variables

### Server (`server/.env`)

| Variable | Description |
|----------|-------------|
| `MONGO_URI` | MongoDB connection string |
| `GOOGLE_CLIENT_ID` | OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | OAuth client secret |
| `GOOGLE_REDIRECT_URI` | OAuth redirect URL (must match Google Console) |
| `GOOGLE_APPLICATION_CREDENTIALS` | Path to Vision service account JSON |
| `GEMINI_API_KEY` | Gemini API key for PII classification |
| `PORT` | Server port (default `8080`) |

### Client (`client/.env`)

| Variable | Description |
|----------|-------------|
| `REACT_APP_BACKEND_URL` | Backend base URL (e.g. `http://localhost:8080`) |

Never commit `.env` files or service account keys to version control.

## Local development

### 1. Install dependencies

```bash
cd server && npm install
cd ../client && npm install
```

### 2. Configure environment

Create `server/.env` and `client/.env` using the tables above. In Google Cloud Console, set the OAuth redirect URI to your callback route (e.g. `http://localhost:3000/auth/google/callback` for the React app flow).

### 3. Start the backend

```bash
cd server
npm start
```

### 4. Start the frontend

```bash
cd client
npm start
```

Open [http://localhost:3000](http://localhost:3000), connect your Google account, and use the dashboard to review flagged photos.

### Production build (optional)

Build the React app and serve it from Express:

```bash
cd client && npm run build
cp -r build ../server/build
cd ../server && npm start
```

## API overview

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/auth/google` | Start Google OAuth |
| `POST` | `/auth/google/callback` | Exchange auth code for tokens |
| `POST` | `/photos` | Fetch media items (also used from client with token) |
| `POST` | `/ocr` | Run Vision text detection on an image URL |
| `POST` | `/checkSensitiveText` | Classify OCR text with Gemini |
| `POST` | `/mark-sensitive` | Persist sensitive / not-sensitive status |
| `GET` | `/get-marked-images` | List already reviewed image IDs |
| `GET` | `/proxy` | Proxy image bytes for the UI |

## How it works

1. User signs in with Google and grants read access to Google Photos.
2. The dashboard loads up to 500 photos (paginated) and skips images already marked in MongoDB.
3. For each unmarked image, the backend runs OCR via Cloud Vision.
4. Non-empty text is sent to Gemini, which returns a PII type and value or `Not Sensitive`.
5. Flagged images appear on the dashboard with bounding-box hints from OCR.
6. The user can mark each image as sensitive or not sensitive for future scans.

## Deployment

The production app is hosted on **Google Cloud Run**. Deploy the server with the React `build` folder copied into `server/build`, and set all server environment variables in the Cloud Run service configuration.

## License

This project is provided as-is for educational and personal use. Review Google API terms and privacy requirements before processing user photos in production.

## Author

[sharmilareddyp](https://github.com/sharmilareddyp)
