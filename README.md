# trellis-runpod

On-demand **TRELLIS image → 3D (GLB)** as a RunPod **serverless** worker. Send an image, it
generates on a GPU only while running, returns the GLB. Scales to zero when idle — you pay per
second of use, no GPU rental. Anyone can deploy this with their **own** RunPod key.

Built on [camenduru/trellis-tost](https://github.com/camenduru/trellis-tost)'s proven TRELLIS
build (CUDA 12.6 + torch 2.5.1 + flash-attn + kaolin/nvdiffrast, **weights baked in**); the
handler (`worker_runpod.py`) is a clean inline contract — no external storage.

## I/O contract
```
input  = {"image_b64": <png base64>, "seed": 42, "simplify": 0.95,
          "texture_size": 1024, "preprocess_image": true}
output = {"glb_b64": <glb base64>}     |     {"error": "..."}
```

## Deploy it (one-time, ~2 clicks)
1. RunPod console → **Serverless → New Endpoint → Deploy from GitHub** → connect GitHub, pick
   **`ZheningHuang/trellis-runpod`**.
2. GPU: **24 GB+** (e.g. RTX 4090 / A5000). **Min Workers = 0** (on-demand), **Max Workers = N**
   (how many you want in parallel). Container disk ~30 GB.
3. Deploy → wait for the first build (RunPod downloads CUDA + weights, ~20–40 min, one-time) →
   copy the **Endpoint ID**.

## Call it
Any HTTP client against `https://api.runpod.ai/v2/<endpoint_id>/run` with
`Authorization: Bearer <RUNPOD_API_KEY>`, or the bundled client
(`agent/providers/runpod_trellis.py` in agentic-LiteReality):
```python
RunPodTrellisService(max_parallel=8).reconstruct_many(
    {"Chair0": "chair.png", ...}, out_dir="out/")   # parallel, prints time + cost
```

Set in your env: `RUNPOD_API_KEY` and `RUNPOD_TRELLIS_ENDPOINT=<endpoint id>`.
