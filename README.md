# Facegate

On-device face detection and recognition in a single HTML file. No server, no
account, no upload — face descriptors are encrypted with AES-256-GCM and stored
only in the browser you enrolled them in.

Open `index.html` over HTTPS (or `localhost`) and the browser will ask for camera
permission. `file://` works in some browsers but not all — most block
`getUserMedia` outside a secure context.

## Models

Recognition needs the face-api.js weights, about 7 MB. Facegate fetches them
once from a list of mirrors, then stores every file in IndexedDB, so from the
second run onwards it starts with no internet at all.

If the download fails — a blocked CDN, a captive portal, an offline machine —
you have two ways to fix it permanently:

**Pick the folder.** Download the weights, then use *Load models from a folder*
on the loading screen (or Settings → Models). The files are copied into the
browser's own storage and reused forever.

**Ship them alongside.** Put the same files in a folder named `models` next to
`index.html`. That path is tried first, before any network request.

Either way the files you need are:

```
tiny_face_detector_model-weights_manifest.json
tiny_face_detector_model-shard1
face_landmark_68_model-weights_manifest.json
face_landmark_68_model-shard1
face_recognition_model-weights_manifest.json
face_recognition_model-shard1
face_recognition_model-shard2
```

They live in the `weights/` folder of
[justadudewhohacks/face-api.js](https://github.com/justadudewhohacks/face-api.js).
*Best* accuracy mode additionally uses `ssd_mobilenetv1_model-*` (5.4 MB) and
*Fast* mode uses `face_landmark_68_tiny_model-*`; both are optional, and
Facegate falls back to a mode it can actually run rather than failing.

To run with no network whatsoever, also save `face-api.min.js` as
`vendor/face-api.min.js` next to `index.html` — that path is tried before any
CDN.

## Accuracy

A match is evidence, not proof. On a small roster in good light expect roughly
97–99% with the lookalike guard enabled. It degrades with backlight, motion
blur, masks, small/distant faces, and identical twins.

What the app does to earn that number:

- **Alignment** — full 68-point landmarks align each face before the descriptor
  is computed, instead of embedding a raw crop.
- **Quality gating** — samples that are blurry, too dark, too flat, too small or
  badly angled are refused during enrolment and excluded from learning.
- **Guided poses** — enrolment walks through straight, left, right, up, down,
  close and smiling, and stores each accepted pose mirrored as well.
- **Robust scoring** — a blend of the nearest sample, the mean of the nearest
  three and the profile centroid, so one bad sample cannot create a permanent
  false match.
- **Lookalike guard** — the winner must beat the runner-up by a configurable
  margin, or the face is reported as Unknown rather than guessed.
- **Temporal confirmation** — a name only appears after several consecutive
  agreeing matches on the same tracked face.
- **Optional blink check** — a printed photo or a static screen does not blink.
  This is a speed bump, not real anti-spoofing: a video replay still passes.

## Data

Everything is encrypted with a key stretched from your passphrase
(PBKDF2-SHA256, 310,000 rounds) and kept in IndexedDB. There is no recovery if
you lose the passphrase. Backups export in the same encrypted form.

Face data is sensitive personal information under the Philippine Data Privacy
Act and comparable laws elsewhere. Get consent before enrolling anyone.
