# Build: Video/Keyframe Review Tool (Rubric-style "click, comment, turn notes into an AI prompt")

> Self-contained build spec. Hand this file to an AI coding agent in any empty folder and say "build this".
> It doesn't depend on any particular project.

## Context
Rubric (getrubric.app) is a local web dashboard for AI agents ("0 npm dependencies", installed by pasting a prompt into the agent). One part of it is a review view: a human scrubs through a video, clicks a spot on a frame, and leaves a comment. The comments are then turned into a prompt an AI tool can act on, e.g. *"@00:12.4, at (62%, 40%): object X should be red."*

This spec rebuilds only that review feature, kept generic and minimal:
- It works with any **video file**, or with a **set of keyframe images** that have timestamps. Review data is just `time + position + text`, so it doesn't need a real video.
- It doesn't try to match Rubric's design. It has the same features with a plain UI.

## Deliverable
One file, **`review.html`**: vanilla HTML/CSS/JS, no build step, no dependencies, no server needed (double-click to open). Target size is about 400–600 lines.

Optional second file **`README.md`** (about 20 lines) covering how to open the tool, the input formats, and the export format.

## Inputs (all by drag-and-drop or file picker, using the File API, so the tool works from `file://`)
1. **Video**: any format the browser plays (mp4, webm). Load it with `URL.createObjectURL`.
2. **Or keyframes**: several images. The timestamp comes from the filename (`00-12.400.png`, `t12.4.png`, or `frame_0300.png` plus an FPS field), or from an optional `keyframes.json` of the form `[{ "t": 12.4, "src": "a.png" }]`. Show each image while the playhead is between its `t` and the next one. Playback is a simple timer.
3. **Optional markers JSON**, generic sections or beats shown on the timeline, of the form `[{ "start": 0, "end": 4.2, "label": "intro" }]`. Each note gets tagged with the marker it falls in.
4. **Optional existing `notes.json`**: re-import it to keep reviewing.

## Features
1. **Viewer**: a video element, or an image for keyframe mode, inside a fixed-aspect stage.
   - Keys: Space plays and pauses, `,` and `.` step one frame (default 1/30 s, editable FPS field), `J` and `L` jump ±1 s, `M` adds a note at the current time.
2. **Timeline bar**:
   - Shows the playhead, the marker segments (labeled), and a dot for each note.
   - Click or drag to seek. Clicking a note dot seeks to it and selects it.
3. **Thumbnail strip** (video mode): capture a frame with `<canvas>` every N seconds (default 2) and at each marker start. In keyframe mode the images themselves are the strip. Click a thumbnail to seek.
4. **Pin and comment**:
   - Clicking the stage pauses playback and drops a pin.
   - Store x/y as a **percentage of the actual video frame**. Account for `object-fit: contain` letterboxing by using `videoWidth`/`videoHeight` against the element's box, and ignore clicks in the letterbox bars.
   - A text box opens, and saving creates `{ id, t, x, y, marker, text, status: "open" }`.
   - A note with no pin (added with `M`) has `x`/`y` set to null, meaning it applies to the whole frame.
5. **Notes panel**: sorted by time. Each note shows a timestamp `MM:SS.s`, its marker, and its text, and has actions to seek, edit, delete, and mark resolved. Pins for notes within ±0.5 s of the playhead are drawn on the stage.
6. **Persistence**:
   - Autosave to `localStorage`, keyed by the media filename and duration.
   - **Export** downloads `notes.json`. **Import** loads it back.
7. **Prompt export** ("Copy prompt" and "Download .md"). Leave out resolved notes. Format:
   ```
   # Review notes for <filename> (duration 01:09.7)
   Apply each change below. Times are seconds from the start; positions are % of frame (x from left, y from top).

   1. @00:12.4 [intro] at (62%, 40%): object X should be red
   2. @00:58.0 [s4] whole frame: hand sticker overlaps the hourglass, move it left
   ```
   - Also offer an optional **frame snapshot per note**: a "Download notes + frames (.zip-free)" action that saves each pinned frame as a PNG (`note-01-00-12.4.png`, with the pin drawn on it) next to the .md, so vision models can see what was meant. Use individual downloads, not a zip library.

## Out of scope (add later only if needed)
- Drawn shapes, boxes, or arrows. A pin is enough.
- Writing directly to disk or syncing with an agent through a server. Downloading files is enough for now. If you want it later: a stdlib `python -m http.server` plus a tiny POST handler.
- Multiple users, auth, or a Rubric-style dashboard with agent tabs.
- Calling an LLM from inside the tool. The prompt gets pasted into whatever agent you use.

## Implementation notes
- One `state` object (`media`, `markers`, `notes`, `fps`, `selectedId`) and one `render()` function that redraws the panel, timeline, and pins. No framework.
- Time source: `video.currentTime` in video mode, otherwise a `performance.now()` clock.
- Frame capture: `drawImage(video)` after the `seeked` event. Generate thumbnails one at a time through a hidden second `<video>` so the main player doesn't jump around.
- Style: CSS variables, a dark theme, and a layout of stage + timeline on the left with the notes panel on the right. Stack them vertically below 900px width.
- Escape note text when rendering. It's user input, so use `textContent` and never `innerHTML`.

## Effort / token estimate
| Item | Estimate |
|---|---|
| Code size | 1 file, about 400–600 lines |
| First build (write, open, fix) | **about 40–80k tokens**, roughly 1–2 hours of agent time |
| Each polish round after real use | about 15–30k tokens |
| Total to "usable like Rubric's review tab" | **about 100–150k tokens** |
| Full Rubric dashboard (agent tabs, task boards, etc.) | several times more; not part of this spec |

The tricky parts are only the pin coordinate mapping (letterboxing) and frame-accurate seeking and capture. Everything else is standard DOM work.

## Verification (agent must do these before reporting done)
1. Open `review.html` from `file://`, drop in an mp4, and check that playback, the keyboard shortcuts, and timeline seeking all work.
2. Pause, click an object, and add a note. Resize the window: the pin should stay on the object. Seek away and back: the pin should hide and then reappear.
3. Drop in a markers JSON and check that segments show on the timeline and that new notes get the right marker label.
4. Reload the page: notes are restored from localStorage. Export `notes.json`, clear everything, re-import, and check the notes are identical.
5. "Copy prompt" matches the format above, and resolved notes are left out.
6. Keyframe mode: drop 5 images named with timestamps. They should show in the right order and at the right times, and notes should work the same way.
7. A small `selfTest()` that runs when the URL has `?test`: it asserts that the time formatter, the filename-to-timestamp parser, and the letterbox coordinate mapping are correct, and logs PASS/FAIL to the console.
