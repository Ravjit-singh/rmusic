# 🎵 RMusic Secure Cloud Player
RMusic is a hyper-premium, god-tier web application built to stream custom curated playlists directly from cloud storage (Google Drive).
Originally starting as a simple client-side static page, **Version 2.0** represents a complete architectural overhaul. It transforms the app into a highly secure, lightning-fast full-stack application featuring a native iOS/Android-grade UI, an in-memory Node.js caching proxy, and a secure upload relay.
## 🏗️ Architecture & Security Upgrades
The primary goal of this rebuild was to eliminate massive client-side vulnerabilities while drastically improving loading speeds.
### ❌ The Old Architecture (Vulnerable)
 * **Exposed API Keys:** Google Drive API keys were hardcoded into the client-side JavaScript, leaving the quota vulnerable to scraping and abuse.
 * **Direct Uploads:** The frontend posted files directly to an open Google Apps Script endpoint with zero authentication.
 * **Network Latency:** Every page load required the client to wait for Google Drive's API to query and return song data.
### 🛡️ The New Architecture (Secure Fullstack)
 * **The Node.js Micro-Proxy:** A lightweight Express backend now acts as a secure middleman. The frontend *never* speaks to Google Drive. It only requests data from /api/songs.
 * **In-Memory Caching (Zero Latency):** On server boot, the Node proxy fetches all metadata from Google Drive *once* and stores it in RAM. When the client requests the playlist, it is served from the cache in **< 50 milliseconds**.
 * **Encrypted Upload Gateway:** Uploads are now routed through the Node proxy (/api/upload) and protected by a custom Bearer token. Unauthenticated requests are rejected before touching the storage.
 * **Apps Script Relay:** To bypass complex OAuth2 Service Accounts, the Node proxy securely authenticates the file and forwards it to Google Apps Script internally to handle the physical Drive storage.
## ✨ God-Tier UI & UX Features
The frontend was meticulously crafted using **Vanilla HTML/CSS/JS** to mimic the fluid feel of a premium, native mobile application.
 * **Fluid Canvas Visualizer:** A dynamic, glowing waveform generated via the Web Audio API (AnalyserNode) that maps audio frequencies to a live <canvas> element tucked behind the playing track data.
 * **YouTube-Style Double Tap:** Invisible touch zones on the left and right sides of the player allow users to double-tap to skip forward/backward by 10 seconds, complete with a native ripple animation.
 * **Pure CSS Morphing Buttons:** The Play/Pause button utilizes zero image assets. It uses complex CSS border manipulations to smoothly morph between a play triangle and pause bars.
 * **Glassmorphism Design System:** Extensive use of backdrop-filter: blur(), semi-transparent overlays, and dynamic viewports (100dvh) to ensure a frost-glass aesthetic that adapts perfectly to mobile notches and browser bars.
 * **Smart Seekbar:** A minimalist track progress bar that optically expands when hovered or touched, giving the user precise control over playback.
 * **Premium Typography:** Utilizes **Outfit** for clean, geometric UI headings and **JetBrains Mono** for precise, tabular timestamps.
 * **Auto-Syncing Uploads:** When an admin uploads a new track, the UI automatically calculates Google Drive's indexing delay, forces a cache refresh from the server, and seamlessly updates the DOM without requiring a page reload.
## 🛠️ Technology Stack
**Frontend:**
 * HTML5, CSS3, Vanilla JavaScript (ES6+)
 * Web Audio API
 * FontAwesome (Icons)
**Backend:**
 * Node.js
 * Express.js
 * googleapis (Drive v3 API)
 * multer (In-memory multipart/form-data handling)
 * dotenv (Environment variable management)
**Storage & Infrastructure:**
 * Google Drive (Audio File Storage)
 * Google Apps Script (Internal Upload Handler)
## 🚀 Installation & Local Setup
If you want to run this application locally, follow these steps:
### 1. Clone the repository
```bash
git clone [https://github.com/YOUR_USERNAME/rmusic-secure.git](https://github.com/YOUR_USERNAME/rmusic-secure.git)
cd rmusic-secure

```
### 2. Install Dependencies
```bash
npm install

```
### 3. Environment Variables
Create a .env file in the root directory. **Never commit this file to version control.**
```env
# Your Google Cloud Platform API Key (Restricted to Drive API)
DRIVE_API_KEY=your_google_api_key_here

# Custom password to protect the /api/upload and /api/refresh endpoints
SECRET_ADMIN_TOKEN=your_super_secret_password

```
### 4. Configure Google Drive Folders
In index.js, update the folders object with the IDs of your public Google Drive folders:
```javascript
const folders = {
    chill: "YOUR_FOLDER_ID_1",
    gym:   "YOUR_FOLDER_ID_2",
    love:  "YOUR_FOLDER_ID_3",
    sad:   "YOUR_FOLDER_ID_4"
};

```
### 5. Start the Server
```bash
npm start

```
The server will build the initial cache and host the frontend at http://localhost:3000.
## 📡 API Reference
The Node.js proxy exposes the following endpoints:
| Endpoint | Method | Auth Required | Description |
|---|---|---|---|
| /api/songs | GET | No | Returns the fully cached JSON object of all categorized songs. |
| /api/upload | POST | Yes (Bearer Token) | Accepts multipart/form-data. Verifies token, converts audio to Base64, and relays to storage. |
| /api/refresh | POST | Yes (Bearer Token) | Forces the Node server to re-query Google Drive and rebuild the local RAM cache. |
## 🌍 Deployment Strategy
This application utilizes a split deployment model for optimal performance and SEO:
 1. **The Core Engine (Render):** The Node.js proxy and the actual HTML frontend are hosted as a Web Service on Render. Environment variables are injected securely via the Render Dashboard.
 2. **The "Always-On" Trigger:** To bypass Render's free-tier sleep limitations, a secondary automated script periodically pings the /api/songs endpoint, ensuring zero cold-starts.
 3. **The SEO Wrapper (GitHub Pages):** A lightweight index.html file containing optimal Open Graph and SEO meta tags is hosted on GitHub Pages. This wrapper iframes the Render URL (with allow="autoplay" permissions), providing a clean, branded domain (e.g., ravjit-singh.github.io/rmusic) while hiding the backend routing.
*Built with passion by Ravjit Singh during a 3-day sprint.*
