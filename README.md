# Diffusion + YOLO + CLIP

Type a text prompt, generate an image with **Stable Diffusion**, then check the result with two other models:

- **YOLO** finds the objects in the image and says how confident it is.
- **CLIP** compares the image against captions and shows which one fits best.

Everything runs in a Google Colab notebook with a simple web UI built with Gradio.

```
prompt  ->  Stable Diffusion  ->  image  ->  YOLO  (what objects are in it?)
                                         \->  CLIP  (which caption matches it?)
```

## Models used

| Step | Model |
|------|-------|
| Image generation | `runwayml/stable-diffusion-v1-5` (via `diffusers`) |
| Object detection | `yolo11n.pt` (via `ultralytics`) |
| Image-text matching | `openai/clip-vit-base-patch32` (via `transformers`) |

## Requirements

- Google Colab with a GPU runtime (tested on a free **T4**)
- Python packages:

```
diffusers transformers accelerate ultralytics gradio torch
```

## Setup

1. Open the notebook in Colab.
2. Go to **Runtime -> Change runtime type** and select a **GPU**.
3. Run the cells from top to bottom:
   1. Check the GPU (`!nvidia-smi`)
   2. Install the packages
   3. Load Stable Diffusion
   4. Load YOLO
   5. Load CLIP
   6. Run the UI cell

The first run downloads several GB of model weights, so it takes a few minutes.

## Using the UI

When the last cell finishes, it prints a link ending in `gradio.live`. Open it in your browser.

| Control | What it does |
|---------|--------------|
| **Prompt** | Describes the image you want |
| **Extra captions** | Optional, one per line. CLIP compares these against your prompt |
| **Steps** | More steps means slower but usually better images |
| **Guidance scale** | How strictly the image follows the prompt |
| **Seed** | `-1` for random. Reuse a seed to get the same image again |

After you click **Generate** you get:

- the generated image
- the same image with YOLO boxes drawn on it
- a table of detected objects and their confidence
- a bar chart showing which caption CLIP thinks fits best
- the seed that was used

## How to read the results

- **YOLO confidence** is between 0 and 1. A cat at `0.62` means YOLO is fairly sure, not certain. AI-generated images often score lower than real photos.
- **CLIP scores** are shares that add up to 100% across the captions you give it. They only mean something when you compare captions. If you give one caption, it always gets 100%.
- To make CLIP useful, add near-miss captions such as `a cat wearing a blue hat` or `a cat without a hat`.

## Example

Prompt:

```
a cat wearing a red hat, realistic photo
```

Extra captions:

```
a dog wearing a blue hat
a car on a street
a bowl of food on a table
```

Expected result: YOLO detects a `cat`, and CLIP gives the prompt close to 100%.

## Troubleshooting

**CLIP gives strange results (like 50% / 49% for unrelated captions)**
Check that the `image` variable really holds the picture you think it does. If you run cells out of order or reuse the name `image` for something else, CLIP will score the wrong picture. Use `display(image)` to look at it.

**Gradio error about the number of outputs**
The number of values returned by `run()` must match the number of components in `outputs=[...]`. If you add or remove a UI element, change both places.

**`gradio` not found**
Run `!pip install -q gradio` in a cell, then run the UI cell again.

**Out of GPU memory**
Restart the runtime, run the cells once, and avoid loading the models twice. Lower the steps or generate one image at a time.

**Black or blank image**
Stable Diffusion's safety checker can replace an image with a black one. Try a different prompt or seed.

**Some cells show "Warning: unauthenticated requests to the HF Hub"**
This is harmless. Set an `HF_TOKEN` in Colab secrets for faster downloads.

## Ideas for next steps

- Generate several images per prompt and rank them by CLIP score
- Compare CLIP scores for color and counting prompts (`two red apples`)
- Try a larger YOLO model (`yolo11s.pt`, `yolo11m.pt`) for better detection
- Save results (image, detections, scores) to a folder or CSV

## Notes

- The Gradio link only works while the Colab session is running.
- Model weights have their own licenses. Check each model's page before using outputs commercially.
