---
name: furiosa-llm-openai-client
description: Call a Furiosa-LLM OpenAI-compatible server correctly, including the places where Furiosa deviates from OpenAI and from the OpenResponses specification.
api: furiosa-llm-openai-server
operations:
  - GET /v1/models
  - POST /v1/chat/completions
  - POST /v1/completions
  - POST /v1/responses
  - POST /v1/embeddings
  - GET /metrics
generated: '2026-08-16'
method: generated
source: >-
  https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html,
  https://developer.furiosa.ai/latest/en/furiosa_llm/responses-api.html,
  conventions/furiosa-conventions.yml, conformance/furiosa-conformance.yml
x-operations-note: >-
  FuriosaAI publishes no OpenAPI for this surface, so the operations above are HTTP method+path
  as documented in the serving reference, not operationIds. Nothing here is invented; every path
  and every deviation below is stated in the FuriosaAI docs.
---

# Call a Furiosa-LLM OpenAI-compatible server

Use this when talking to a server started with `furiosa-llm serve <ARTIFACT_PATH>`. The standard
OpenAI SDK works unmodified — point `base_url` at the server (default
`http://localhost:8000/v1`) and set `api_key` to whatever the operator configured, or `"EMPTY"`
if they configured nothing.

## Steps

1. **Resolve the model id first.** Call `GET /v1/models` and use `data[0].id`.
   The `model` field in every request is **required by the client and ignored by the server** —
   one `furiosa-llm serve` process hosts exactly one model, so there is no routing by name.
   The response also carries Furiosa-specific fields worth reading before you build a prompt:
   `max_prompt_len`, `max_context_len`, `artifact_id` and `runtime_config`.

2. **Generate.** `POST /v1/chat/completions` for chat-template models, `POST /v1/completions`
   for raw text, `POST /v1/responses` for the OpenResponses shape, `POST /v1/embeddings` for
   embedding models. Set `stream: true` for SSE.

3. **Budget the context.** `max_completion_tokens` plus prompt length must not exceed
   `max_context_len`. Leave `max_completion_tokens` null to let the server use the maximum
   available given the prompt. On `/v1/completions`, `max_tokens` defaults to **16** — a common
   surprise.

## Deviations you must code around

- **Sampling defaults are three-tier.** `temperature`, `top_p`, `top_k`, `min_p`,
  `repetition_penalty`, `max_tokens` resolve as: request body → the model's
  `generation_config.json` → the documented API default. Omitting a parameter does **not**
  guarantee the documented default. Send the value explicitly if it matters.
- **`n` is limited to 1.** Asking for multiple completions will not work.
- **`use_beam_search` and `stream` are mutually exclusive.**
- **`reasoning` is a Furiosa extension.** On reasoning models, the chain appears at
  `response.choices[].message.reasoning` (or `.delta.reasoning` when streaming). The field is
  **absent** on responses without reasoning content — accessing it raises `AttributeError`.
  Always guard with `hasattr`. Enable reasoning with the server flag `--reasoning-parser <name>`
  and, for Qwen3/EXAONE4, `extra_body={"chat_template_kwargs": {"enable_thinking": true}}`.
- **Tool calling requires server flags.** `tools`/`tool_choice` only work if the operator
  started the server with `--enable-auto-tool-choice --tool-call-parser <parser>`. You cannot
  turn it on from the request.
- **`logprobs` / `top_logprobs` are experimental.**
- **Responses API is partial.** `input_image`, `input_audio` and `input_file` are accepted and
  **silently ignored**. Built-in tools (`web_search`, `file_search`, `code_interpreter`,
  `computer_use`, `mcp`) are **not supported** — custom function tools only. `background=true`
  has no effect. `truncation="auto"` falls back to `"disabled"`. `presence_penalty` and
  `frequency_penalty` are accepted but non-functional.
- **Responses persistence is opt-in and volatile.** `GET /v1/responses/{id}`,
  `POST /v1/responses/{id}/cancel` and `previous_response_id` require the operator to have
  started the server with `--enable-responses-api-store`. The store is in-memory, defaults to
  10000 entries with a 3600s TTL, and is lost on restart. If you need durable multi-turn
  context, build it manually by appending prior `output` items to `input`.

## Retries, limits and errors

- **No idempotency and no rate limits.** There is no idempotency key, no `429`, and no
  `RateLimit-*` headers. A retry is a second full generation.
- Queue pressure is visible only on `GET /metrics` — `furiosa_llm_num_requests_waiting` and
  `furiosa_llm_kv_cache_usage_percent`. Since SDK **2026.3.0** `/metrics` is **GET-only**;
  a `POST` returns `405`.
- Errors follow the OpenAI error object. FuriosaAI does not publish an error-code table for
  this surface.

## Also available

`POST /score` and `POST /rerank` (also `/v1/score`, `/v1/rerank`, `/v2/rerank`) are vLLM
extensions, currently supported only for Qwen3-Rerank models. `POST /tokenize`,
`POST /detokenize` and `GET /tokenizer_info` wrap the HuggingFace tokenizer. `GET /version`
reports Furiosa SDK component versions.
