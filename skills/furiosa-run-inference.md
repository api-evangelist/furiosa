---
name: furiosa-run-inference
description: Run a KServe v2 inference request against a model loaded on a Furiosa Model Server, building the tensor payload from the model's own declared metadata.
api: furiosa-server-predict-v2
operations:
  - get-v2
  - get-v2-models-$-modelName-versions-$-modelVersion
  - post-v2-models-$-MODEL_NAME-versions-$-MODEL_VERSION-infer
generated: '2026-08-16'
method: generated
source: openapi/furiosa-predict-v2.yaml, data-model/furiosa-data-model.yml, errors/furiosa-problem-types.yml
---

# Run inference on a Furiosa Model Server

Use this once a model is loaded and ready (see `furiosa-serve-a-model`).

## Steps

1. **Optional — confirm what the server supports.** Call `get-v2` (`GET /v2/`). It returns
   `{name, version, extensions[]}`. The `extensions` array tells you whether the model-repository
   extension is enabled on this deployment.

2. **Fetch the tensor contract.** Call `get-v2-models-$-modelName-versions-$-modelVersion`
   (`GET /v2/models/{MODEL_NAME}/versions/{MODEL_VERSION}`). Read `inputs[]` — each entry is
   `{name, datatype, shape}`. **Never guess these.** This is the single most common cause of a
   failed inference on this API.

3. **Build the request.** Call
   `post-v2-models-$-MODEL_NAME-versions-$-MODEL_VERSION-infer`
   (`POST /v2/models/{MODEL_NAME}/versions/{MODEL_VERSION}/infer`) with an `inference_request`:

   - `inputs` (**required**) — one `request_input` per declared model input:
     `{name, shape, datatype, data}`. `name`, `shape` and `datatype` must match step 2 exactly.
   - `data` is a **flattened row-major array**. Its element type is governed by the sibling
     `datatype` field, not by the JSON schema — so `datatype: "UINT32"` means integers,
     `"BOOL"` means booleans, `"FP32"` means floats. A generated client cannot type this for you.
   - `outputs` (optional) — `[{name}]` to request only specific outputs. Omit to get all.
   - `id` (optional) — a caller-supplied correlation string, echoed back on the response. This
     is the **only** request-correlation mechanism on this API; there is no request-id header.
   - `parameters` (optional) — open key/value extension bag.

   FuriosaAI ships a worked example in the spec's `x-examples`:
   `{"id": "42", "inputs": [{"name": "input0", "shape": [2,2], "datatype": "UINT32", "data": [1,2,3,4]}], "outputs": [{"name": "output0"}]}`

4. **Read the response.** An `inference_response` gives `model_name`, `model_version`, `id`,
   and `outputs[]` of `{name, shape, datatype, data}` — same flattening rule as the inputs.
   Reshape `data` using `shape` before interpreting it.

## Failure handling

- A `400` returns `{"error": "<prose>"}`. There is **no error code registry** — the string is
  human prose and must not be pattern-matched as if it were stable.
- `"Model {name} not found"` / `"Model {name} with version {version} not found"` are the
  server's literal messages when the model is absent from the repository. Go back to
  `furiosa-serve-a-model`.
- Any other `400` on `/infer` is almost always a tensor mismatch. Re-read step 2 and compare
  `name`, `shape` and `datatype` field by field before retrying.

## Do not

- Do not blind-retry a failed inference. There is no idempotency key and no dedupe window; a
  retry is a second full execution on the NPU.
- Do not treat a `400` from `/v2/health/*` or `/v2/models/.../ready` as the same kind of error.
  On those paths a bare `400` means *not ready*.
