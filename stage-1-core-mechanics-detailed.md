# Stage 1 — Core Mechanics, Detailed Breakdown
### Companion to comfyui-mastery-roadmap.md — this expands Stage 1 into daily sessions

Each session below is ~20–40 minutes of hands-on work, one product photo, one variable. Do them roughly in order — each one assumes the previous day's intuition. Report back what you observe after each session and I'll react to your actual results before we move to the next one.

---

### Day 1 — Map the architecture you already have
**Goal:** be able to explain every node in your current graph from memory, no screenshot needed.

- Open your working graph. For each of these, write one sentence in your own words: `Load Diffusion Model`, `Load CLIP`, `Load VAE`, `Load LoRA`, `TextEncodeQwenImageEditPlus` (×2), `KSampler`, `EmptySD3LatentImage`, `VAE Decode`, `Save Image`.
- Specifically answer: *why are there three separate loaders instead of one "Load Checkpoint" node?* (This is the split-loading pattern almost every current model uses.)
- **Report back:** your one-sentence-per-node list. I'll flag anything off before we move on.

---

### Day 2 — Steps, isolated
**Goal:** see, concretely, what "steps" controls — independent of everything else.

- Same product photo, same prompt, same seed (set `control_after_generate` to **fixed**).
- Run at steps = **4, 10, 20, 40**. Four images, nothing else changed.
- Look at: fine detail, edge cleanliness, whether the result actually finishes "resolving" or still looks unfinished/noisy at low steps.
- **Report back:** what changed between 4→40, and where you'd say it stopped visibly improving (that's your "enough steps" point for this kind of edit).

---

### Day 3 — CFG, isolated
**Goal:** feel the over-drive effect directly — this is the exact mechanism that broke your first image.

- Same photo, same prompt, same fixed seed, steps held at whatever you found "enough" on Day 2.
- Run at cfg = **1, 2, 4, 8, 15**.
- Watch for: color oversaturation, harsh contrast, artifacts, the image "trying too hard" to obey the prompt at the high end.
- **Report back:** at what cfg value does it start looking overcooked? Compare that to what actually happened in your very first (broken) result.

---

### Day 4 — Sampler & scheduler combinations
**Goal:** see that these matter, but far less than steps/cfg — so you stop over-indexing on them.

- Same photo/prompt/seed/steps/cfg from your Day 2–3 sweet spot.
- Try: `euler`+`simple` (your default), `euler`+`karras`, `dpmpp_2m`+`karras`.
- **Report back:** are the differences subtle or dramatic? (Expect subtle — this sets your expectations correctly for how much time to spend tuning this later.)

---

### Day 5 — Denoise, and why edit workflows use 1.0
**Goal:** understand what denoise actually gates, specifically for an *edit* model like Qwen.

- Same setup, try denoise = **1.0, 0.7, 0.4**.
- **Report back:** what happens at low denoise — does it barely change the image at all, or does it break in some way? This tells you whether your edit workflow needs denoise=1.0 because the model conditions on the reference image *through the text encoder* rather than through a partially-noised copy of it (unlike classic img2img).

---

### Day 6 — Seed control and A/B habits
**Goal:** build the comparison habit you'll use for the rest of your ComfyUI life.

- Set `control_after_generate` to **fixed**, note the seed number down somewhere.
- Run the exact same settings twice — confirm you get an identical image both times (determinism check).
- Now change *only* the prompt wording slightly (e.g. add "soft shadow" to your instruction) with that same fixed seed, and compare.
- **Report back:** did fixing the seed make it easier to tell what your prompt change actually did?

---

### Day 7 — Positive vs. negative conditioning
**Goal:** confirm what the negative encoder is actually doing on an edit model (it's not quite the same as classic SD negative prompts).

- Run your workflow with the negative encoder's text emptied out entirely, everything else at your Day 2–3 sweet spot.
- Then run again with your normal negative instruction ("不能改变主体的外观" or similar) restored.
- **Report back:** did removing the negative prompt cause the model to drift on the product's appearance? Even a subtle difference is useful data.

---

### Day 8 (optional but recommended) — See the same concepts on a plain model
**Goal:** confirm these are *diffusion* concepts, not *Qwen-specific* ones.

- If you have any SDXL or SD1.5 checkpoint (even a small one), do a bare txt2img with it — no edit, no reference image.
- Notice: same steps/cfg/seed/sampler dials exist, same behavior patterns.
- **Report back:** did anything feel unfamiliar, or did it confirm the same mental model transfers?

---

## Self-test before moving to Stage 2
Without running anything, predict:
1. What happens if you set cfg=15 and steps=4 together?
2. What happens if you leave the seed on `randomize` while comparing two prompts?
3. Why might a Qwen edit workflow need denoise=1.0 while a classic img2img workflow uses something lower?

If you can answer all three confidently, you're done with Stage 1 — genuinely, not just on paper.
