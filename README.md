# The Penthouse Entry System

A single-file web app that uses your webcam to identify members at the door by comparing live faces against a folder of member photos exported from JoinIT.

Everything runs in the browser. No server. No data leaves the device.

---

## Quick start (60 seconds)

1. **Export member photos from JoinIT** (see *Getting member photos* below).
2. **Open `index.html`** — easiest way: serve the folder locally so the camera works on `https`/`localhost`:

   ```bash
   cd "/Users/cmd/Documents/Claude/Projects/Facial Recognition"
   python3 -m http.server 8000
   ```

   Then open <http://localhost:8000> in Chrome, Edge, or Safari.

   *(You can also double-click `index.html`, but some browsers block the camera on `file://`.)*

3. **Click "Load photo folder"** → pick the folder of member photos. Wait for indexing (a few seconds per photo).
4. **Click "Start Camera"**. Point it at the door.
5. **Test**: walk into frame. You'll see your name in green if you're enrolled, or a red "Not a member" alert otherwise.

---

## Getting member photos from JoinIT

The prototype just needs a folder of photos. Each filename becomes the member's name.

**Recommended:** export profile photos from JoinIT to a folder, naming each file with the member's name:

```
members/
  ├── John_Smith.jpg
  ├── Jane_Doe.jpg
  ├── Marcus_Lee.png
  └── …
```

The app converts `John_Smith.jpg` → "John Smith". Underscores, hyphens, and dashes all become spaces. Capitalization is normalized.

### If JoinIT has an export feature

Use it. Bulk-export profile photos with member name as the filename.

### If JoinIT only has an API

You can run a one-time script to download photos. Save it as `sync-joinit.py` and run it once, then point the app at the output folder. (We can build a proper sync if you give me the JoinIT API docs / endpoint.)

### Photo quality tips

- **Front-facing, well-lit, single face per photo.** Profile shots and group photos hurt accuracy.
- **One photo per member is enough**, but two or three at different angles improves robustness — just name them `John_Smith.jpg`, `John_Smith_2.jpg`, etc. Each will be indexed under "John Smith".
- Min ~150×150 px face area. JoinIT profile photos are usually fine.

If the indexer says "X had no detectable face," those files were skipped. Replace them with a clearer photo.

---

## What the app does

| Action | Behavior |
| --- | --- |
| **Match** | Green banner with name, member photo shown top-right, log entry recorded. |
| **No match** | Red banner "Not a member — Staff Alerted", alert tone, log entry recorded. |
| **Same person re-detected** | Suppressed for the *Dedup window* (default 30s) so the log doesn't spam. |
| **Export CSV** | Downloads the full check-in log with timestamps and match distances. |

---

## Tuning

The right-side **Settings** panel has two knobs:

- **Match threshold** (default `0.50`) — Euclidean distance between face descriptors. Lower = stricter (fewer false matches, more false rejects). Raise to ~0.55–0.60 if real members are being missed; lower to ~0.45 if non-members are being matched.
- **Dedup window** — how long after a match/no-match before the same person is logged again.

The **Alert tone** checkbox toggles the audio beep on no-match.

---

## Troubleshooting

**"Could not access camera"** — Browser blocked camera permission. Click the camera icon in the URL bar, or run via `localhost`/`https`. `file://` URLs are blocked by Chrome.

**"Failed to load face models"** — The face-api.js model weights are loaded from a CDN. Needs internet on first launch. Once loaded, the page will work offline if cached.

**Indexer skips most photos** — Photos are too small, too dark, or have multiple faces. Crop to head-and-shoulders, retake against a plain background.

**Real members marked "Unknown"** — Raise the threshold slider toward 0.55–0.60. If specific members are missed often, add a second photo of them.

**Strangers matched as members** — Lower the threshold toward 0.45. If a specific lookalike pair is the issue, get a sharper photo of one of them.

**App is slow** — Detection runs every animation frame. On older tablets, drop the camera resolution by editing `width: { ideal: 1280 }` to `640` in `index.html`.

---

## What this prototype does *not* do (yet)

These are the obvious next steps once you've validated the core flow:

1. **Live JoinIT sync** — currently a manual export. Wire up the JoinIT API for automatic refresh.
2. **Door unlock / staff webhook** — banner + tone only. Plug in a webhook URL (Slack, door relay, SMS) on match/no-match events.
3. **Persistent log** — log is in-memory; export CSV before closing the page. Production version should write to a server or local file.
4. **Per-member descriptor cache** — re-indexes every photo on load. For 200+ members, cache descriptors in IndexedDB.
5. **Liveness / spoof detection** — a printed photo can fool the current model. Add a "blink to enter" or 3D depth check for higher security.

Tell me which of these to tackle next and I'll build it.
