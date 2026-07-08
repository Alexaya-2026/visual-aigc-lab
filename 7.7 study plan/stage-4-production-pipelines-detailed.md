# Stage 4 — Production Pipelines, Detailed Breakdown
### Companion to comfyui-mastery-roadmap.md — this expands Stage 4 into daily sessions

Same format as Stages 1–3: ~20–40 minutes of hands-on work per session, one variable at a time, report back and I'll react before you move on. This stage is a bit different in character, though — you're no longer isolating *what a setting does*, you're isolating *whether your known-good workflow survives being run unattended*. Keep that distinction in mind: a batch failure here is either an automation-layer bug or an edit-quality bug, and telling those apart fast is the actual skill.

**Assumed starting point:** you've cleared Stage 2 (reliable single-image edits) and ideally have at least the masking/segmentation half of Stage 3 solid, since Stage 4's batches will lean on whichever edit template you trust most. If a batch run misbehaves and you're not sure whether it's new (automation) or old (the underlying edit was never actually reliable), the first move is always to run that one failing image through your workflow manually, one at a time, outside the batch — that isolates the layer immediately.

**Keep the swipe file running.** This is where it starts compounding: "which resize/crop values hit which marketplace's spec," "which seed strategy actually mattered," "how big was the batch before something broke" are exactly the entries you'll want six months from now when you're setting this up for a new product line.

**Same caveat as Stage 3:** the specific batch-loader and save-format node names below are accurate as of now, but this corner of the custom-node ecosystem changes. If Manager surfaces something with a different name, search by capability ("batch image loader," "save image webp," "image resize") rather than the exact string.

**Baseline for this stage:** whatever edit-type workflow you trust most from Stage 2/3 (start with something simple — a white cutout is a good first target). You're not tuning it further here. If it breaks in a batch, that's data about automation, not a cue to start re-tuning cfg mid-stage.

---

### Day 1 — Batch-load a folder, run one template unattended
**Goal:** the actual first automation test — does your trusted single-image workflow survive running against a folder with no babysitting?

- Install a folder-batch loader via Manager if you don't have one — search "batch image loader." A commonly used option (WAS Node Suite's `Load Image Batch`) has a `mode` set to something like `incremental_image`: it loads the next image in a directory each time the workflow runs, rather than all of them at once.
- Set the loader's mode to increment through a small test folder (5–10 photos, nothing precious). Wire it in place of your usual `Load Image` node, everything else unchanged from your trusted template.
- In ComfyUI's "Extra options" next to Queue Prompt, set batch count to the number of images in the folder, then queue once and let it run without touching anything.
- **Report back:** did it actually process every image, or did it stall, repeat one image, or silently skip something? First automation failures are almost always here, not in the edit itself.

---

### Day 2 — Filename discipline
**Goal:** make sure you can tell which output belongs to which input without opening every file.

- Your batch loader almost certainly outputs a filename alongside the image. Wire that into your `Save Image` node's filename prefix (directly, or via a simple string-concatenation node if it needs one) instead of relying on ComfyUI's default auto-incrementing counter.
- Re-run Day 1's folder with this wired in.
- **Report back:** can you match every output file to its source input by name alone, at a glance, without cross-referencing anything?

---

### Day 3 — Seed strategy across a batch
**Goal:** find out whether a shared seed across different products actually matters, instead of assuming.

- Run the same small folder twice: once with the seed fixed to one number for every image, once with `control_after_generate` set to increment so each image gets a different seed.
- Look closely at outputs from unrelated products — not just do they look fine individually, but is there any visible "sameness" (identical background texture pattern, identical shadow placement) that feels like it's coming from the shared seed rather than the prompt.
- **Report back:** did the fixed seed cause any cross-product artifact, or was it genuinely a non-issue? (Seed only seeds the noise for that image's own latent, so there's a real chance the answer is "no difference" — worth confirming rather than assuming either way.)

---

### Day 4 — Save a reusable template per edit type
**Goal:** turn your trusted workflow into something you can pull up by name instead of rebuilding.

- Save your current graph two ways: a regular workflow save, and a **Save (API format)** export. Open both files and look at the difference — the API format strips UI-only data (node positions, colors) and is the format you'll actually need if Stage 5 has you driving ComfyUI headlessly later.
- Do this once per edit type you actually use (white cutout, lifestyle background, etc.) — you're building a small named library, not just one file.
- **Report back:** confirm you can close ComfyUI, reopen it, and reload one saved template correctly, with no manual rewiring.

---

### Day 5 — Marketplace output, pass 1: resize and crop
**Goal:** hit a real target spec, not a guess.

- Pick one actual marketplace you sell on and look up its *current* image requirements — resolution range, aspect ratio, minimum/maximum dimensions. (This is the roadmap's own advice: check the platform's current guidelines when you get here, since they do change.)
- Add a resize/crop step to your Day 4 template (an `Image Resize` or `Scale Image to Total Pixels` node, plus a crop node if the aspect ratio doesn't match your source) that produces exactly that spec.
- **Report back:** the exact numbers you're now targeting, and whether cropping to the required aspect ratio ever cut into the product itself on any test image.

---

### Day 6 — Marketplace output, pass 2: format and file size
**Goal:** don't get a batch rejected on file size after the edit itself was fine.

- ComfyUI's default `Save Image` node only writes PNG. For a real batch you'll likely want JPEG or WebP at a controlled quality — install a format-aware save node if you don't have one (search "save image webp" or "image save" in Manager; several options expose a `quality` parameter and format choice in one node).
- Save the same image as PNG, JPEG (quality ~85), and WebP (quality ~85). Compare file sizes and look closely at a 100% crop for visible quality loss.
- **Report back:** which format actually met your target marketplace's file-size limit with the least visible quality cost.

---

### Day 7 — The QC pass
**Goal:** a review habit that scales, instead of opening every single output at full size.

- Run a real folder (10+ images) through your full Day 1–6 template.
- Design a fast review method — a contact-sheet/grid of thumbnails you can scan in one glance is the classic approach, but even a consistent "zoom to each product's edge for two seconds" habit counts, as long as it's something you'll actually do every batch rather than skip when you're in a hurry.
- **Report back:** using your review method, what did you actually catch — and separately, did a full manual review of the same folder catch anything your fast method missed? Worth knowing the miss rate before you trust the shortcut.

---

### Day 8 — Full pipeline dry run *(this is your graduation check)*
**Goal:** the roadmap's actual Stage 4 target — run it for real and trust it.

- 15–20 real product photos, genuinely new to this pipeline, no more tweaking mid-run.
- Full chain: batch load (Day 1) → correct filenames (Day 2) → your chosen seed strategy (Day 3) → saved template (Day 4) → marketplace resize/format (Days 5–6) → your QC pass (Day 7).
- **Report back:** did you trust the output enough to spot-check rather than review every image individually? That's the actual graduation signal from the main roadmap — not perfection, just earned trust.

---

## Self-test before moving to Stage 5
Without running anything, answer:
1. A batch run produces one bad image out of twenty. What's the first thing you check to tell whether it's an automation bug or an edit-quality bug?
2. Why does the API-format workflow export matter for where you're headed next (Stage 5), when the regular save already works fine for you today?
3. Your QC pass caught 90% of what a full manual review caught, in a fraction of the time. Is that a good trade for a 20-image batch? For a 200-image batch? What changes between those two cases?

If you can answer all three with actual reasoning, you're done with Stage 4.
