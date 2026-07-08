# Stage 2 — Instruction-Based Editing, Detailed Breakdown
### Companion to comfyui-mastery-roadmap.md — same discipline as the Stage 1 detailed breakdown, applied to editing

This supersedes the earlier looser "Stage 2 deep dive" doc. That version had templates and failure tables but never pinned down the numbers underneath them — which is exactly the gap that would let a stray Lightning LoRA or a wrong cfg quietly wreck a "prompt-only" comparison, same as it nearly did in Stage 1.

Each session below is ~20–40 minutes, one product photo (or a specific pair), one variable changed. Report back what you observe after each day and I'll react to your actual results before we move to the next one — same loop as Stage 1.

---

### Day 0 — Pin your Stage 2 baseline
**Goal:** lock in one fixed configuration that every day below uses, so an edit-type difference is never secretly a sampler-settings difference.

- Write down the exact steps / cfg / sampler / scheduler you landed on during Stage 1 Days 2–4. **LoRA off.** This combination is your **Stage 2 baseline** — every day below uses it unless a day explicitly says otherwise (Day 8 does, deliberately — that's its entire point; Day 9 also may, see its note below).
- If you never fully locked those in: use steps=20, cfg=4, sampler=`euler`, scheduler=`simple` as a starting baseline instead — that's what ComfyUI's own native Qwen-Image template defaults to, so it's a reasonable default to start from. Treat it as a starting point, not a settled answer: if you want to compare it against other sampler/scheduler combos, that's a fair thing to test empirically (Stage 1 Day 4 already showed you these differences are usually subtle anyway, so don't over-invest here).
- **Prompting formula drill:** rewrite 5 vague prompts you'd naturally write ("make the background nicer," "clean this up") into the formula: `[action] + [what changes] + [what it becomes] + [what must stay unchanged]`. Run all 5 against one easy-set photo, baseline settings, fixed seed.
- **Report back:** your locked baseline numbers, and what changed between the vague and formula-based prompts.

---

### Day 1 — Background replacement: solid color
**Goal:** isolate what the *color instruction alone* does, with everything else nailed down.

- Baseline from Day 0, LoRA off, seed fixed.
- Prompt: `"Replace the background with a solid [color] background. Keep the product (shape, texture, color, reflections, shadow) completely unchanged. The background should be flat and uniform with no gradient or texture."`
- Run 3 times — white, black, one brand color — changing only the color word.
- Watch for: background uniformity (check corners, not just center), color fringing at the product edge.
- **Report back:** which color (if any) was least uniform, and whether fringing showed up consistently or only on one run.

---

### Day 2 — Background replacement: lifestyle scene + multi-image references
**Goal:** isolate what the *reference image* contributes, separate from the text instruction.

- Baseline from Day 0, LoRA off, seed fixed.
- Wire `image1` (product) and `image2` (reference scene) into your `TextEncodeQwenImageEditPlus` node.
- Prompt: `"Place the product from image1 into the setting shown in image2. Match the lighting direction and color temperature of image2. Keep the product itself unchanged."`
- Run 3 times, same product/prompt, only `image2` changes (try a bright, a neutral, and a warm-toned reference scene).
- Watch for: does the lighting-match instruction actually transfer, or does the product look pasted on regardless of the reference?
- **Report back:** which reference scene integrated best, and whether shadow direction matched the new scene without you having to ask separately.

---

### Day 3 — Clean cutout: easy case vs. hard case
**Goal:** feel where Qwen's cutout quality actually starts to break — on purpose, with a controlled comparison.

- Baseline from Day 0, LoRA off, seed fixed.
- Prompt: `"Remove the background completely, leaving only the product on a pure white background. Preserve fine edge detail exactly."`
- Run it once on an easy-set product, once on a fine-edge hard-set product (jewelry, fabric, anything with loose/thin edges). Same prompt, same baseline, only the input photo changes.
- Watch for: halo/fringe at the edge, whether fine detail (chain links, loose threads) survives or gets smoothed away.
- **Report back:** describe the actual difference in edge quality between the two — this is your first real data point on "prompt vs. segmentation," which matters for Stage 3.

---

### Day 4 — Lighting/shadow adjustment
**Goal:** isolate direction, and catch the most common failure — color drifting along with light.

- Baseline from Day 0, LoRA off, seed fixed.
- Prompt: `"Adjust the lighting to come from [direction]. Keep the product's true color and material properties unchanged — only the lighting and resulting shadows should shift."`
- Run 3 times — light from left, right, top — only the direction word changes.
- Watch for: does the product's actual color shift along with the lighting change? (Common failure — check it specifically, don't just glance at the overall vibe.)
- **Report back:** did any of the three shift color, and did shadow direction actually match what you asked for?

---

### Day 5 — Retouch: vague vs. specific instruction
**Goal:** confirm that instruction specificity — not the model's judgment — should be doing the targeting.

- Baseline from Day 0, LoRA off, seed fixed.
- Run twice on the same photo: once with `"Remove the blemish from the product"` (vague), once with `"Remove the scuff mark on the lower-left edge of the lid. Keep everything else unchanged"` (specific). Same seed both times.
- Watch for: did the vague version "fix" something you didn't ask about, or miss the actual blemish entirely?
- **Report back:** the concrete difference between the two outputs — this tells you how much slack you can safely leave in a retouch prompt before it starts guessing.

---

### Day 6 — Color/material variant generation
**Goal:** catch the failure mode where color changes but material finish silently changes with it.

- Baseline from Day 0, LoRA off, seed fixed.
- Prompt: `"Change the product's color to [color], keeping the exact same shape, material texture, lighting, shadow, and background."`
- Run 3 times, only the target color changes.
- Watch for: does a matte product stay matte across all three, or does one variant come out glossier/flatter than the original?
- **Report back:** which color (if any) caused a visible finish/sheen change alongside the color change.

---

### Day 7 — Model variant comparison: fp8 vs. GGUF (vs. bf16 if you have the VRAM)
**Goal:** see the actual quality cost of compression, instead of assuming it from a spec sheet.

- Same prompt, same steps/cfg/seed as your Day 0 baseline — only the loaded diffusion model file changes.
- Run it on one easy-set product and one hard-set product (busy background).
- Watch for: where does the quality gap actually show up — fine detail, background uniformity, color accuracy? It's often not evenly distributed.
- **Report back:** whether the gap was visible enough to matter for real listings, or academic.

---

### Day 8 — Lightning LoRA, deliberately, with matched settings
**Goal:** finally reintroduce the thing that broke your very first result — this time with the correct, matched configuration, so you see it work *and* see it break, both on purpose.

**Before you run anything, check which Lightning LoRA file you actually have** — this is the same "check the exact file, not just the general name" trap as the Stage 3 ControlNet situation. Lightning LoRA ships in more than one variant (4-step and 8-step versions both circulate), and the recommended scheduler isn't always `simple` — some workflows built around these LoRAs use `beta` instead. The steps/scheduler below are a reasonable starting point, but treat them as "confirm against your file's name/model card," not gospel.

Run three passes on the same product/prompt/seed:

1. **Your Day 0 baseline, LoRA off** (your quality reference point).
2. **LoRA on, steps=8 (or 4, if that's the variant you have), cfg=1, sampler=`euler`, scheduler=`simple` (try `beta` too if `simple` looks off)** — the settings the LoRA is actually built for.
3. **LoRA on, but using your Day 0 baseline cfg instead of cfg=1** — this deliberately recreates your original break, now with full understanding of why.

Time passes 1 and 2 if you can.

- **Report back:** the quality difference between 1 and 2, the speed difference, and confirm that pass 3 broke in the way you now expect. If it *didn't* break the way you expected, that's more interesting than if it did — tell me exactly what happened.

---

### Day 9 — Deliberate failure-mode tour
**Goal:** use your best settings from Days 0–8 against your three hardest photos, and document exactly where instruction-based editing stops being the right tool.

- Use whichever configuration actually won on Day 8 — your Day 0 baseline if pass 1 held up best, or the matched Lightning LoRA settings if pass 2 did. Lock that choice in before you start; no more tweaking mid-session.
- Run it against: your reflective/transparent product, your fine-text product, your busy-background product.
- Don't try to fix failures with more prompt engineering this session — just document what breaks and how.
- **Report back:** three short notes, one per photo. This list becomes your Stage 3 starting punch list.

---

## Self-test before moving to Stage 3
Without running anything, answer:
1. Your reflective-object photo from Day 9 — is more prompt tweaking likely to fix it, or is that now a Stage 3 problem? Why specifically?
2. Why did Day 5 test a vague *and* a specific version of the same prompt instead of just using the specific one?
3. If Day 8 pass 3 hadn't broken the way you expected, what would that tell you about your original break back in Stage 1 — was it really just cfg, or something else?

If you can answer all three with actual reasoning (not just a guess), you're done with Stage 2.

---

## Graduation check
Pick **two product photos you haven't used anywhere in Stage 2.** Without this doc or your swipe file open:
1. Produce a clean solid-color background swap.
2. Produce a white cutout.

First or second attempt only, both products, using your Day 0 baseline. If it holds, move to Stage 3. If not, trace the failure back to the specific day it belongs to rather than re-running the whole stage.
