# Day 2, Fine-Grained — Steps, Isolated
### Expands the "Day 2" entry in stage-1-core-mechanics-detailed.md

---

## Before you touch anything: lock everything except `steps`

- [ ] Use the **same product photo** across all four runs — your bag photo is fine, stick with it for continuity with earlier tests.
- [ ] Keep your prompt text **identical** in both `TextEncodeQwenImageEditPlus` nodes — don't retype it between runs, even a stray character changes the encoding.
- [ ] Set `control_after_generate` to **fixed** *before* your first run. Run once, then copy the resulting seed number down somewhere outside ComfyUI (a notes app, anywhere) — this is the one number every run this session must share.
- [ ] Keep `cfg` at whatever your working baseline is (your bypassed-LoRA baseline of cfg=3.0 works fine here).
- [ ] LoRA node stays bypassed, as decided.

If any of these drift between runs, you're no longer isolating `steps` — you're comparing two things changing at once, which was exactly the trap we agreed to avoid.

---

## The four runs

Run these in order, changing **only** the `steps` field each time. Save each output with a filename that records the setting (e.g. `bag_steps04.png`) so you're not relying on memory later.

| Run | steps | What you're watching for |
|---|---|---|
| 1 | **4** | Does the background red look clean, or noisy/blotchy? Do the bag's edges look defined or mushy? Does it look "unfinished"? |
| 2 | **10** | Same checks — has the noisiness from Run 1 mostly cleared up? |
| 3 | **20** | Same checks — is this visibly better than Run 2, or starting to look similar? |
| 4 | **40** | Same checks — can you actually tell this apart from Run 3 at a glance? |

---

## Why this happens (the mechanism, tied to what you already know)

Each step is one iteration of the reverse-diffusion process — the model predicting a small correction and removing a slice of noise, guided by your conditioning (from Q2/Q3 of the last doc: this is happening to that blank latent, steered by your photo+text conditioning, not by revealing a hidden real image). The `scheduler` (`simple`, in your graph) decides how large each noise-removal slice is at each step.

More steps = smaller, more precise corrections = more chances for the model to refine fine detail. But this has diminishing returns: past a certain point, the model has effectively converged on its best answer for this seed/prompt/cfg combination, and additional steps just re-confirm that same answer rather than meaningfully changing it. That convergence point is exactly what you're hunting for in this exercise.

---

## What to specifically compare across your 4 images
- **Edge quality** on the bag's transparent/reflective plastic — this is your hardest test case, so it'll show step-count differences more clearly than an easy subject would
- **Background uniformity** — clean flat red, or grainy/uneven at low steps?
- **Fine texture** in the vegetables/boba tea — resolved detail vs. mushy blur
- **Composition** — this should *not* meaningfully change between runs. If Run 1 and Run 4 have different framing, product position, or structure (not just detail level), that's a signal something besides `steps` drifted — treat it as a debugging exercise rather than a step-count finding.

---

## Fill this in and send it back

| Steps | Filename | Edge quality | Background uniformity | Still looks "unfinished"? |
|---|---|---|---|---|
| 4 | | | | |
| 10 | | | | |
| 20 | | | | |
| 40 | | | | |

Plus one sentence: **"It stopped visibly improving to me somewhere around ___ steps."**

That number is genuinely useful data, not just an exercise — it's specific to your subject matter, your prompt, and your baseline settings, which is exactly the kind of thing a generic tutorial can't tell you and your own swipe file can.

---

## Quick troubleshooting
- **All 4 images look nearly identical** → check `control_after_generate` didn't silently revert to `randomize` — this is the most common miss.
- **Images differ wildly in composition, not just polish** → seed likely isn't actually fixed, or the prompt text changed between runs. Worth chasing down — this is real debugging practice, not a detour from it.
