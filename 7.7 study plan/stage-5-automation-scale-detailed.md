# Stage 5 — Automation & Scale, Detailed Breakdown
### Companion to comfyui-mastery-roadmap.md — same discipline as Stage 1/2/3, applied to going headless

The main roadmap calls this stage "ongoing," and that's accurate — there's no graduation check that closes it out the way Stages 1–4 do. What follows is a foundational sequence to get you from "I run everything by hand in the UI" to "I have a real, unattended batch pipeline." After Day 9 you keep extending it as your catalog grows; that's expected, not a sign you're behind.

Same format as before: one session, one new capability, one report-back before moving on. A few sessions here run longer than your usual 20–40 minutes (Days 1, 5, and 8 especially) — that's normal for infrastructure work, not a sign something's wrong.

**One habit specific to this stage:** commit your `workflow_api.json` files to version control (even a local git repo with no remote is enough) before you start scripting against them. You'll thank yourself the first time a script starts producing wrong output and you need to know whether the workflow changed or your code did.

---

### Day 1 — Export and read the API-format file *(longer session)*
**Goal:** understand the thing you're about to script against, before writing any code.

- In ComfyUI, export your working Qwen edit graph as an API-format file. Depending on your version this is either **Workflow menu → Export (API)**, or, on older builds, enable **Dev Mode** in settings and use the **Save (API Format)** button that appears. This file is different from a normal saved workflow — it has no layout info, just node IDs and values, which is exactly what a script needs.
- Open it in a text editor. Find and write down: the node ID holding your prompt text, the node ID holding your seed, and the node ID holding your input image filename. These three are what you'll be patching programmatically for the rest of this stage.
- **Report back:** the three node IDs, and confirm you can tell which `class_type` each one is just by reading the JSON.

---

### Day 2 — One job, sent by script instead of by hand
**Goal:** confirm the API path works at all, with everything else held identical to a manual run.

- Write the minimal script: load `workflow_api.json`, POST it to `http://127.0.0.1:8188/prompt`, then poll `http://127.0.0.1:8188/history/{prompt_id}` every couple seconds until it shows output, then fetch the image via `/view`.
- Use your **Stage 2 Day 0 baseline** exactly — same product, same prompt, same fixed seed, LoRA off — so the only variable that changed is "script vs. UI."
- **Report back:** does the script's output match a manual UI run with the same seed pixel-for-pixel (or close enough to confirm it's the same path)? If the script fails, note the exact error — connection refused, 404, or a silent hang are three different problems with three different fixes.

---

### Day 3 — Parameterize it: new product/prompt without touching the JSON by hand
**Goal:** turn Day 2's one-off script into a reusable function.

- Refactor so product image path and prompt text are function arguments, not hardcoded values — your script edits the Day 1 node IDs before sending.
- You'll also need the upload step here: images referenced by the workflow have to exist on the ComfyUI server first, via `POST /upload/image`, before you queue the prompt that references them.
- Run it for 3 different products/prompts, arguments only, no manual JSON edits.
- **Report back:** all 3 correct and distinct? This function is your reusable boilerplate for everything after this point.

---

### Day 4 — Real-time status via WebSocket
**Goal:** know exactly when a job finishes instead of guessing with a polling delay.

- Add the WebSocket connection (`ws://127.0.0.1:8188/ws?clientId=...`), and listen for `executing` messages. The gotcha worth knowing before you hit it: completion is signaled by an `executing` message where the node field is `None` **for your specific `prompt_id`** — if you don't filter by ID, a busy queue with other jobs running will make your script think it's done when it isn't.
- Compare against Day 2's polling loop on the same job. Time both.
- **Report back:** was the WebSocket version meaningfully better for your batch sizes, or was polling already good enough? Either answer is fine — this tells you which to default to going forward.

---

### Day 5 — Batch loop over a folder *(longer session)*
**Goal:** the actual point of this stage — multiple images, unattended.

- Point your Day 3 function at a folder of 5–10 real product photos, loop over them, save outputs to a structured folder (e.g., named by filename or product ID, not `output_00001.png`).
- Let it run completely unattended — no babysitting, no watching the queue.
- **Report back:** how many succeeded without you touching anything, and exactly what broke on the ones that didn't.

---

### Day 6 — Wiring in your actual catalog
**Goal:** move from "every file in a folder" to "every row in my product list."

- Build or use a simple CSV/spreadsheet with columns like `product_id`, `image_filename`, `prompt_override` (so different rows can request different edit types).
- Script reads it row by row, uses `product_id` for output naming, so downstream (marketplace upload, whatever comes next) can match outputs back to real listings without you renaming anything by hand.
- **Report back:** does the output organization actually match what you'd want to hand off downstream right now, or does the naming scheme need another pass?

---

### Day 7 — Guardrails and failure handling
**Goal:** catch problems automatically instead of discovering them during review.

- Add three things: a **smoke test** (run one known-good image before the real batch starts; abort if it fails), a **parameter schema** (reject a row up front if the file's missing or malformed, rather than letting it fail deep in the pipeline), and **logging** (which row succeeded, which failed, and why).
- Then deliberately break something — delete an input file, blank out a prompt — and confirm your script catches it cleanly instead of either crashing without a trace or silently producing a bad image.
- **Report back:** what actually happened when you broke it on purpose. If it failed silently, that's the most important thing to fix before you trust this with a real batch.

---

### Day 8 — Reproducibility and version control *(longer session)*
**Goal:** know exactly what produced any given output, and be able to get back to it.

- Keep committing your `workflow_api.json` to git as you already started back before Day 2 — tag or commit again every time you change it, so this becomes the point where that habit turns into a full system rather than a first step.
- Attach metadata to every batch run: workflow file (+ commit hash), model files used, seed, steps/cfg. A sidecar JSON per output, or a single log per batch run, both work — pick whichever you'll actually keep updating.
- **Report back:** pick an output from earlier this week and try to answer, using only what you saved — what workflow version, what settings, what seed produced it? If you can't answer that confidently, the metadata step needs more detail, not more automation.

---

### Day 9 — Custom nodes: only if a real gap shows up
**Goal:** decide, from evidence rather than speculation, whether you actually need one.

This day is intentionally not a recipe. Look back at Days 5–8: is there one step you're still doing manually, outside ComfyUI, for every single product — a renaming convention, a catalog-specific crop ratio, something your marketplace needs that no existing node quite does?

- If yes: bring me the specific repetitive step next time, and we'll figure out together whether it belongs as a small custom node or just more glue code in your wrapper script — the second option is very often the right call and gets skipped too often in tutorials that assume you always want a "real" node.
- If no clear gap has shown up yet: that's a completely valid answer. Building a custom node speculatively, before a real repetitive pain point exists, is exactly the kind of premature automation the main roadmap warns about.

**Report back either way** — either the specific gap, or confirmation that nothing's asking for one yet.

---

## Self-test
Without checking anything, answer:
1. A workflow runs fine in the UI but the same job via script fails or hangs. What are the two most likely causes, and how would you check each?
2. Why does filtering WebSocket messages by `prompt_id` matter, rather than just waiting for any "finished" signal?
3. You find a great output from three weeks ago and want to reproduce it exactly. What four things do you need on hand to actually do that?
4. Day 7 used three separate guardrails — a smoke test, a parameter schema check, and logging — instead of one try/except wrapped around the whole batch. What does each of the three actually catch that a single generic error handler would miss?

If you can answer all four with real reasoning, your foundation for this stage is solid — everything past this point (catalog scale, more sophisticated custom nodes, multi-machine setups) builds on exactly these habits, just at larger scale.

## Rough "operationally automated" checkpoint
Not a hard graduation line, since this stage doesn't have one — but a good gut check: run 20+ real products through your pipeline overnight, unattended, and review results in the morning rather than watching it run. If that's comfortable, you've cleared the actual point of Stage 5, even though the stage itself never really "ends."
