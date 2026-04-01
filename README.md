# orange3-imageanalytics (WASM fork)

Experimental fork of [orange3-imageanalytics](https://github.com/biolab/orange3-imageanalytics)
for running Orange3 in the browser via Pyodide + PyQt6 WASM.

Forked from upstream [`0.13.0`](https://github.com/biolab/orange3-imageanalytics/tree/0.13.0)
(commit `ae36a0a`).

## Changes

- `widgets/owimageimport.py`: Replaced `getExistingDirectory()` with non-blocking
  QFileDialog + open() + accepted signal.

- `widgets/owsaveimages.py`: Replaced blocking `get_save_filename()` loop (exec() +
  QMessageBox.question) with callback chain: `_open_save_dialog` → `_confirm_overwrite`
  → `_finish_save_as`.

- `image_embedder.py`: Replaced async httpx `ServerEmbedderCommunicator` with
  synchronous `requests.Session` calls (asyncio.run() fails in Pyodide because
  the WebLoop is already running).

## Known limitations

- api.garaza.io server embedders (Inception v3, VGG-16) are blocked by CORS.
  Requires a CORS proxy or custom embedding API to use.
- SqueezeNet: WASM NumPy-based CNN inference is slow for batch image processing.
- Image Grid widget: Blocked by openTSNE (C extension not available in Pyodide).

## Install (Pyodide)

```python
await micropip.install(
    "https://team-monolith-product.github.io/orange3-imageanalytics/orange3_imageanalytics-0.13.0-py3-none-any.whl"
)
```

## Release procedure

1. Make changes on `master` and commit.

2. Build the wheel:

```bash
python -m build --wheel
```

3. Create a GitHub Release:

```bash
gh release create wasm-{version} dist/orange3_imageanalytics-{version}-py3-none-any.whl \
    --title "{version}-wasm" --notes "..."
```

4. Deploy to GitHub Pages (serves the wheel with CORS):

```bash
git checkout gh-pages
cp dist/orange3_imageanalytics-{version}-py3-none-any.whl .
git add *.whl
git commit -m "deploy: orange3_imageanalytics-{version}"
git push origin gh-pages
git checkout master
```
