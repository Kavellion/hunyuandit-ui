# Hunyuan-DiT v1.2 UI

## Example output

![Porcelain cyborg, frame 000001](examples/porcelain-cyborg-000001.jpg)

*Porcelain cyborg, frame 000001 — the first picture this app ever rendered. Seed 24571774.*

![Porcelain cyborg, frame 000017](examples/porcelain-cyborg-000017.jpg)

*Porcelain cyborg, frame 000017 — the same prompt with guidance steering on. Seed 827075950.*

![Marble goddess, frame 000023](examples/marble-goddess-000023.jpg)

*Marble goddess, frame 000023. Seed 1480281369.*

![Neon double exposure, frame 000027](examples/neon-double-exposure-000027.jpg)

*Neon double exposure, frame 000027. Seed 989757818.*

![Marble god, frame 000034](examples/marble-god-000034.jpg)

*Marble god, frame 000034. Seed 827235556.*

All five generated with this app on an RTX 4070 12 GB, 1024x1024, 50 steps. Frame 000001 is from the first proof session on Nov 1, 2026; the other four are from the guided runs at guidance 5.0 that followed it (about 56 seconds per image — see the status note below for why the two sets differ).

## Before you start

- **Python: nothing to install.** Python 3.12 is included in this package, in the `python_embeded` folder, ComfyUI portable style. Any Python you already have for other apps is left completely alone and is not needed here.
- **This model is NOT gated — no license click, no token.** `Tencent-Hunyuan/HunyuanDiT-v1.2-Diffusers` downloads anonymously from Hugging Face. There is no token box in this app because there is nothing to paste.
- **The model license is NOT MIT or Apache.** Hunyuan-DiT is released under the **Tencent Hunyuan Community License Agreement** — royalty-free use under its conditions (including its Acceptable Use Policy); products or services with over 100 million monthly active users need a separate license from Tencent, and outputs must not be used to improve any other large language model. Tencent claims no rights in the outputs. Read the full section in **Models and permissions** below before using the model commercially. The app code in this package is MIT (see License).
- A Windows PC with an NVIDIA graphics card (12 GB VRAM is the design target), about **16 GB free disk space**, and an internet connection.
- The first Generate downloads the model weights: **about 14.4 GB** (the repo's float32 files; in float16 they are about 7.2 GB resident — see How it works).
- **Status: PC-proven.** All three proofs passed on the target PC (RTX 4070 12 GB, 16 GB RAM) on Nov 1, 2026 — a first single image, then a three-line prompt queue with batches of five, frames 000001–000016 in one session. The same morning brought the follow-up runs: the same three prompts again at guidance 5.0 (frames 000017–000031) and a ten-frame marble god batch at guidance 5.0, slider only (frames 000032–000041) — 41 frames in all. Speed at 1024x1024 / 50 steps: about **28 seconds per image unguided**, about **56 seconds per image at guidance 5.0** (guidance runs the model twice per step, and the times show it). One honest note: the first session's settings record showed guidance 0.0 even though the slider read 5.0 — a type-in box defect in the app let a stray 0 silently override the slider. The app shipped in this release carries the fix (see Code verification), and the guided runs above are the comparison it made possible.

**Hunyuan-DiT, without Jupyter and without Colab.**

Hunyuan-DiT is Tencent's bilingual diffusion transformer: one prompt string feeds **two** text encoders at once — a BERT text encoder with a 77-token window and an mT5-XL encoder with a 256-token window — so English and Chinese prompts both work, and prompts can run longer than a single 77-token encoder allows. The whole pipeline loads in float16, the model card's own documented load dtype, with CPU offload always on, which is what makes a 12 GB card enough. The whole thing is wrapped in the same plain app shape as the other UIs in this family — run it, a page opens in your browser, type a prompt, pictures appear.

## What it does

- Text-to-image generation on your own GPU — no cloud, no account of any kind
- Prompt + negative prompt, with **real classifier-free guidance** — the negative genuinely steers the render (the pipeline builds negative embeddings for both encoders)
- **Steps and guidance sliders with type-in boxes** — defaults are **Hunyuan's own: 50 steps, guidance 5.0**, taken from the pipeline's call signature, not the SD family's 40 / 4.5. A filled type-in box wins over its slider when it holds a real value; the slider owns zero (see Code verification for why that rule exists)
- **Sizes:** Hunyuan renders its standard shapes cleanly, and the pipeline snaps any size to the nearest standard shape (resolution binning, on by default and left on here). The presets are exactly the ten standard shapes — Square 1024×1024 (default), Big square 1280×1280, three landscape, Wide, three portrait, Tall — plus Custom width/height boxes. When a custom size gets snapped, the status line and the settings file say what size was actually produced
- Up to 10 pictures per prompt — rendered as separate single calls, one after another, never a batched call — each with its own seed, saved as 000001.png, 000002.png, and so on
- Seed box (-1 = fresh random seed per image; a fixed seed S gives S, S+1, S+2, ... across a multi-image run)
- Dynamic prompts: {a|b|c} groups, one random pick per group, re-rolled for every single image; settings.txt records the expanded prompt actually used
- A prompt queue (one prompt per line — when it has lines, Generate runs the queue instead of the single prompt box, all lines sharing the one batch folder, frames numbered continuously, one settings.txt block appended per finished line — stop mid-queue and every finished line's frames AND settings survive, so the resume point is exact)
- A session-only prompt history dropdown (last 10), a Random prompt button, a Stop button to cancel a run, Star batch to copy a batch into images_out/favorites/, and an Unload models button to free VRAM and RAM
- Saved setups — a whole recipe stored under a name in `setups.json` and loaded back in one click; see **Saved setups** below
- CPU offload is **always on** — there is no off switch (see Requirements)
- Every run adds its pictures to the batch folder and appends a block to `settings.txt` next to them (date, time, frame range, model, load dtype, offload, size, steps, guidance, prompt, negative prompt, expanded prompts, seeds, and the produced size when binning snapped it)

**No LoRA slots — honestly.** `HunyuanDiTPipeline` in Diffusers 0.42.0 has no LoRA loading API at all (verified in the pipeline source — it does not inherit the LoRA loader mixin), so there is no honest slot to offer. If a future Diffusers gives this pipeline one, the slots can come back.

No encoder mode switch (both encoders always load — this pipeline has no lean mode), no img2img, no inpainting, no ControlNet, no upscaling, no animation — v1 is text-to-image only, kept simple on purpose.

## Saved setups

One compact row: type a name in the **Setup name** box and click **Save setup**; later, pick the name in the **Saved setups** dropdown and click **Load setup** to restore the whole recipe into the controls in one click. **Delete setup** removes the picked one; saving over an existing name replaces it, and the status line says so.

A setup captures nine fields: prompt, negative prompt, steps, guidance, size (preset plus custom width/height), seed, and batch name. Setups persist in `setups.json` next to `app.py` (created on the first save, readable by hand) and survive restarts; a code-only app.py swap does not touch it. NOT part of a setup: the prompt queue and Images per prompt — those are per-run choices, not recipe.

## Requirements

- Windows, an NVIDIA GPU (12 GB VRAM is the design target), and a recent driver
- Python 3.12 is bundled in the `python_embeded` folder: no Python install, no PATH changes, no venv — your other Pythons are left completely alone, and this app lives entirely in this folder
- Disk space: about 16 GB free for the first download (14.4 GB of weights), plus room for pictures
- System RAM: the weights are about **7.2 GB resident** in float16 (transformer ~3.0 GB, mT5-XL ~3.3 GB, BERT ~0.7 GB, VAE ~0.17 GB) with CPU offload always on — components live in system RAM and stream to the GPU as they run. 16 GB of system RAM works (that is what the proofs ran on); more is more comfortable.

## Install

1. Unzip the package to a fresh folder, e.g. `C:\AI\hunyuandit` — the whole app lives in the one folder, and the layout matters (see How it works).
2. Run **install.bat** once. It sets up pip in the bundled Python and installs PyTorch (CUDA build) plus the pinned packages in `requirements.txt`. Your system Python is never touched.
3. Run **run.bat**. A page opens in your browser at 127.0.0.1. Leave the terminal window open while you work — closing it stops the app — and keep the app's browser tab open (minimized is fine) while a queue runs.

If anything ever looks broken, delete the folder, re-extract, and run install.bat again — your pictures live in `images_out/` inside the folder, so copy that out first if you want to keep them.

## First test

1. Leave everything at defaults: Square 1024×1024, 50 steps, guidance 5.0, seed -1, Images per prompt 1, batch name `hunyuan-pictures`.
2. Click **Generate**. The first click downloads about 14.4 GB and loads the model — this takes a while — and one picture lands in `images_out/hunyuan-pictures/000001.png`. (At load the pipeline prints its standard "disabled safety checker" warning — the model repo ships with the safety checker off; the warning is informational and generation proceeds.)
3. Then try a 3-image run, and a two-line queue — the family's three proofs.

## Where pictures go

Pictures land in **images_out/** next to run.bat — one subfolder per batch name (default `hunyuan-pictures`), reused across runs (frames keep numbering upward, never a _2/_3 folder), with a `settings.txt` in each batch folder recording every run. Star batch copies a batch into **images_out/favorites/** under a fresh name. `setups.json` lives next to app.py, inside the inner `hunyuandit-ui` subfolder.

## How it works

- The whole pipeline loads in **float16** — `HunyuanDiTPipeline.from_pretrained("Tencent-Hunyuan/HunyuanDiT-v1.2-Diffusers", torch_dtype=torch.float16)` — because that is the model card's own documented load dtype. The repo ships float32 files (~14.4 GB on disk); in float16 the weights are ~7.2 GB resident. There is no quantization and no post-load dtype cast.
- **CPU offload is always on** (`enable_model_cpu_offload()`, no off switch): components live in system RAM and stream to the GPU as they run, which is what makes a 12 GB card enough.
- Both text encoders always load and hear the same prompt string, each in its own window (BERT 77 tokens, mT5 256 — set inside the pipeline's `encode_prompt`). There is no prompt-template argument in Diffusers 0.42.0 (the original Tencent repo's templates were not ported), so the app passes your text as-is.
- **Sizes snap.** The pipeline's resolution binning (on by default) snaps any requested size to the nearest of its ten standard shapes; `check_inputs` only demands divisibility by 8, and the app rounds typed sizes to the nearest multiple of 16 (16–2048) so that rule always holds, and records the actually produced size in the status line and settings whenever binning snaps it.
- The package layout is **wrapped**: the zip's top folder `hunyuandit-ui/` holds `install.bat`, `run.bat`, `requirements.txt`, the bundle README, the LICENSE, the `python_embeded/` runtime, and the app itself in the inner `hunyuandit-ui/` subfolder. `run.bat` runs the inner app from the outer folder, which is why pictures land next to `run.bat`.
- The page binds to 127.0.0.1 only and opens in your browser.

## Code verification

Verified on the build machine in a pristine virtual environment created from this package's own `requirements.txt` (diffusers 0.42.0, transformers 5.20.2, gradio 5.50.0), against files fetched from the model repo itself — then **PC-proven on the target PC on Nov 1, 2026** (see the status note at the top).

- **Tokenizer bench — 13/13.** Both real tokenizers load from the repo's own files: the BERT tokenizer (WordPiece, vocab 47,020, encodes English and Chinese) and the mT5 tokenizer (SentencePiece; loads, encodes past the 77-token mark up to its 256 window). The sentencepiece → protobuf chain is exercised directly — the SentencePieceExtractor parses the repo's `spiece.model` and its piece count equals the tokenizer's vocab exactly (250,100) — and guard tests fail if `protobuf`, `tiktoken`, or `sentencepiece` is removed from `requirements.txt`, plus a negative control that blocks protobuf imports and watches the extractor die. (That chain is pinned on purpose: it is the exact one that broke a sibling build's first load when it went untested.)
- **Weight-key audit — 10/10.** Every component's weights were audited against the real model classes built from the repo's own configs: transformer (1,544 tensors, 1,499,952,032 parameters), VAE (248), BERT encoder (391) — 0 missing, 0 unexpected, 0 shape mismatches, and each file's byte size matches its parameter count × 4 (float32) exactly. The sharded mT5 encoder's index covers every model key — the one key not stored verbatim, `encoder.embed_tokens.weight`, is the model's registered tied weight onto the stored `shared.weight` — and its stored parameters × 4 equal the index's total size exactly.
- **Pipeline bench — 7/7.** The real pipeline's `check_inputs` accepts this app's exact call pattern and rejects a non-multiple-of-8 size and a prompt+embeds combination with the real error texts. A pipeline built from tiny real components (real classes, the repo's real tokenizers and scheduler config) ran real end-to-end calls in the app's exact shape — including an empty-string negative — and produced real images. The real binning function snaps an unsupported size into the standard shapes and leaves 1024×1024 alone.
- **Real UI bench — 18/18.** The shipped `app.py` builds its real Gradio page under the real pinned gradio: the size dropdown holds exactly the ten standard shapes + Custom with Square as default, steps/guidance default to 50 / 5.0, and there is no radio, no checkbox, and no password field anywhere (no encoder modes, no token box).
- **Stub suite — 39/39.** The app logic (loader shape, generate, queue, setups, helpers, UI wiring) against fakes that mirror the real pipeline's defaults (50 / 5.0 / binning on), its real check_inputs rules and texts, and the real binning contract. The suite was re-run in full against the shipped app.py after the guidance fix below (39/39) and against the cold-extracted app.py from the release zip.

**The guidance fix, plainly.** The Nov 1 proof session exposed a defect the benches had missed: a type-in box can arrive at the app as 0 even when it was never deliberately set, and the app treated any arrival as an override — so a stray 0 silently replaced the Guidance slider's 5.0 on every proof run, and the settings file (which records what the pipeline actually received) showed 0.0. The shipped app.py fixes the rule for both boxes: a type-in overrides its slider only when it holds a real non-zero value; an empty box or a 0 leaves the slider in charge, and the slider owns zero. A regression test reproducing the exact proof case (slider 5.0, type-in arriving as 0.0) shows 0.0 reaching the pipeline before the fix and 5.0 after it. The guided comparison runs in the gallery above were rendered with the fix in place — and with the slider alone.

## Models and permissions

- Model: `Tencent-Hunyuan/HunyuanDiT-v1.2-Diffusers` (Hunyuan-DiT v1.2, Diffusers format), by Tencent. Ungated — anyone can download it.
- The model is under the **Tencent Hunyuan Community License Agreement** (2024/5/14). In plain terms: a royalty-free license to use and distribute under its conditions, including its Acceptable Use Policy; if you distribute the model you must pass the agreement along, mark modified files, and include its notice text; products or services with over **100 million monthly active users** need a separate license from Tencent; **outputs must not be used to improve any other large language model** (other than Hunyuan derivatives); and **Tencent claims no rights in the outputs** — the images you generate are yours under that clause. It is not MIT or Apache; read the full agreement in the model repo before commercial use.
- This package never uploads anything; generation is fully local after the download.

## License

The app code in this package is MIT, by Alan Carmack (see the LICENSE file). The MIT license covers the app code only, not the model — the model terms are above.
