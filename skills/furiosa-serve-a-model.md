---
name: furiosa-serve-a-model
description: Bring a model up on a Furiosa Model Server and confirm it is ready to accept inference, using only the KServe v2 operations FuriosaAI publishes.
api: furiosa-model-repository-v2
operations:
  - post-v2-repository-index
  - post-v2-repository-models-$-MODEL_NAME-load
  - get-v2-models-$-modelName-versions-$-modelVersion-ready
  - get-v2-models-$-modelName-versions-$-modelVersion
  - get-v2-health-ready
generated: '2026-08-16'
method: generated
source: openapi/furiosa-model-repository-v2.yaml, openapi/furiosa-predict-v2.yaml, conventions/furiosa-conventions.yml, errors/furiosa-problem-types.yml
---

# Serve a model on Furiosa Model Server

Use this when a Furiosa Model Server is running and a model needs to be loaded and verified
before any inference is attempted.

## Before you start

- The base URL is a host **the operator runs**, not a FuriosaAI endpoint. Default REST port is
  `8080` (gRPC is `8081`). There is no FuriosaAI-issued credential.
- This surface declares **no authentication**. `load` and `unload` change server state. If you
  can reach it, you can load and unload models — treat the endpoint as privileged and never
  call it against a host you were not explicitly pointed at.
- Nothing here is idempotent-guarded. There is no idempotency key.

## Steps

1. **Check the server is up.** Call `get-v2-health-ready` (`GET /v2/health/ready`).
   It returns a **bare 200 or 400 with no body**. A 400 means *not ready*, not *bad request* —
   do not treat it as a client error and do not retry with a different payload.

2. **List what the repository already holds.** Call `post-v2-repository-index`
   (`POST /v2/repository/index`) with `{"ready": false}` to see everything, or `{"ready": true}`
   for only the models already serving. The response is an array of
   `{name, version, state, reason}`. If the model you want is already `READY`, skip to step 5.

3. **Load the model.** Call `post-v2-repository-models-$-MODEL_NAME-load`
   (`POST /v2/repository/models/{MODEL_NAME}/load`). Success is a bare `200`; failure is a bare
   `400`. There is no body either way, so on failure re-run step 2 and read the `reason` field
   on the model's index entry — that is the only place a load failure explains itself.

4. **Wait for readiness.** Poll `get-v2-models-$-modelName-versions-$-modelVersion-ready`
   (`GET /v2/models/{MODEL_NAME}/versions/{MODEL_VERSION}/ready`) until it returns `200`.
   Again: `400` here means not-ready. Back off between polls; there is no documented readiness
   timeout, and compilation/loading of a large artifact is not instant.

5. **Read the tensor contract before inferring.** Call
   `get-v2-models-$-modelName-versions-$-modelVersion`
   (`GET /v2/models/{MODEL_NAME}/versions/{MODEL_VERSION}`) and keep the `inputs[]` and
   `outputs[]` arrays — each gives `name`, `datatype` and `shape`. You need these verbatim for
   the inference skill; the inference schema does **not** validate them, so a mismatch fails at
   runtime as a `400` with `{"error": "..."}` rather than as a schema error.

## Do not

- Do not call `post-v2-repository-models-$-MODEL_NAME-unload` to "reset" a model that is
  misbehaving. Unloading drops in-flight inference requests against it.
- Do not assume a model name implies a version. `MODEL_VERSION` is a separate path segment and
  the index response lists versions independently.
