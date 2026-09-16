# Project Log

The newest entries go at the top. Each entry covers what was done, what changed, and why.

---

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
