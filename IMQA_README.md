# IMQA LiteLLM Gateway

Production-ready LiteLLM proxy configuration for the IMQA platform and other products.

## Quick Start

### Local Development

```bash
# Install LiteLLM
pip install litellm

# Set environment variables (copy from .env.imqa)
export VERCEL_AI_GATEWAY_API_KEY=your-key
export OPENROUTER_API_KEY=your-key
export LITELLM_MASTER_KEY=sk-your-key

# Start the proxy
litellm --config imqa_config.yaml --port 4000
```

### Test the Proxy

```bash
# Health check
curl http://localhost:4000/health

# Test a completion
curl http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-master-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3-pro",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Available Models

### Tier 1: High-Capability (with fallbacks)

| Model Alias | Primary Provider | Fallbacks |
|-------------|------------------|-----------|
| `gemini-3-pro` | Vercel/Google | OpenRouter, GPT-4o |
| `gpt-4o` | Vercel/OpenAI | OpenRouter |
| `claude-4-sonnet` | Vercel/Anthropic | OpenRouter |

### Tier 2: Fast/Cheap (with fallbacks)

| Model Alias | Primary Provider | Fallbacks |
|-------------|------------------|-----------|
| `fast-model` | Vercel/Gemini Flash | GPT-4o-mini, OpenRouter |
| `gpt-4o-mini` | Vercel/OpenAI | OpenRouter |
| `gemini-2.5-flash` | Vercel/Google | OpenRouter |

### Tier 3: Reasoning

| Model Alias | Primary Provider | Fallbacks |
|-------------|------------------|-----------|
| `reasoning-model` | Vercel/Grok | OpenRouter, o1-mini |

### Direct Access (Wildcards)

Use provider prefix for any model:

```
google/gemini-2.5-pro       → Vercel AI Gateway
openai/gpt-4o               → Vercel AI Gateway
anthropic/claude-4-sonnet   → Vercel AI Gateway
xai/grok-4-fast             → Vercel AI Gateway
openrouter/any/model        → OpenRouter direct
replicate/owner/model       → Replicate direct
```

## Railway Deployment

### 1. Create Railway Project

```bash
# Connect your repo
railway link

# Or create new project
railway init
```

### 2. Add Services

- **Redis**: Right-click → Add Database → Redis
- **PostgreSQL** (optional): Right-click → Add Database → PostgreSQL

### 3. Set Environment Variables

In Railway dashboard, set:

```
VERCEL_AI_GATEWAY_API_KEY=your-key
OPENROUTER_API_KEY=your-key
REPLICATE_API_KEY=your-key
LITELLM_MASTER_KEY=sk-your-secure-key

# Use Railway variable references
REDIS_HOST=${{Redis.REDISHOST}}
REDIS_PORT=${{Redis.REDISPORT}}
REDIS_PASSWORD=${{Redis.REDISPASSWORD}}
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

### 4. Configure Start Command

```
litellm --config imqa_config.yaml --port $PORT
```

### 5. Deploy

Push to your connected branch, or:

```bash
railway up
```

## Features

### Caching (Redis)

- Reduces API costs by caching repeated requests
- 1-hour TTL by default
- Semantic caching available (similar prompts)

### Rate Limiting

- Global: Configured via `router_settings`
- Per-key: Set via virtual keys in database

### Monitoring

- Prometheus metrics exposed at `/metrics`
- Optional: Langfuse integration for detailed traces

## Updating Configuration

Since deployed from git repo:

1. Edit `imqa_config.yaml`
2. Commit and push
3. Railway auto-deploys

No Docker rebuild needed!

## IMQA Integration

The IMQA apps already point to LiteLLM:

```env
LITELLM_API_URL=https://litellm.oppla.dev/v1
LITELLM_API_KEY=sk-litellm-xxx
LLM_MODEL=google/gemini-3-pro-preview
```

With the new config, you can use:
- `gemini-3-pro` (alias with fallbacks)
- `google/gemini-3-pro-preview` (direct via Vercel AI Gateway)
- Any wildcard route like `openai/gpt-4o`
