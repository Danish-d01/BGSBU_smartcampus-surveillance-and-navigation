# BGSBU Smart Campus Surveillance & Navigation System

> **Baba Ghulam Shah Badshah University, Rajouri, J&K**  
> Built with Python (Flask) · OpenCV · Leaflet.js · OSRM API

---

## Project Structure

```
bgsbu_system/
├── app.py                  ← Flask backend (main entry point)
├── requirements.txt
├── static/
│   ├── css/
│   │   └── style.css       ← All page styles (dark tech theme)
│   ├── js/
│   │   ├── main.js         ← Landing page status polling
│   │   ├── admin.js        ← Admin panel live feed + log
│   │   └── navigation.js   ← Leaflet map + OSRM routing
│   └── faces/              ← Auto-saved alert screenshots
└── templates/
    ├── index.html          ← User landing page
    ├── admin.html          ← Live surveillance panel
    ├── logs.html           ← Saved alert image viewer
    └── navigation.html     ← Campus map & route planner
```

---

## Quick Setup (Step-by-Step)

### 1 · Prerequisites

- Python 3.9+ installed
- A working webcam
- Internet connection (for map tiles + OSRM)

### 2 · Install Dependencies

```bash
# Navigate to project folder
cd bgsbu_system

# (Recommended) Create a virtual environment
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate.bat       # Windows

# Install packages
pip install -r requirements.txt
```

### 3 · Run the Application

```bash
python app.py
```

You should see:

```
╔══════════════════════════════════════════════╗
║  BGSBU Smart Campus Surveillance System      ║
║  http://127.0.0.1:5000                       ║
╚══════════════════════════════════════════════╝
```

### 4 · Open in Browser

| Page | URL |
|------|-----|
| User Home | http://127.0.0.1:5000 |
| Admin Panel (Live Camera) | http://127.0.0.1:5000/admin |
| Alert Logs | http://127.0.0.1:5000/logs |
| Navigation Map | http://127.0.0.1:5000/navigation |
| MJPEG Video Stream | http://127.0.0.1:5000/video |
| JSON Status API | http://127.0.0.1:5000/status |

---

## Module Explanations

### 1 · Smart Surveillance (Face Detection)
- **How it works**: OpenCV reads each webcam frame via `VideoCapture(0)`.
- The Haar Cascade classifier (`haarcascade_frontalface_default.xml`, bundled with OpenCV) scans the grayscale frame for frontal faces.
- Detected faces get green bounding boxes + the on-screen label "Face Detected".
- The `/status` JSON endpoint (`face_detected: true/false`) is polled by the browser every 1.5–2 seconds so the UI updates in real time.

### 2 · Motion Detection
- **Algorithm**: Frame differencing — `cv2.absdiff(prev_gray, current_gray)` followed by binary thresholding and dilation.
- **Threshold**: Only contours with area > **8000 px²** trigger an alert, filtering out camera noise and minor vibrations.
- **Background subtractor**: A MOG2 subtractor is also initialised for future enhancement.
- **Auto-save**: When motion is active, a JPEG screenshot is saved to `static/faces/` at most every **5 seconds** (controlled by `SAVE_INTERVAL`).
- **Sound alert**: The browser plays a Web Audio API beep (sawtooth wave) at most every **3 seconds** (`BEEP_INTERVAL`) — no external sound file required.

### 3 · Video Streaming
- Flask route `/video` returns a `multipart/x-mixed-replace` HTTP response — the standard technique for MJPEG streaming.
- The generator function `generate_frames()` reads frames in a loop, runs the detection pipeline, JPEG-encodes each result, and `yield`s it.
- The `<img src="/video">` tag in `admin.html` auto-plays this stream.
- Frame is **horizontally flipped** (`cv2.flip(frame, 1)`) so the webcam behaves like a mirror — natural for self-monitoring.

### 4 · Admin Panel
- **Live Feed page** (`/admin`): Displays the MJPEG stream, real-time alert pills for face/motion status, and a scrolling detection log.
- **Logs page** (`/logs`): Lists all JPEG files saved in `static/faces/` as a responsive image grid with hover overlays.

### 5 · User Panel (Landing Page)
- Styled hero page with the BGSBU badge, animated title, system status dots, and three quick-access cards (Navigation, Admin, Logs).
- Status dots update live via polling — green for face, red (blinking) for motion.

### 6 · Navigation Map
- **Map library**: Leaflet.js 1.9.4 (CDN — no install needed).
- **Default layer**: Esri World Imagery satellite tiles (free, no API key).
- **Street layer**: OpenStreetMap tiles (toggle via sidebar buttons).
- **Centre**: `[33.396290, 74.347233]` — BGSBU Rajouri campus.
- **14 campus POIs** with custom SVG pin markers (colour-coded by category) and popups showing name, description, and coordinates.

### 7 · Routing
- **API**: OSRM (Open Source Routing Machine) public demo server — `https://router.project-osrm.org` — completely free, no API key.
- **Modes**: Walking (`foot`), Driving (`car`), Cycling (`bike`).
- **Coordinate format**: OSRM requires `longitude,latitude` — the code passes them in this correct order.
- **Fallback**: If OSRM returns no route (can happen for remote/unmapped roads), a dashed yellow straight-line polyline is drawn between the two points, with an approximate straight-line distance.
- Route info panel shows distance and estimated duration.

### 8 · UI Design
- **Theme**: Dark military/tech aesthetic with CSS custom properties.
- **Fonts**: Rajdhani (headings/labels) + Exo 2 (body) — Google Fonts.
- **Animations**: CSS keyframe animations for grid background, glow pulse, blink effect.
- **Responsive**: CSS Grid collapses to single-column on narrow screens.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Camera not opening | Ensure no other app is using webcam. Try changing `cv2.VideoCapture(0)` to `1` or `2`. |
| `ModuleNotFoundError: cv2` | Run `pip install opencv-python` |
| Map shows grey tiles | Check internet connection (tiles load from Esri/OSM CDN) |
| OSRM returns no route | Normal for remote campuses — fallback straight line is shown |
| Beep not playing | Click anywhere on the page first (browser requires user interaction for AudioContext) |
| `static/faces/` missing | It is auto-created on startup; check write permissions if not |

---

## Technologies Used

| Technology | Purpose |
|------------|---------|
| Python 3 / Flask | Web server, video stream, REST API |
| OpenCV (cv2) | Webcam capture, face detection, motion detection |
| Haar Cascade | Face classification (bundled with OpenCV) |
| Leaflet.js | Interactive map rendering |
| OpenStreetMap | Street map tiles |
| Esri World Imagery | Satellite map tiles (free) |
| OSRM Public API | Turn-by-turn routing (free, no key) |
| Web Audio API | In-browser beep sound (no file needed) |
| HTML / CSS / JS | Frontend interface |

---

*All APIs and libraries used are free and open-source. The system runs entirely locally with no paid services.*
