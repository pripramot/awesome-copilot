# AI Arsenal Switcher 🔄⚡

Never let AI provider outages stop your work. This plugin provides multi-provider AI management with automatic failover between Gemini, OpenAI, and Claude — keeping your AI-powered applications always running.

## Features

- **Multi-Provider Support** - Gemini (free), OpenAI GPT, and Anthropic Claude
- **Automatic Fallback** - Seamlessly switches providers on failure
- **Cost Optimization** - Prioritizes free tiers, uses paid providers as backup
- **Health Monitoring** - Tracks provider health and auto-recovers
- **Vertex AI Integration** - Enterprise-grade Google Cloud AI capabilities
- **Gemini CLI** - Direct Gemini AI integration with streaming support

## Included

| Resource | Type | Description |
|----------|------|-------------|
| `ai-provider-management` | Skill | Multi-provider manager with automatic fallback |
| `gemini-integration` | Skill | Google Gemini AI with streaming and multimodal support |
| `vertex-ai` | Skill | Enterprise ML on Google Cloud Vertex AI |

## Installation

```bash
gh copilot plugin install ai-arsenal-switcher
```

## Quick Start

### Provider Priority (default)

1. **Gemini Flash** — Free, fast, great for most tasks
2. **GPT-4o Mini** — High quality, cost-effective paid option
3. **Claude Haiku** — Excellent for analysis and writing

### Setup Environment Variables

```bash
export GEMINI_API_KEY="your-gemini-key"
export OPENAI_API_KEY="your-openai-key"      # Optional
export ANTHROPIC_API_KEY="your-claude-key"   # Optional
```

### Usage in Code

```python
from ai_arsenal import AIProviderManager, PROVIDERS

manager = AIProviderManager(PROVIDERS)
result = await manager.generate("Your prompt here")
print(f"Response from {result['provider']}: {result['text']}")
```

## Why AI Arsenal?

| Scenario | Without Arsenal | With Arsenal |
|----------|----------------|--------------|
| Gemini outage | ❌ App breaks | ✅ Auto-switches to GPT |
| Rate limit hit | ❌ Errors | ✅ Queues or switches |
| High costs | 💸 Always paid | 💚 Uses free tier first |

---

*Keep your AI apps running 24/7*
