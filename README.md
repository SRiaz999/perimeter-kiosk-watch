# Perimeter · Kiosk Watch

**Real-time shoulder-surfing detection for ATMs and agent-banking kiosks. It runs entirely in the browser, with no cloud and no stored footage.**

🔴 **Live demo:** https://peaceful-smakager-9fbbc4.netlify.app/
🌐 **Project website:** https://sriaz999.github.io/perimeter-kiosk-watch/

Built with Edge Impulse, WebAssembly, and plain JavaScript.

---

## The problem

Shoulder surfing is when someone stands close to a kiosk and watches a customer type their PIN. It needs no hacking tools and no skimming hardware, only a clear view. It's also invisible: privacy screens are passive, nothing flags when someone is peeking, and victims usually find out days later when unauthorized transactions appear.

The risk is highest at **agent-banking points**, where shopkeepers process transactions in open stalls with people standing close behind the customer. Pakistan alone has over 731,000 branchless-banking agents (State Bank of Pakistan data, reported by Business Recorder), and global card-fraud losses reached $33.4 billion in 2024 (Nilson Report).

## What it does

Perimeter watches the area around a kiosk through a webcam and classifies every frame as one of two classes:

| Class | Meaning | UI response |
|---|---|---|
| `positive` | One person in frame (safe) | Green border |
| `negative` | Multiple people in frame (possible shoulder surfing) | Red border, alert banner, timestamped log entry |

Everything happens on the device. No image or video frame is uploaded, stored, or sent anywhere, which protects customer privacy and works with unreliable internet.

## How it works

```
Webcam → resize frame to 96×96 → pack RGB pixels into features
      → Edge Impulse model (WebAssembly) → positive / negative + confidence
      → alert if "negative" confidence ≥ threshold
```

1. **Capture:** the browser reads webcam frames with `getUserMedia`.
2. **Pre-process:** each frame is drawn onto a hidden 96×96 canvas, and each pixel is packed into the single-number RGB format Edge Impulse expects: `(r << 16) + (g << 8) + b`.
3. **Classify:** the features go to the Edge Impulse model, compiled to WebAssembly and running in the browser tab.
4. **Alert:** if the alert label (default `negative`) scores above the chosen confidence threshold (50%, 60%, or 75%), the interface turns red and logs the event. Alerts are rate-limited to one log entry every 2 seconds.

The input size and class labels are read from the model at load time, so a retrained model can be dropped in without code changes.

## Tech stack

- **Edge Impulse:** data labeling, training, testing, and export
- **WebAssembly (browser, SIMD build):** on-device inference
- **Vanilla HTML/CSS/JavaScript:** the app, with no framework or build step
- **Netlify:** HTTPS hosting (browsers only allow camera access on secure origins)

## How I built it

1. **Defined the problem.** I started from "where can low-cost edge AI close a real gap in fintech?" and chose shoulder surfing at kiosks because it's common, cheap for attackers, and currently undetected.
2. **Framed it as a classification task.** Instead of detecting and counting faces, I trained a simple whole-frame classifier: *one person* vs *multiple people*. It's lightweight, fast, and fits the use case.
3. **Built the dataset in Edge Impulse.** I labeled images as `positive` (single person) and `negative` (two or more people), then split them into training and test sets.
4. **Trained and tested the model.** I checked results in Edge Impulse's Live Classification view, where example images scored about 0.99 for `positive` and 0.92 for `negative`.
5. **Exported for the browser.** I exported the model as **WebAssembly (browser, SIMD)**, which gives `edge-impulse-standalone.js` and `.wasm`.
6. **Built the web app.** Using Claude as an AI coding assistant, I translated my concept and functional requirements into a working web application. I guided its development by specifying the webcam capture, feature extraction, and inference functionality, as well as designing the intended security-console interface with live readouts, debugging panels, and event logging.
7. **Deployed and recorded a demo.** I hosted the folder on Netlify to get an HTTPS link for camera access, and recorded a screen demo of one person showing `positive` and a second person triggering `negative`.
8. **Iterated on the model.** I retrained and re-exported several times (deploy versions 3 to 6). Because the app reads its settings from the model, each new export only needed the `.wasm` file swapped in.

## Challenges and what I learned

- **Improving model accuracy through dataset expansion.** My initial model could identify groups of more than two people as `negative` but struggled to recognize exactly two people. To fix this, I expanded the dataset with more examples of exactly two people, then retrained the model and rebuilt the web app with the improved model. This taught me the importance of dataset diversity, identifying model limitations, and iteratively improving machine-learning systems based on their performance.
- **Wrong export format.** My first download was the C++ library (for microcontrollers), which can't run in a browser. The right target is the WebAssembly export.
- **Browser requirements.** `.wasm` files must be served over HTTP with the right MIME type, and the camera needs HTTPS once the app is off `localhost`.
- **Lesson:** build in visible debugging (raw output, an event log) from the start. It cut the time to find each problem.

## Limitations

- It's a whole-frame classifier. It can't tell how close a second person is, whether they're looking at the screen, or whether they're a legitimate bystander far away.
- Accuracy depends on the training data matching the real camera angle, distance, and lighting at the kiosk. I tested with a laptop webcam, not a kiosk-mounted camera.
- This is a working prototype, not a production security product.

## Future work

- Train on more data captured at real kiosk camera angles and heights.
- Add a distance or proximity signal so far-away bystanders don't trigger alerts.
- Trigger a response beyond a visual alert, such as a privacy shutter or a prompt to cover the PIN pad.
- Port it to a Raspberry Pi or other low-cost hardware for deployment at agent points.

## Credits and sources

- Model training and export: [Edge Impulse](https://edgeimpulse.com)
- Global card-fraud figures: The Nilson Report
- Pakistan branchless-banking agent count: State Bank of Pakistan data, reported by Business Recorder
