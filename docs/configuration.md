# Configuration Guide

## Setup

```bash
cp config/config.example.toml config/config.toml
```

Edit `config/config.toml` with your API keys.

## Required API Keys

### LLM Provider (Steps 1-2)

The pipeline uses an OpenAI-compatible chat API. Choose a provider in `[llm]`:

#### DeepSeek (default)

1. Go to [platform.deepseek.com](https://platform.deepseek.com)
2. Create an account and generate an API key
3. Add to config: `deepseek_api_key = "sk-..."`

#### OpenRouter (optional)

OpenRouter exposes many models via a single OpenAI-compatible endpoint.

1. Go to [openrouter.ai](https://openrouter.ai)
2. Create an account and generate an API key
3. Configure:

```toml
[api_keys]
openrouter_api_key = "sk-or-..."

[llm]
provider = "openrouter"
model = "deepseek/deepseek-chat"   # any slug from openrouter.ai/models
```

Pick any model ID from the [OpenRouter model catalog](https://openrouter.ai/models) — for example `anthropic/claude-3.5-sonnet`, `openai/gpt-4o`, or `google/gemini-2.0-flash-001`. Leave `model` empty to use the OpenRouter default (`deepseek/deepseek-chat`).

**Choose the model in Python** (overrides TOML and env):

```python
from src.standard.pipeline import run_standard_pipeline
from src.standard.llm_client import resolve_llm_config

config["llm"]["provider"] = "openrouter"

# Option A: pass model to the pipeline
result = run_standard_pipeline(text, config, llm_model="anthropic/claude-3.5-sonnet")

# Option B: resolve config first
llm = resolve_llm_config(config, model="openai/gpt-4o")
print(llm["model"])  # openai/gpt-4o
```

#### Provider defaults

| Provider | Default `base_url` | Default `model` | Config key |
|----------|-------------------|-----------------|------------|
| `deepseek` | `https://api.deepseek.com` | `deepseek-chat` | `api_keys.deepseek_api_key` |
| `openrouter` | `https://openrouter.ai/api/v1` | `deepseek/deepseek-chat` | `api_keys.openrouter_api_key` |

Set `base_url` in `[llm]` to override the provider preset (e.g. a self-hosted OpenAI-compatible proxy). Leave empty to use the default for the selected provider.

**Custom endpoint example:**

```toml
[llm]
provider = "openrouter"
base_url = "https://my-proxy.example.com/v1"
model = "deepseek/deepseek-chat"
```

### Niutrans API Key (optional)

Required only when `pipeline.step4_engine = "niutrans"`. The default (`"google"`) uses Google Translate for Step 4 and needs no Niutrans key.

1. Go to [niutrans.com](https://niutrans.com)
2. Register and get a free API key (free tier available)
3. Add to config: `niutrans_api_key = "your-key"` and set `step4_engine = "niutrans"`

## Configuration Options

```toml
[general]
target_language = "en"    # Final output language
log_level = "info"        # debug, info, warning, error

[api_keys]
deepseek_api_key = ""     # Required when llm.provider = "deepseek"
openrouter_api_key = ""   # Required when llm.provider = "openrouter"
niutrans_api_key = ""     # Required only when pipeline.step4_engine = "niutrans"

[llm]
provider = "deepseek"     # "deepseek" | "openrouter"
base_url = ""             # empty = provider default; set to override
model = ""                # empty = provider default model
temperature = 1.3         # 1.1-1.5 range (1.3 recommended)
http_referer = ""         # OpenRouter optional attribution
app_title = ""            # OpenRouter optional attribution

[pipeline]
model = "deepseek-chat"   # Fallback for DeepSeek only when [llm].model is empty
temperature = 1.3
intermediate_lang = "fi"
step4_engine = "google"   # "google" (default) | "niutrans"
```

### Step 4 engine (`pipeline.step4_engine`)

| Value | Engine | API key |
|-------|--------|---------|
| `google` (default) | Google Translate | None (free public API) |
| `niutrans` | Niutrans | `niutrans_api_key` |

With `google`, Steps 3 and 4 both use Google Translate. The distant-language hops (JA→FI→EN) still restructure text. With `niutrans`, Step 4 uses a different NMT architecture for stronger cross-engine fingerprint disruption.

**Model resolution order** (highest precedence first):

1. `llm_model=` argument to `run_standard_pipeline()`, or `model=` to `resolve_llm_config()`
2. `LLM_MODEL` environment variable
3. `[llm].model` in `config.toml`
4. `[pipeline].model` — **DeepSeek only** (ignored for OpenRouter so you don't accidentally send `deepseek-chat` instead of an OpenRouter slug)
5. Provider default (`deepseek-chat` or `deepseek/deepseek-chat`)

## Environment Variable Overrides

Optional runtime overrides (take precedence over TOML):

| Variable | Purpose |
|----------|---------|
| `LLM_PROVIDER` | `deepseek` or `openrouter` |
| `LLM_BASE_URL` | Override API base URL |
| `LLM_API_KEY` | Generic API key override |
| `OPENROUTER_API_KEY` | OpenRouter key when provider is `openrouter` |
| `DEEPSEEK_API_KEY` | DeepSeek key when provider is `deepseek` |
| `LLM_MODEL` | Override model slug |

**Example — switch to OpenRouter via environment (no TOML edit):**

```bash
export LLM_PROVIDER=openrouter
export OPENROUTER_API_KEY=sk-or-...
export LLM_MODEL=deepseek/deepseek-chat
python -m src.standard.pipeline --input "Your text here"
```

## Supported Target Languages

| Code | Language |
|------|----------|
| en | English |
| zh | Chinese |
| ja | Japanese |
| ko | Korean |
| fr | French |
| de | German |
| es | Spanish |
| pt | Portuguese |
| ru | Russian |
| ar | Arabic |
| it | Italian |
| nl | Dutch |
