# MD-llm

Provider configurations and user help for MessyDesk's LLM services. There is no service code
here: the LLM providers (Ollama, vLLM, OpenAI, Azure OpenAI, Google Gemini) are called directly by
MD-consumers' `llm-openai` and `llm-gemini` adapters. This repository plays the part a service
repository normally does: its descriptor (`/config`) and its help (`help/index.md`).

Design: MessyDesk `plan/llm-adapter.md`.

```
providers/ollama.json   md-llm-ollama   llm-openai   on-premise   http://localhost:11434/v1
providers/vllm.json     md-llm-vllm     llm-openai   on-premise   http://localhost:8000/v1
providers/openai.json   md-llm-openai   llm-openai   external     OPENAI_API_KEY
providers/azure.json    md-llm-azure    llm-openai   external     AZURE_OPENAI_API_KEY
providers/gemini.json   md-llm-gemini   llm-gemini   external     GOOGLE_API_KEY
help/index.md           user help, sent to MessyDesk by each consumer
compose.llm.yaml        consumers (and an optional Ollama) for MessyDesk/local/compose.yaml
```

## Running a consumer

One consumer per provider. In MD-consumers:

```bash
CONFIG_JSON_PATH=../MD-llm/providers/ollama.json SERVICE_TOKEN=... MD_URL=http://localhost:8200 node src/index.mjs
```

```bash
CONFIG_JSON_PATH=../MD-llm/providers/azure.json AZURE_OPENAI_API_KEY=... SERVICE_TOKEN=... node src/index.mjs
```

`TOPIC` defaults to the file's `service.id`. A consumer handles one job at a time; start several
with the same config for parallel calls to one provider.

With podman/docker, next to MessyDesk's local compose:

```bash
cd ../MessyDesk/local
podman-compose -f compose.yaml -f ../../MD-llm/compose.llm.yaml --profile llm up -d
```

## The config file

```jsonc
{
  "service": { ... },      // the service descriptor, registered with MessyDesk as is
  "provider": { ... },     // for the adapter only; never sent anywhere
  "help": "../help/index.md"   // markdown, relative to this file; sent to MessyDesk at start-up
}
```

### `service`

A normal service descriptor (MessyDesk `wiki/service-descriptor-format.md`) with
`external_tasks: "prompts"`, so the user's prompts become its tasks, plus the fixed `autotag` task.

| Field | Notes |
|---|---|
| `location`, `access` | Badges in the UI: `on-premise`/`external`, `open source`/`proprietary` |
| `service_groups` | Who may use it. Token limits are set on these groups in MessyDesk (admin) |
| `params_help` | `temperature`, `max_output_tokens`: asked from the user for each run |
| `models.<id>` | The id is what MessyDesk stores on the result |
| `models.<id>.family` | Same model on several providers: same `family`, so the UI shows it once and lets the user pick the provider |
| `models.<id>.supported_types` / `supported_formats` | `text` and/or `image`; which files the model is offered for |
| `models.<id>.max_input_tokens` | Longer texts fail with a clear message instead of being cut |
| `models.<id>.max_output_tokens` | Default answer length when the user gives none (else 4096) |
| `models.<id>.max_image_edge` | Images are scaled down to this (default 2048 px) |
| `models.<id>.temperature: false` | The model takes no temperature (reasoning models such as GPT-5) |
| `models.<id>.structured_output: false` | JSON mode with the schema in the instructions, for models without JSON Schema support |

### `provider`

| Field | Adapter | Notes |
|---|---|---|
| `name` | both | Shown in `response.json` metadata; defaults to the service id |
| `base_url` | llm-openai | Chat Completions base URL, ending in `/v1` (Azure: `https://<resource>.openai.azure.com/openai/v1/`) |
| `api_key_env` | both | **Name** of the env var holding the key. Keys never go in this file |
| `auth_header` | llm-openai | `api-key` for Azure; default is `Authorization: Bearer` |
| `api_version` | llm-openai | Adds `?api-version=` (classic Azure deployment URLs) |
| `headers` | llm-openai | Extra request headers |
| `model_map` | both | Model id → the provider's own name (Azure deployment, Ollama tag, vLLM path) |
| `max_tokens_param` | llm-openai | `max_completion_tokens` (default, OpenAI/Azure) or `max_tokens` (Ollama, vLLM) |
| `json_mode` | llm-openai | `object` for servers without `json_schema` response format |
| `strict_json` | llm-openai | `true` for OpenAI strict structured outputs (the schema must allow it) |
| `store` | llm-openai | `false`: ask OpenAI/Azure not to store the request |
| `timeout_ms` | both | Per request (default 120 000) |
| `retry` | both | `{ max_attempts, initial_delay_ms }` for 429/5xx; `Retry-After` is honoured |

## Notes per provider

- **Ollama** reads only the first `num_ctx` tokens of a prompt and drops the rest without an
  error. Start Ollama with a context that fits the texts (`OLLAMA_CONTEXT_LENGTH=8192`) and keep
  `max_input_tokens` a little under it. Models must be pulled first (`ollama pull gemma3:4b`).
- **vLLM** serves one model per server by default; give each its own base URL (a second config
  file) or run vLLM behind a router.
- **OpenAI / Azure** GPT-5 models are reasoning models: they take no temperature, and their
  reasoning tokens count as output tokens.
- **Gemini** receives images inline; nothing is uploaded to Google's file store.
