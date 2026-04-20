# AR Floor Plan — Complete Setup Guide

## What This Does

Two-part system:

1. **Resolume Arena** projects your PDF floor plan at life-size on the floor
2. **A QR code** lets visitors scan with their phone → opens a web page → they point the camera at a printed marker near the projection → your 3D model (`model.glb`) appears in AR, aligned to the plan

The AR uses **MindAR.js** — free, open-source, runs entirely in the browser (no app install needed).

---

## Files Included

```
ar-floorplan/
├── index.html          ← The AR web app (this is what people open)
├── targets.mind        ← Your MindAR image-tracking target
├── model.glb           ← Your 3D model
├── qr-generator.html   ← Helper to generate the QR code
└── README.md           ← This file
```

---

## PART 1: Life-Size Floor Projection with Resolume Arena

### Step 1 — Convert Your PDF to an Image
Resolume doesn't natively read PDFs. Convert your floor plan PDF to a high-res PNG/JPEG:
- Use any tool: Adobe Acrobat export, online converter, or macOS Preview → File → Export as PNG
- Export at **300 DPI** minimum for sharp projection

### Step 2 — Calculate Real-World Pixel Mapping
You need to know the real dimensions of your floor plan (e.g., 12m × 8m).

- Measure your **projection area** on the floor (the actual lit rectangle from your projector)
- Example: projector covers 6m × 4m on the floor → your image must be scaled so the plan fits proportionally

### Step 3 — Resolume Arena Setup

1. **Launch Resolume Arena**
2. Drag your floor plan PNG into a **clip slot** in a layer
3. Go to the **Output** menu → **Advanced Output**
4. Set up your projector as an output:
   - Map the output slice to your projector
   - Use **Input Selection** to crop/position the floor plan image within the output
5. **Scale calibration**:
   - Place a physical measuring tape on the floor under the projection
   - Adjust the clip's **scale** in Resolume until a known dimension (e.g., a 1-meter wall) matches 1 real meter on the floor
   - Use the **Transform** controls (Scale X, Scale Y, Position) in the clip properties
6. **Keystone / Warp correction**:
   - If the projector is at an angle, use Arena's **warp mesh** or **corner pin** in Advanced Output to correct the trapezoid distortion so lines are straight on the floor
7. Once calibrated, the floor plan is projected life-size. **Don't move the projector**.

### Pro Tips for Resolume
- Mount the projector **directly above** pointing straight down if possible — eliminates keystone distortion
- Use a **short-throw or ultra-short-throw** projector for floor projection
- Projection brightness: at least **3000+ lumens** for a lit room
- Set the Resolume composition resolution to match your projector's native resolution

---

## PART 2: AR Overlay via QR Code + MindAR

### How It Works
- MindAR uses **image tracking** — it recognises a printed reference image through the phone camera
- Your `targets.mind` file contains the compiled tracking data for that reference image
- When the camera sees the reference image, MindAR anchors the 3D model (`model.glb`) to it

### Step 1 — Prepare the Marker Image

You need the **original image** that was used to compile `targets.mind`. This is the image visitors' phones will recognise.

**Print it:**
- Print it at a known size (e.g., A4 or A3)
- **Laminate it** if possible (reduces glare, lasts longer)
- Place it **on the floor next to or on top of the projection**, in a fixed position that corresponds to where the 3D model should anchor

### Step 2 — Host the AR Web App (Free with GitHub Pages)

The AR page must be served over **HTTPS** (required for camera access). The easiest free method:

#### Option A: GitHub Pages (Recommended)

1. Create a GitHub account (free) at [github.com](https://github.com)
2. Create a **new repository** called `ar-floorplan` (set it to **Public**)
3. Upload these files to the repository:
   - `index.html`
   - `targets.mind`
   - `model.glb`
4. Go to **Settings → Pages**
5. Under "Source", select **main** branch, root folder → **Save**
6. Wait 1–2 minutes. Your site will be live at:
   ```
   https://YOUR-USERNAME.github.io/ar-floorplan/
   ```

#### Option B: Netlify Drop (Even Faster)

1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag and drop the folder containing `index.html`, `targets.mind`, and `model.glb`
3. Done — you'll get a live HTTPS URL instantly
4. (Optional) Sign up to claim a custom subdomain

#### Option C: Local Network (For Testing)

```bash
# Install a simple HTTPS server (Node.js required)
npx serve --ssl-cert cert.pem --ssl-key key.pem .

# Or use Python (HTTP only — won't work for camera on mobile, but fine on desktop Chrome with flags)
python3 -m http.server 8000
```

For local HTTPS testing, you can also use [mkcert](https://github.com/nicerloop/mkcert) to generate trusted local certificates.

### Step 3 — Generate the QR Code

1. Open `qr-generator.html` in your browser (just double-click it)
2. Paste your live URL (e.g., `https://your-username.github.io/ar-floorplan/`)
3. Click **Generate QR Code**
4. Screenshot or right-click → save the QR code image
5. Print it and place it where visitors can scan it (on a stand near the projection, on a wall, etc.)

### Step 4 — Test It

1. Scan the QR code with your phone camera (or any QR scanner)
2. The AR web page opens in your browser
3. Allow camera access when prompted
4. Point the phone camera at the **printed marker image** on the floor
5. The 3D model should appear, anchored to the marker

---

## Tuning the 3D Model Alignment

In `index.html`, find this line:

```html
<a-gltf-model
  id="ar-model"
  src="./model.glb"
  position="0 0 0"
  rotation="0 0 0"
  scale="0.15 0.15 0.15"
```

- **`scale`**: Increase or decrease to make the model bigger/smaller. Use the `＋` / `－` buttons in the AR app to find the right value, then hard-code it here.
- **`position`**: Offset the model relative to the marker center. `position="0.5 0 0"` shifts it 0.5 units to the right.
- **`rotation`**: Rotate in degrees. `rotation="-90 0 0"` lays the model flat if it's standing upright.

### Matching Scale to the Projection

The key to perfect alignment:

1. The **marker's physical position** on the floor determines where the 3D model anchors
2. Place the marker at the **origin point** of your floor plan (e.g., a known corner)
3. Adjust `scale` until the 3D model matches the life-size projection beneath it
4. Adjust `position` to offset if the model's origin doesn't match the marker center

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Camera won't start | Must be HTTPS. Check browser permissions. |
| Model doesn't appear | Ensure `model.glb`, `targets.mind`, and `index.html` are in the same folder |
| Tracking is jittery | Better lighting. Print marker bigger. Avoid glossy paper. |
| Model is too big/small | Adjust `scale` in `index.html` or use the on-screen buttons |
| Model is sideways/upside-down | Adjust `rotation` (try `-90 0 0` or `0 180 0`) |
| QR code won't scan | Make it at least 3cm × 3cm. Ensure good contrast. |
| "Not Secure" warning | You're on HTTP, not HTTPS. Use GitHub Pages or Netlify. |

---

## Technology Stack

- **MindAR.js** v1.2.5 — open-source image tracking in the browser
- **A-Frame** v1.5.0 — WebXR/3D framework
- **No app install required** — works in Safari (iOS) and Chrome (Android)
- **Free hosting** via GitHub Pages or Netlify

---

## Quick Reference

```
Print marker image  →  Place on floor near projection
Host files on HTTPS →  GitHub Pages / Netlify (free)
Print QR code       →  Points to your hosted URL
Visitor scans QR    →  AR page opens → camera sees marker → 3D model appears
```
