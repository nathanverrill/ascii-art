# POC Plan: `poc.ipynb`

The goal is one notebook that runs from top to bottom with no helper modules, classes or
config files. Each section below is one or two notebook cells.

## 0. Setup

```python
import time, json, requests
import numpy as np, pandas as pd
from pathlib import Path
from PIL import Image, ImageDraw, ImageFont

OLLAMA = "http://localhost:11434/api/generate"
MODELS = ["llama3.2:3b", "qwen2.5:7b", "gemma2:9b"]
CHARS  = " .:-=+*#%@"          # dark ramp, light -> dark
WIDTH  = 80                    # ASCII columns
Path("results").mkdir(exist_ok=True)
```

Check that Ollama is running: `requests.get("http://localhost:11434/api/tags")`.

## 1. Image to ASCII

```python
def image_to_ascii(path, width=WIDTH):
    img = Image.open(path).convert("L")
    h = int(img.height / img.width * width * 0.5)   # characters are about twice as tall as they are wide
    px = np.array(img.resize((width, h))) / 255
    idx = ((1 - px) * (len(CHARS) - 1)).round().astype(int)
    return "\n".join("".join(CHARS[i] for i in row) for row in idx)
```

Print the result for one image to check it by eye.

## 2. Call Ollama

```python
def ask(model, prompt):
    r = requests.post(OLLAMA, json={"model": model, "prompt": prompt, "stream": False,
                                    "options": {"temperature": 0}}).json()
    return r["response"], {
        "total_s":   r["total_duration"] / 1e9,
        "load_s":    r["load_duration"] / 1e9,
        "in_tokens": r["prompt_eval_count"],
        "out_tokens": r["eval_count"],
        "tok_per_s": r["eval_count"] / (r["eval_duration"] / 1e9),
    }
```

Use Ollama's own timing fields instead of timing the call yourself. Run one warm-up call
per model first, so model load time does not skew the numbers.

## 3. Tasks and prompts

A plain dict:

```python
TASKS = {
    "describe": "Describe what this ASCII art shows in one sentence.",
    "flip":     "Mirror this ASCII art horizontally. Output only the ASCII art, same size.",
    "invert":   f"Invert brightness using this ramp: '{CHARS}'. Output only the ASCII art, same size.",
    "edit":     "Add a small sun in the top-right corner. Output only the ASCII art, same size.",
}
prompt = f"{instruction}\n\n```\n{ascii}\n```"
```

## 4. Check the output

```python
def clean(text):             # remove code fences if the model added them
    return text.strip().strip("`").strip()

def grid_ok(inp, out):       # same shape?
    a, b = inp.splitlines(), out.splitlines()
    return len(a) == len(b) and all(len(x) == len(y) for x, y in zip(a, b))
```

For `flip` and `invert` the correct answer can be computed, so also record
`char_accuracy` (the share of characters that match the expected grid).

## 5. ASCII to image

```python
def ascii_to_image(text, path):
    lines = text.splitlines() or [""]
    font = ImageFont.load_default()
    w, h = 6, 11                                   # cell size for the default font
    img = Image.new("L", (max(map(len, lines)) * w, len(lines) * h), 255)
    d = ImageDraw.Draw(img)
    for y, line in enumerate(lines):
        d.text((0, y * h), line, fill=0, font=font)
    img.save(path)
```

## 6. Benchmark loop

```python
rows = []
for img in sorted(Path("images").glob("*")):
    art = image_to_ascii(img)
    for model in MODELS:
        for task, instr in TASKS.items():
            out, stats = ask(model, f"{instr}\n\n```\n{art}\n```")
            out = clean(out)
            rows.append({"image": img.name, "model": model, "task": task,
                         "grid_ok": task != "describe" and grid_ok(art, out), **stats})
            if task != "describe":
                ascii_to_image(out, f"results/{img.stem}_{model.replace(':','-')}_{task}.png")
df = pd.DataFrame(rows)
df.to_csv("results/benchmarks.csv", index=False)
```

## 7. Results

- `df.groupby(["model", "task"]).mean(numeric_only=True)`
- One bar chart of `tok_per_s` by model and one of `grid_ok` rate by model
- Show the input and output images side by side for a few examples
- **Baseline (optional):** if a vision model is pulled (for example `llava`), run the
  `describe` task on the real image using the `images` field in the Ollama request, and
  compare its `in_tokens` and `total_s` with the ASCII version. This is the core
  "is ASCII faster?" comparison.

## Order of work

1. Sections 0 and 1: check that the ASCII looks right for 2–3 images.
2. Section 2 with one model and the `describe` task.
3. Sections 4 and 5: check that validity and rendering work.
4. Section 6 with all models, then section 7.
5. Add the vision baseline if there is time.

## Out of scope for the POC

Not included: color, streaming, retries, async, a CLI, packaging, tests, or prompt tuning
frameworks. Add them only if the results justify it.
