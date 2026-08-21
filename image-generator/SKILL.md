---
name: image-generator
description: Generate, edit, or batch-generate raster images with the bundled GPT Image CLI. Use when a user asks for image generation through this local skill, needs batch image jobs, wants to edit existing images with prompts, or needs to configure or diagnose the image API runtime.
---

# Image Generator

Use the bundled scripts instead of rewriting image API calls. The scripts handle backend selection, request validation, retries, output decoding, and batch concurrency.

## Workflow

1. Use `scripts/image_gen.py` for image generation, image editing, dry runs, and JSONL batch jobs.
2. Use `scripts/runtime.py` only when dependencies or API configuration need setup or diagnosis.
3. Prefer `--backend auto`. It uses local Codex OAuth auth when available, then falls back to the OpenAI SDK with `OPENAI_API_KEY`.
4. Keep prompts exactly as the user requested unless they ask for prompt improvement.
5. Write outputs to a project-local path unless the user gives another destination.

## Common Commands

Install dependencies in the active environment:

```bash
python -m pip install -r image_generator/scripts/requirements.txt
```

Create or update the shared runtime:

```bash
python image_generator/scripts/runtime.py bootstrap
python image_generator/scripts/runtime.py doctor
```

Configure OpenAI SDK credentials when Codex OAuth is unavailable:

```bash
python image_generator/scripts/runtime.py config \
  --api-key "your-api-key" \
  --model gpt-image-2 \
  --backend openai
```

Generate one image:

```bash
python image_generator/scripts/image_gen.py generate \
  --backend auto \
  --prompt "A clean 16:9 presentation cover about blockchain infrastructure" \
  --size 2048x1152 \
  --quality medium \
  --out output/image.png
```

Edit an existing image:

```bash
python image_generator/scripts/image_gen.py edit \
  --backend auto \
  --image input.png \
  --prompt "Keep the layout, improve contrast, and make the title more readable" \
  --out output/edited.png
```

Run a dry run before calling the API:

```bash
python image_generator/scripts/image_gen.py generate \
  --backend auto \
  --prompt "test image" \
  --out output/test.png \
  --dry-run
```

## Batch Jobs

Use JSONL where each non-empty line is either a prompt string or an object with a `prompt` field:

```jsonl
{"prompt":"Slide cover, blue-green editorial tech style","out":"slide_01.png"}
{"prompt":"Architecture overview diagram as a polished slide","out":"slide_02.png"}
```

Then run:

```bash
python image_generator/scripts/image_gen.py generate-batch \
  --backend auto \
  --input jobs.jsonl \
  --out-dir output/images \
  --size 2048x1152 \
  --quality medium
```

## Notes

- Default model: `gpt-image-2`.
- Default runtime home: `~/.image-generator`; override with `IMAGE_GENERATOR_HOME`.
- New runtime config uses `IMAGE_GENERATOR_IMAGE_MODEL` and `IMAGE_GENERATOR_IMAGE_BACKEND`.
- Legacy `IMAGE2_*` and `CODEX_PPT_*` variables are still read for compatibility.
- For transparent backgrounds, use `--model gpt-image-1.5 --background transparent`; `gpt-image-2` does not support transparent backgrounds.
