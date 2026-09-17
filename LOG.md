# Project Log

The newest entries go at the top. Each entry covers what was done, what changed, and why.

---

## 2026-09-16: Models for a 32 GB machine; results layout

- **Process:**
  - Checked the machine: Apple M1 Pro with 32 GB RAM. Other apps were using about 10 GB, and the disk had 94 GB free (90% full).
  - Downloaded the models one at a time with `ollama pull`.
- **Decisions (models):**
  - **Size limit of about 9 GB per model.** macOS lets the GPU use about 21 GB, so this leaves room for the context cache and other apps. I skipped 27B–32B models; they would squeeze other apps, and the disk is already tight.
  - **Text models:** `llama3.2:1b`, `llama3.2:3b`, `gemma3:4b`, `qwen2.5:7b`, `llama3.1:8b`, `qwen3:8b`, `gemma2:9b`, `gemma3:12b` and `phi4:14b`.
  - **Vision models for the baseline:** `llava:7b`, `gemma3:4b` and `gemma3:12b`. `VISION_MODEL` became a `VISION_MODELS` list, and baseline results are saved to `results/vision_baseline.csv`.
  - **Download size.** The new models total about 50 GB.
  - **One model in memory at a time.** Each model is loaded (with a warm-up call), run on all images and tasks, then unloaded with `keep_alive: 0`. The benchmark loop now goes model, then image, then task.
  - **`qwen3`:** reasoning mode is turned off (`think: false`) so its timings are comparable and its output isn't padded with `<think>` text.
- **Decisions (layout):**
  - **Results grid.** The results section is now an HTML grid per image: the input (picture plus ASCII) on the left, then one row per model with one column per task. Stats are a small grey line under each response.
  - **Nothing cut off.** Responses are never truncated: ASCII sits in `<pre>` blocks with no height limit, and very long lines scroll inside their own cell. The same grid is saved to `results/report.html`.
  - **Full outputs saved.** Every response goes into the `output` column of `benchmarks.csv`.
  - **Stats moved to the end.** The summary table and charts are now under "Stats", and the matplotlib side-by-side cell was removed. The vision baseline shows its full answers.
  - **How it was checked.** I ran the notebook with the two llama3.2 models in the scratchpad and screenshotted `report.html` in headless Chrome.

## 2026-09-16: POC notebook built and run

- **Process:**
  - Created `.venv` (Python 3.14.2) and installed jupyter, nbconvert, pillow, numpy, requests, pandas and matplotlib.
  - Ollama 0.32.6 was already running with only `nomic-embed-text` installed, so I downloaded `llama3.2:3b` (2.0 GB). That's the only model this run tested.
  - Built `poc.ipynb` from `PLAN.md` and ran it headless with `jupyter nbconvert --execute --inplace`.
- **Changes:**
  - Added `poc.ipynb` (committed with its outputs), `images/` with 4 generated shapes (circle, triangle, smiley, gradient) and `results/` (`benchmarks.csv` plus rendered PNGs).
  - Updated the README with setup for the virtual environment and the headless run command.
- **Decisions and changes from PLAN.md:**
  - **Installed models only.** `MODELS` is filtered to installed models, so the notebook runs even if some aren't downloaded.
  - **Built-in test images.** Sample images are generated in the notebook when `images/` is empty, so nothing needs downloading.
  - **Ollama options.** Added `num_ctx: 8192` and `num_predict: 4096` to cap runaway output.
  - **Failed runs are recorded.** `ask()` handles failed generations: Ollama returned `done: False`, a string of `!!!!` and no timing fields for `gradient` + `invert`. The same happened with and without `num_ctx`, so the cause is the model or Ollama, not the notebook. Such runs are now recorded with `done=False` instead of stopping the notebook.
  - **Rendering.** `ascii_to_image` draws each character in a fixed 7x12 cell, because Pillow's default font isn't monospace.
  - **Vision baseline.** The cell is present but skipped, because no `llava` model is installed.
- **First results (llama3.2:3b, 80 columns, 16 runs):**
  - **Speed:** about 53 tokens/sec. `describe` takes about 1 s with about 400–600 input tokens.
  - **Flip:** kept the grid shape on circle (accuracy 0.98) and smiley (0.94). Those shapes are roughly symmetric, so copying the input would score about the same. On gradient and triangle it ran away to the 4096-token cap (about 84 s each).
  - **Invert:** never kept the grid shape; accuracy was 0.00–0.10.
  - **Descriptions:** all wrong ("spaceship", "Milky Way", "staircase").
  - **Takeaway so far:** a 3B model can copy the grid layout but doesn't understand the image or apply transforms.
- **Next:** download the other models (`qwen2.5:7b`, `gemma2:9b`) and `llava` for the baseline; consider asymmetric test images so flip scores mean something.

## 2026-09-16: Project log started

- **Process:** Added this `LOG.md` to record processes, changes and decisions from here on.

## 2026-09-16: README and POC plan

- **Changes:**
  - Replaced the placeholder `README.md` with a project overview, the metrics, example tasks, setup and layout.
  - Added `PLAN.md`, a section-by-section plan for `poc.ipynb`.
- **Decisions:**
  - **One notebook, nothing else.** No helper modules, classes, config files, CLI or tests. The user asked for the simplest version that works.
  - **Runs fully offline.** It uses only Ollama's local HTTP API (`localhost:11434`), and models are downloaded ahead of time.
  - **Timings come from Ollama.** `total_duration`, `eval_count` and the other fields come from Ollama, so the notebook doesn't time calls itself. One warm-up call per model keeps load time out of the numbers.
  - **Settings:** `temperature: 0` and `stream: False`, so runs are repeatable and the code stays simple.
  - **ASCII format:** grayscale, the 10-character ramp `" .:-=+*#%@"`, 80 columns, and height scaled by 0.5 because characters are about twice as tall as they are wide.
  - **Tasks:** `describe`, `flip`, `invert` and `edit`. `flip` and `invert` have answers that can be computed, so the output can be scored as well as judged by eye.
  - **Validity check:** the output must have the same number of lines and the same line lengths as the input.
  - **Starting models:** `llama3.2:3b`, `qwen2.5:7b` and `gemma2:9b`. These are placeholders to change as needed.
  - **Optional vision baseline:** a vision model such as `llava` describes the real image, to answer "is ASCII actually faster?"
  - **Left out on purpose:** color, streaming, retries, async and prompt-tuning frameworks.
- **Next:** build `poc.ipynb` from `PLAN.md`.
