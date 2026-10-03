# review.html

Double-click to open (works from `file://`, no dependencies). Open `review.html?test` and check the console for the self-test.

**Inputs** (drop or pick, several at once is fine)
- a video (mp4/webm), **or** keyframe images named `00-12.400.png` (MM-SS.fff), `t12.4.png` (seconds) or `frame_0300.png` (frame ÷ FPS field); or images + `keyframes.json` `[{"t":12.4,"src":"a.png"}]`
- optional `markers.json` `[{"start":0,"end":4.2,"label":"intro"}]`
- optional `notes.json` (a previous export) to keep reviewing

**Use**: click the frame to pin a comment, `M` for a whole-frame note. Notes autosave in localStorage.

**Export**: *Copy prompt* / *Download .md* (open notes only), *Notes + frames* (md + one PNG per pinned note, pin drawn on), *Export notes.json*.
