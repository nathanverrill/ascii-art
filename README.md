# ascii-art

An experiment: can local LLMs work on images faster and more cheaply if they get the
image as **ASCII art** instead of pixels?

```
real image  ->  ASCII art  ->  LLM (Ollama, local)  ->  ASCII art  ->  rendered image
```

Text-only models can't see pixels, and vision models spend a lot of tokens on them.
A small ASCII grid (for example 80×40 characters) keeps the rough shape and shading
of an image in a few thousand characters. This project checks how well that works
and how fast it is.

Everything runs **locally with Ollama and no internet access**.

## What gets measured

For each model and each task:

- **Speed:** total time, time to first token, and tokens per second (Ollama reports these)
- **Size:** input and output token counts
- **Validity:** does the output keep the grid shape (same width and height, only allowed characters)?
- **Quality:** a side-by-side image of input and output, judged by eye

## Example tasks

- **Describe:** "What is in this picture?" (ASCII in, text out)
- **Transform:** "Flip it horizontally", "invert light and dark", "add a border" (ASCII in, ASCII out)
- **Edit:** "Add a sun in the top right corner" (ASCII in, ASCII out)

## Requirements

- Python 3.10 or newer
- [Ollama](https://ollama.com) running locally (`ollama serve`)
- The models you want to test, pulled while you still have internet access:

  ```bash
  # text models, all <= ~9 GB, sized for a 32 GB Apple Silicon Mac
  for m in llama3.2:1b llama3.2:3b gemma3:4b qwen2.5:7b llama3.1:8b qwen3:8b gemma2:9b gemma3:12b phi4:14b; do ollama pull $m; done
  # vision models for the pixel baseline
  ollama pull llava:7b
  ```

  The notebook loads one model at a time and unloads it when it's done, and skips any model that isn't installed.

- Python packages:

  ```bash
  python3 -m venv .venv
  .venv/bin/pip install jupyter pillow numpy requests pandas matplotlib
  ```

## Usage

```bash
ollama serve                        # in one terminal
.venv/bin/jupyter notebook poc.ipynb  # in another

# or run headless
.venv/bin/jupyter nbconvert --to notebook --execute --inplace poc.ipynb
```

The notebook only tests models from `MODELS` that are installed. Put test images
(`.png`) in `images/`; if the folder is empty, the notebook generates four simple shapes. Results are written to `results/`
(`benchmarks.csv`, `vision_baseline.csv`, `report.html` and the rendered PNGs).

## Layout

```
README.md
PLAN.md         # how the POC notebook is built
LOG.md          # processes, changes, decisions
poc.ipynb       # the whole experiment
images/         # input images
results/        # benchmark CSV and rendered outputs
```

## Status

This is an experimental proof of concept. See [PLAN.md](PLAN.md).
