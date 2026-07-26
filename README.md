# SBI PO Mock Engine

A single-file, offline-capable SBI PO Prelims practice app that **generates unlimited mock papers on the student's own device**.

No server. No database. No API keys. Open `index.html` and it works — including on a phone with no connection after first load.

---

## Why this exists

Most mock-test apps ship a fixed bank of questions. When you have finished them, you are done, and any wrong answer key in that bank is wrong forever.

This project inverts that. It ships **the logic that makes questions**, not the questions themselves. Every paper is built fresh, and — critically — **the app verifies its own answers before showing you anything**.

---

## The verification guarantee

This is the part that matters, so it is worth being precise about what is and isn't guaranteed.

| Section | How the answer is established | Can it be wrong? |
|---|---|---|
| **Quantitative (35)** | Correct-by-construction. The generator computes the answer and *then* writes the question around it. | No |
| **Seating / floor puzzles (15)** | All 8! arrangements are brute-forced. A puzzle is discarded unless **exactly one** arrangement satisfies its clues. | No |
| **Syllogisms (4)** | Exhaustive model enumeration. A conclusion "follows" only if it holds in **every** model satisfying the premises. | No |
| **Inequalities (4)** | The operator chain is walked and resolved symbolically. | No |
| **Other reasoning (12)** | Computed from a simulated walk, family graph, or cipher. | No |
| **English (30)** | Hand-written by a human, drawn from a fixed pool. | **Yes — human-authored, not machine-verified** |

That last row is the honest limitation. Read the [Known limitations](#known-limitations) section before deploying this to real students.

### The verifiers catch real bugs

During development the brute-forcer rejected **five of nine** hand-designed puzzles as contradictory, and the model checker independently reproduced two wrong syllogism keys in the original hand-written bank. The model checker also caught a bug *in itself* — a stemmer turning `"bus"` into `"bu"`, silently splitting one category into two. That is the entire argument for this approach: assertions are cheap, and checks are not.

---

## Quick start

### Run locally
```bash
git clone https://github.com/<you>/sbi-po-mock-engine.git
cd sbi-po-mock-engine
open index.html          # macOS  (use `xdg-open` on Linux, `start` on Windows)
```

### Deploy to GitHub Pages
1. Push this repo to GitHub.
2. **Settings → Pages → Source → GitHub Actions.**
3. Push to `main`. The included workflow publishes the site automatically.

Your app is then live at `https://<you>.github.io/sbi-po-mock-engine/`.

> **Note:** GitHub only *hosts* the file. Generation happens entirely in the student's browser — which is why this scales to any number of users at zero cost.

### Run the test suite
```bash
npm test
```
This fails the build if any generated paper contains an unsolvable puzzle, a wrong syllogism key, a duplicate question, or a malformed option set. It runs automatically on every push via `.github/workflows/test.yml`.

---

## How it works

```
index.html                  Everything, inlined. This is the deployable artefact.
├── src/generator.js        Seeded PRNG, brute-force permutation solver,
│                           syllogism model checker, inequality resolver
├── src/questions.js        Turns verified structures into exam questions
├── src/english_pool.json   Hand-written English (6 RC sets, 3 cloze, 39 standalone)
└── src/build_html.py       Rebuilds index.html after editing the above
```

After editing anything in `src/`, rebuild the single-file app:
```bash
python3 src/build_html.py
```

### Seeds, not databases

A generated paper is stored as **a single integer**. Because generation is fully deterministic, "Paper #482913" rebuilds byte-identically every time it is opened. This means:

- Storage cost is a handful of bytes per paper, not megabytes.
- Two students can compare scores on the same paper by sharing a seed.
- Papers survive a browser refresh without ever being serialised.

### Marking

SBI Prelims convention: **+1** correct, **−0.25** wrong, **0** unattempted. Section split is 30 English / 35 Quantitative / 35 Reasoning over 60 minutes.

---

## What's in the box

- **3 hand-audited papers** — 300 questions, every key independently verified. These are the flagship set.
- **Unlimited generated papers** — created on demand, 70 fresh verified questions each.
- Section-locked navigation, question palette, countdown timer, anti-cheat hooks.
- Real scorecard with accuracy and cutoff comparison.
- Review mode: filter by wrong / correct / unattempted, plus a weak-topic breakdown.

---

## Known limitations

Please read these before putting this in front of paying students.

**1. English repeats.** The pool holds 90 hand-written items. Each paper draws 30, so English begins visibly repeating from roughly the 4th paper onward, and is ~93% reused across 40 papers. Quant and Reasoning stay fresh; English does not. *Expanding `src/english_pool.json` is the single highest-value contribution to this project.*

**2. Structural familiarity.** Beyond ~20 papers, questions stay numerically fresh but become structurally familiar — the same topic templates with new numbers. This is fine for building speed and accuracy. It is not a substitute for the genuine variety of a well-run paid series.

**3. Rank and percentile are indicative only.** A real All-India rank needs a live candidate pool. These figures are derived from your score so they never contradict it, but they are flavour, not signal. Treat the raw score and accuracy as the real feedback.

**4. English answer keys are human-written.** Everything else in this README is a machine guarantee. This row is not. RC inference and tone questions cannot be brute-forced, so if a student disputes an answer, the English section is where to look first.

**5. Measured duplicate rate.** Across 40 consecutive generated papers: **5.5%** of generated (non-English) questions repeat. Not zero — worth knowing.

---

## Contributing

The most useful contributions, in order:

1. **More English content** in `src/english_pool.json` — the main bottleneck.
2. **New reasoning generators** (input-output, data sufficiency, puzzles with categories). Any new generator **must** ship with a verifier, and the test suite must fail without one.
3. **More arithmetic templates** in the `ARITH` array in `src/questions.js`.

House rule: **no generator without a verifier.** If correctness cannot be checked mechanically, it belongs in the hand-written pool where it is honestly labelled as such.

---

## License

MIT — see [LICENSE](LICENSE).
