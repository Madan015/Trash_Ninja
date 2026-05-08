# Trash Ninja

**Demo:** [YouTube](https://www.youtube.com/watch?v=D0wcw9iyTZk&ab_channel=EishanAshraf)

A mobile app that uses AI image recognition to help users correctly sort and dispose of their waste. Point your camera at any piece of trash and Trash Ninja will tell you exactly what to do with it — recycle, compost, donate, or bin it.

---

## What It Does

1. **Capture or upload** a photo of an item using the in-app camera or your device's gallery.
2. **AI classification** — the image is sent to a backend running a Hugging Face waste classification model.
3. **Get a recommendation** — the app displays what type of waste it is and the correct disposal action.

### Disposal Actions

| Detected Label | Recommended Action |
|---|---|
| battery | BATTERY DISPOSAL |
| biological | COMPOST |
| cardboard, paper, metal, plastic, glass (all types) | RECYCLE |
| clothes, shoes | CLOTHING DONATION |
| trash | GENERAL WASTE |
| Low confidence (< 50%) | Take a better picture |

---

## Tech Stack

### Frontend — React Native (Expo)

| Package | Version | Purpose |
|---|---|---|
| React Native | 0.74.5 | Mobile UI framework |
| Expo | ~51.0.28 | Build toolchain & device APIs |
| React Navigation (native-stack) | 6.x | Screen navigation |
| expo-camera | ~15.0.16 | Camera capture with full controls |
| expo-image-picker | ~15.0.7 | Gallery image selection |
| expo-font | ~12.0.10 | Custom font loading |
| axios | ^1.7.7 | HTTP requests to backend |
| react-native-reanimated | ^3.15.4 | Animations |
| @react-native-community/slider | ^4.5.3 | Zoom control |
| @expo/vector-icons | ^14.0.4 | Material icons |

### Backend — Python (Flask)

| Package | Version | Purpose |
|---|---|---|
| Flask | 3.0.3 | HTTP server |
| transformers | 4.45.1 | Hugging Face model pipeline |
| torch | 2.2.2 | PyTorch model inference |
| tensorflow | 2.16.2 | TensorFlow runtime |
| Pillow | 10.4.0 | Image preprocessing |
| huggingface-hub | 0.25.1 | Model download |

**Model:** [`watersplash/waste-classification`](https://huggingface.co/watersplash/waste-classification) on Hugging Face  
**Deployed API:** `https://recycleapi.adaptable.app/classify`

---

## Architecture & Implementation

```
App.js                 # Navigation stack + font loading
├── HomeScreen         # Landing page with camera / upload options
├── CameraScreen       # Live camera preview with controls
├── UploadScreen       # Gallery picker (auto-opens on mount)
└── ResultScreen       # Calls backend, displays result
```

### Navigation Flow

```
HomeScreen  →  CameraScreen  ┐
            →  UploadScreen  ┘ → ResultScreen → HomeScreen
```

Image URIs are passed between screens via React Navigation route params. No global state management (Redux/Context) is needed for this linear flow.

### CameraScreen

- Full camera preview with a control bar for: flash toggle, torch toggle, front/back camera flip, zoom slider (0–1 range).
- Photos are taken without auto-saving to the device library.
- The captured URI is passed directly to `ResultScreen`.

### ResultScreen

- Receives the image URI, renders a 300×300 preview, and immediately posts the image to the classify endpoint as `multipart/form-data` via axios (120 s timeout).
- Shows a loading spinner during the request.
- Maps the returned label to a human-readable action string and renders both.

### Backend (`backend/app.py`)

- Single POST endpoint: `/classify`
- Accepts a multipart image, runs it through the Hugging Face pipeline, and returns:

```json
{ "classification": "plastic", "action": "RECYCLE" }
```

- Any label with a confidence score below 0.5 returns `"Not identifiable"`.

---

## Running the App

### Prerequisites

- Node.js 18+ and npm
- Python 3.9+
- Expo Go app on a physical device **or** an Android/iOS emulator

### 1. Frontend

```bash
# Install dependencies
npm install

# Start the Expo dev server
npm start

# Then choose a platform
npm run android    # Android emulator or device
npm run ios        # iOS simulator or device (macOS only)
npm run web        # Browser
```

Scan the QR code in the terminal with **Expo Go** to run on a physical device.

### 2. Backend (local)

```bash
cd backend

# Install Python dependencies
pip install -r requirements.txt
# Windows shortcut:
dependency_script.bat

# Start the Flask server (runs on http://localhost:5000)
python app.py
```

> **Note:** The app is currently configured to call the hosted API at `https://recycleapi.adaptable.app/classify`. To point it at your local backend, update the URL in `components/ResultScreen.js`.

---

## Project Structure

```
Trash_Ninja/
├── App.js                          # Entry point, navigation stack, font loading
├── app.json                        # Expo config (permissions, icons, splash)
├── package.json
├── babel.config.js
├── react-native.config.js          # Font asset linking
│
├── assets/
│   ├── fonts/
│   │   └── ProtestStrike-Regular.ttf
│   ├── icon.png
│   ├── adaptive-icon.png
│   ├── splash.png
│   └── logo.jpg
│
├── screens/
│   └── HomeScreen.js
│
├── components/
│   ├── Button.js                   # Reusable icon button (Material Icons)
│   ├── CameraScreen.js
│   ├── UploadScreen.js
│   └── ResultScreen.js
│
└── backend/
    ├── app.py                      # Flask server + model inference
    ├── requirements.txt
    └── dependency_script.bat       # Windows dependency installer
```

---

## Permissions Required

| Permission | Platform | Used For |
|---|---|---|
| Camera | Android & iOS | Live camera preview and capture |
| Media Library (read) | Android & iOS | Selecting images from gallery |
| Media Library (write) | Android | Saving captured photos |

Permissions are requested at runtime via `expo-camera` and `expo-media-library`.

---

## Credits

- Waste classification model: [watersplash/waste-classification](https://huggingface.co/watersplash/waste-classification)
- Built at **Hack the Valley 2024**
