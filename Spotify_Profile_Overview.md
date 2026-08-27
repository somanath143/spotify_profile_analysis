# Spotify Profile – Interview‑Ready Overview

---

## 1️⃣ Project Snapshot

- **Goal** – Visualise a user’s personal Spotify data (top tracks, recent plays, playlists, audio‑features) in a modern web UI.
- **Why it matters** – Demonstrates secure OAuth integration, client‑side API consumption, and a polished React UI – all hot topics in front‑end/back‑end interviews.

---

## 2️⃣ Tech Stack

| Layer | Tech |
|-------|------|
| Front‑end | React (Create‑React‑App), Reach Router, Styled‑Components, JavaScript (ES6) |
| Back‑end | Node ≥10.13, Express, `dotenv`, `request`, `connect-history-api-fallback`, `cluster` |
| API | Spotify Web API (OAuth Authorization Code flow) |
| Deployment | Heroku (Node buildpack) |
| Dev tools | Yarn, concurrently, ESLint, Prettier |

---

## 3️⃣ Folder Structure

```
spotify_profile_analysis-main/
│   .env.example
│   README.md
│   package.json   ← server scripts
│
├─ client/               ← React app
│   ├─ package.json   ← front‑end scripts
│   └─ src/
│       ├─ components/   ← UI blocks (TopTracks, TrackItem, FeatureChart, …)
│       ├─ spotify/       ← API wrapper (fetches data with the token)
│       ├─ styles/        ← theme, mixins, media queries
│       └─ index.js       ← ReactDOM.render
│
└─ server/               ← Express server
    └─ index.js          ← OAuth routes, static serving, clustering
```

*(All component and style files are linked in the PDF via relative paths; you can click them in a markdown viewer.)*

---

## 4️⃣ OAuth Authorization‑Code Flow

> The user clicks **Login**, is redirected to Spotify, grants permission, and is sent back with an **access token** and a **refresh token**.

![OAuth Flow Diagram](oauth_flow_1786052858735.png)

### Step‑by‑step (plain English)
1. **Login click** → front‑end navigates to `/login` (server).
2. Server creates a random `state` string, stores it in a cookie, and redirects the browser to `https://accounts.spotify.com/authorize?...` with the required scopes.
3. User logs in on Spotify and clicks **Accept**.
4. Spotify redirects to `/callback?code=…&state=…` on our server.
5. Server verifies the `state` cookie (prevents CSRF).
6. Server exchanges the `code` for **access & refresh tokens** (secret + client‑id are only on the server).
7. Server redirects back to the front‑end (`FRONTEND_URI`) with `#access_token=…&refresh_token=…` in the URL hash.
8. Front‑end parses the hash, stores the tokens (e.g., in React context), and removes the hash from the address bar.
9. All subsequent Spotify API calls are made directly from the browser using the `Authorization: Bearer <access_token>` header. When the access token expires, the front‑end calls `/refresh_token` to obtain a fresh one.

---

## 5️⃣ Server Walk‑through (`server/index.js`)

| Section | What it does |
|---------|--------------|
| **Environment** (`dotenv`) | Loads `CLIENT_ID`, `CLIENT_SECRET`, `REDIRECT_URI`, `FRONTEND_URI`, `PORT` from `.env`. |
| **Clustering** | Uses `cluster` to fork a worker per CPU core, enabling concurrent request handling. |
| **Static serving** | `express.static('../client/build')` serves the compiled React app. `connect-history-api-fallback` rewrites unknown routes to `index.html` so client‑side routing works. |
| **/login** | Generates `state`, sets cookie, builds Spotify authorization URL with scopes, redirects the user. |
| **/callback** | Verifies `state`, exchanges `code` for tokens via `request.post`, redirects to the front‑end with tokens in the hash. |
| **/refresh_token** | Accepts a `refresh_token` query param, calls Spotify’s token endpoint, returns a fresh `access_token`. |
| **Catch‑all `*`** | Serves `index.html` for any other route (SPA fallback). |
| **Listening** | `app.listen(PORT)` – each worker listens on the same port, the OS load‑balances incoming connections. |

---

## 6️⃣ Front‑end Walk‑through (`client/src`)

1. **Entry point** – `src/index.js` renders `<App />` into `#root`.
2. **Routing** – Reach Router defines routes:
   - `/` – Home (shows a login button if not authenticated).
   - `/top-tracks` – Shows `TopTracks` component.
   - `/recommendations` – Shows `Recommendations`.
   - …etc.
3. **Authentication handling** – `LoginScreen.js` redirects to `/login`. After the redirect back, `App` parses the URL hash, stores `access_token` & `refresh_token` in a React context (`AuthContext`). Tokens are kept in memory (not persisted) for security.
4. **API wrapper** – `src/spotify/index.js` exports functions like `getTopTracks(token)`, `getRecommendations(token)`. Each function uses `fetch` with the `Authorization` header.
5. **UI components** – All UI is built with **styled‑components** and a shared `theme`:
   - `TopTracks.js` fetches top‑track data and maps each entry to `<TrackItem />`.
   - `TrackItem.js` displays album artwork, track name, artists, album name, and formatted duration.
   - `FeatureChart.js` visualises audio features (danceability, energy, etc.) using a lightweight chart library.
   - `Recommendations.js` pulls Spotify recommendations based on the user’s top artists/tracks.
   - `Playlist.js` displays the user’s playlists.
6. **Styling system** – `src/styles/theme.js` defines colors, fonts, spacing, transition values. `mixins.js` supplies reusable snippets (`overflowEllipsis`, `flexCenter`). Media queries (`media.js`) make the layout responsive.

---

## 7️⃣ Running Locally (Quick‑Start)

```bash
# 1. Install Node (if not already)
nvm use   # or install a recent LTS version

# 2. Install deps for server & client
yarn && yarn client:install   # installs server deps then client deps

# 3. Create .env at the project root (copy from .env.example)
#    Set these values:
#    CLIENT_ID, CLIENT_SECRET – from your Spotify developer app
#    REDIRECT_URI=http://localhost:8888/callback
#    FRONTEND_URI=http://localhost:3000
#    PORT=8888   (optional)

# 4. Start both server and client
yarn dev   # runs `concurrently` – server on 8888, client on 3000

# 5. Open http://localhost:3000, click **Login**, and explore the UI.
```

---

## 8️⃣ Deploying to Heroku

1. Create a Heroku app: `heroku create my-spotify-profile`.
2. Set config vars (same as `.env` values) via `heroku config:set …`.
3. The `heroku-postbuild` script in `package.json` runs `yarn && yarn client && yarn build`, which builds the React app.
4. The Express server serves the static build (`client/build`).
5. Push: `git push heroku main`.

---

## 9️⃣ Interview‑Ready Q&A

| Question | One‑sentence Answer |
|----------|----------------------|
| **What does this project do?** | Visualises a logged‑in Spotify user’s personal data (top tracks, playlists, audio features) in a responsive React UI. |
| **Why is the token exchange done on the server?** | To keep `CLIENT_SECRET` hidden; the client never sees it, preventing credential leakage. |
| **What is the purpose of the `state` parameter?** | Protects against CSRF attacks by ensuring the response originates from the same login request. |
| **How does the app handle token expiration?** | The front‑end calls the `/refresh_token` endpoint with the stored refresh token to obtain a new access token. |
| **Why use `connect-history-api-fallback`?** | Guarantees that deep links (e.g., `/top-tracks`) still serve `index.html` so the SPA router can handle the route client‑side. |
| **Explain the clustering code.** | `cluster.isMaster` forks a worker per CPU core; each worker runs the Express server, increasing throughput on multi‑core machines. |
| **What would you improve?** | Move token storage to HttpOnly secure cookies, add unit tests for server routes, replace the deprecated `request` library with `node-fetch` or `axios`, and implement dark‑mode theming. |

---

## 🔟 Study Plan (1‑2 Days)

| Time | Goal |
|------|------|
| 0‑2 h | Clone the repo, run `yarn dev`, verify login flow works. |
| 2‑4 h | Read `server/index.js` line‑by‑line; draw the OAuth flow on paper. |
| 4‑6 h | Explore front‑end: `src/App.js`, routing, `AuthContext`, and each UI component. |
| 6‑8 h | Review styling system (`theme`, `mixins`, `media`). |
| 8‑10 h | Deploy a test version to Heroku; confirm env vars and static serving. |
| 10‑12 h | Practice answering the Q&A table out loud; rehearse the high‑level architecture explanation. |

---

## 📚 Further Reading

- Spotify Web API Docs – <https://developer.spotify.com/documentation/web-api>
- OAuth 2.0 Security Best Practices – <https://oauth.net/2/> 
- Node.js Cluster Module – <https://nodejs.org/api/cluster.html>
- Styled‑Components Theming – <https://styled-components.com/docs/advanced#theming>

---

*Prepared for interview preparation – concise, visual, and ready to export to PDF.*
