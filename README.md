# The Dad Museum 🖼️

A single-page Father's Day experience: earn admission with a "Guess the Dad"
quiz, receive a vintage ticket, walk through opening museum doors, read the
welcome plaque, then explore one continuous wall of framed family photos —
ending in a Happy Father's Day message.

Built with plain HTML/CSS/JavaScript. No build step, no frameworks.

---

## View it

Just open **`index.html`** in a browser (double-click works).

If photos don't load when double-clicking (some browsers are strict about
`file://`), run a tiny local server from this folder instead:

```bash
cd dad-museum
python3 -m http.server 8000
# then open http://localhost:8000
```

To share with family, host the whole folder (Netlify drop, GitHub Pages, etc.)
or zip it and send it.

> The page uses Google Fonts for the elegant lettering. With no internet it
> falls back to system serif fonts — still looks good, just a touch plainer.

---

## Add / organize the photos

Photos live under **`images/`**:

```
images/
  quiz/
    john/      ← one folder per dad; the FOLDER NAME is the correct answer
      q1-crop.png   ← the cropped "clue" shown in the frame
      q1-full.jpg   ← the full photo revealed after answering
      q6-crop.png
      q6-full.jpg
    Santosh/   (q2, q8)
    Tushar/   (q3, q5)
    Vinayak/   (q4, q7)
  gallery/     ← every other photo, shown on the wall (g001.jpg, g002.jpg, …)
```

**How a question works:** each question is a **crop + full pair** that share a
question number. `q1-crop.png` is the close-up clue; after the user guesses,
it smoothly zooms out to `q1-full.jpg`. The dad whose folder the pair lives in
is the correct answer. The four answer buttons are simply the four folder names.

Currently set up from your folders: **8 questions** (q1–q8) across
**john, Santosh, Tushar, Vinayak**, plus **99** gallery photos.

### To change the quiz
1. In `images/quiz/<DadName>/`, add a matching pair: `qN-crop.png` (clue) and
   `qN-full.jpg` (reveal). Use the same `N` for the two halves of one question.
2. Put any other photos into `images/gallery/`.
3. Regenerate the photo lists:

   ```bash
   ./generate-data.sh
   ```

That rewrites **`data.js`** (read by `index.html`). Refresh the page — done.
The script auto-sorts questions by number and warns if a crop or full is missing.

> Notes
> - Folder/answer names are auto-prettified for display (`john` → "John").
> - The quiz shuffles question order on every attempt (and on retry).
> - `PASS_SCORE` / question count are in the `CONFIG` block in `index.html`.
> - Before answering, ONLY the crop is shown — the full photo is never visible
>   early. After answering, the full photo zooms out from the crop's region.

### Tuning the reveal zoom (optional)
The reveal makes the full photo start zoomed in, then pull back to the whole
image. By default it zooms out from the center. To make a question's zoom-out
start from the exact spot its crop came from, add an entry to `CROP_META` in
`index.html` (keyed by question id `Q<number>`):

```js
const CROP_META = {
  "Q1": { cropX: 62, cropY: 48, cropScale: 2.8 },  // % from left, % from top, zoom
};
```

All fields are optional and this never affects correctness — it's purely the
feel of the animation.

### Gallery framing & controls
Each wall photo keeps its **real aspect ratio** (no cropping — portraits stay
portrait, landscapes stay landscape) inside a gold CSS frame that hugs it. The
frame size comes from each photo's dimensions in `data.js` (read with `mdls`, so
EXIF-rotated photos are measured the way the browser displays them). Clicking a
frame opens the **lightbox** (whole photo, larger).

Controls (always visible): **← Previous · Next → · ▶ Autoplay/⏸ Pause · Final
Message ❤️**. Navigation is index-based, so Prev/Next move exactly one photo and
nothing is skipped. **Autoplay** advances one photo every few seconds
(`CONFIG.AUTOPLAY_MS`); it pauses automatically when you open a photo or reach
the final message, and manual Prev/Next keeps it running until you press Pause.

### Quiz reveal sizing & pacing
The quiz frame resizes to each photo's real aspect ratio for the reveal, so the
full photo is shown **uncropped** (portrait stays portrait, landscape stays
landscape — no faces cut off). The cropped clue still fills the frame as the
question; only the reveal uses the true aspect. After answering, the quiz does
**not** auto-advance — a **Next Question** button appears (it says **See
Results** on the last question) so you can linger on the photo as long as you like.

On wide screens the quiz lays out **two columns** (photo on the left, feedback +
answer buttons + Next on the right) so everything fits without scrolling; on
phones it stacks and the image is capped so the Next button stays in view.

### Gallery music
A background song (`audio/photograph.mp3`) plays during the
gallery walk-through. It does **not** play on page load — it starts when you
click **Begin Tour** (a user gesture, so browsers allow it), loops, and plays at
a moderate volume (0.3). Toggle it with the **♪ Music On / ♪ Music Off** button
in the gallery controls. To swap the song, replace that MP3 (keep the filename,
or update the `<audio id="bgMusic">` `src` in `index.html`). Music is independent
of **Autoplay** — autoplay only advances photos (every ~2s).

### Quiz order
Each attempt includes all 8 questions (2 per dad), shuffled fresh, arranged so
the same dad never appears twice in a row. The four answer buttons are also
re-randomized every question.

### Photo size
Originals were resized to ~1600px (fulls/gallery) and ~1400px (crops) for fast
loading. New full-res photos should be shrunk too, e.g.:

```bash
sips -Z 1600 -s formatOptions 72 photo.jpg --out images/gallery/g100.jpg
```

---

## How the quiz reveal works
Each question shows **one** photo cropped/zoomed inside a gold frame. After you
guess, the same photo smoothly **zooms out to the full picture** before the next
question — so the crop and the reveal are always the same image (no separate
crop files to manage).

---

## Tweak the experience
Open `index.html` and edit the **`CONFIG`** block near the bottom:

```js
const CONFIG = {
  QUESTIONS_PER_QUIZ: 8,   // questions per attempt
  PASS_SCORE: 5,           // correct answers needed to enter
  CROP_SCALE: 2.6,         // how tight the quiz crop is
  REVEAL_MS: 1200,         // zoom-out duration
  ADVANCE_DELAY_MS: 1800,  // pause on the full photo before next question
  GALLERY_STEP: 0.72,      // how far Prev/Next moves
};
```

All on-screen wording (entrance, plaque, ticket, final message) is plain text
in the HTML — search for it and edit freely.

---

## Controls
- **Gallery:** Previous / Next buttons, arrow keys, drag with the mouse, or
  swipe on touch. **Final Message ❤️** jumps to the ending any time.
- **Any photo:** click to enlarge. Close with ✕, tapping outside, or `Esc`.
- **🔊 / 🔇** (top-right in the gallery) toggles soft ambient room tone.
