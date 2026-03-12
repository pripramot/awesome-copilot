---
name: ai-provider-management
description: 'Manage multiple AI providers (Gemini, OpenAI, Claude) with automatic fallback, load balancing, cost optimization, and unified API interfaces. Build resilient AI applications that gracefully handle provider outages and rate limits.'
---

# AI Provider Management (จัดการ AI หลายตัว)

Build resilient AI applications that seamlessly switch between providers with automatic failover and cost optimization.

## Provider Configuration

```python
from dataclasses import dataclass
from typing import Optional
import os

@dataclass
class ProviderConfig:
    name: str
    api_key: str
    model: str
    priority: int  # Lower = higher priority
    cost_per_1k_tokens: float
    max_tokens: int = 4096
    enabled: bool = True

# Configure your providers
PROVIDERS = [
    ProviderConfig(
        name="gemini",
        api_key=os.getenv("GEMINI_API_KEY", ""),
        model="gemini-1.5-flash",
        priority=1,
        cost_per_1k_tokens=0.0,  # Free tier
    ),
    ProviderConfig(
        name="openai",
        api_key=os.getenv("OPENAI_API_KEY", ""),
        model="gpt-4o-mini",
        priority=2,
        cost_per_1k_tokens=0.00015,
    ),
    ProviderConfig(
        name="claude",
        api_key=os.getenv("ANTHROPIC_API_KEY", ""),
        model="claude-haiku-3-5",
        priority=3,
        cost_per_1k_tokens=0.00025,
    ),
]
```

## Auto-Fallback Manager

```python
import asyncio
import logging
from typing import List

logger = logging.getLogger(__name__)

class AIProviderManager:
    def __init__(self, providers: List[ProviderConfig]):
        self.providers = sorted(providers, key=lambda p: p.priority)
        self.health_status = {p.name: True for p in providers}
        self.error_counts = {p.name: 0 for p in providers}
        self.MAX_ERRORS = 3

    async def generate(self, prompt: str, **kwargs) -> dict:
        """Try providers in priority order with automatic fallback."""
        errors = []

        for provider in self.providers:
            if not provider.enabled or not self.health_status[provider.name]:
                continue

            try:
                logger.info(f"Trying provider: {provider.name}")
                response = await self._call_provider(provider, prompt, **kwargs)
                # Reset error count on success
                self.error_counts[provider.name] = 0
                return {
                    "text": response,
                    "provider": provider.name,
                    "model": provider.model
                }
            except Exception as e:
                self.error_counts[provider.name] += 1
                if self.error_counts[provider.name] >= self.MAX_ERRORS:
                    self.health_status[provider.name] = False
                    logger.warning(f"Provider {provider.name} marked unhealthy")
                errors.append(f"{provider.name}: {str(e)}")

        raise RuntimeError(f"All providers failed: {'; '.join(errors)}")

    async def _call_provider(self, provider: ProviderConfig, prompt: str, **kwargs):
        if provider.name == "gemini":
            return await self._call_gemini(provider, prompt, **kwargs)
        elif provider.name == "openai":
            return await self._call_openai(provider, prompt, **kwargs)
        elif provider.name == "claude":
            return await self._call_claude(provider, prompt, **kwargs)
        raise ValueError(f"Unknown provider: {provider.name}")
```

## Provider Implementations

### Gemini Provider

```python
import google.generativeai as genai

async def _call_gemini(self, config: ProviderConfig, prompt: str, **kwargs) -> str:
    genai.configure(api_key=config.api_key)
    model = genai.GenerativeModel(config.model)
    response = model.generate_content(prompt)
    return response.text
```

### OpenAI Provider

```python
from openai import AsyncOpenAI

async def _call_openai(self, config: ProviderConfig, prompt: str, **kwargs) -> str:
    client = AsyncOpenAI(api_key=config.api_key)
    response = await client.chat.completions.create(
        model=config.model,
        messages=[{"role": "user", "content": prompt}],
        max_tokens=kwargs.get("max_tokens", config.max_tokens)
    )
    return response.choices[0].message.content
```

### Claude Provider

```python
import anthropic

async def _call_claude(self, config: ProviderConfig, prompt: str, **kwargs) -> str:
    client = anthropic.AsyncAnthropic(api_key=config.api_key)
    message = await client.messages.create(
        model=config.model,
        max_tokens=kwargs.get("max_tokens", config.max_tokens),
        messages=[{"role": "user", "content": prompt}]
    )
    return message.content[0].text
```

## Health Recovery

```python
async def recover_providers(self):
    """Periodically attempt to recover unhealthy providers."""
    for provider in self.providers:
        if not self.health_status[provider.name]:
            try:
                await self._call_provider(provider, "health check", max_tokens=10)
                self.health_status[provider.name] = True
                self.error_counts[provider.name] = 0
                logger.info(f"Provider {provider.name} recovered")
            except Exception:
                pass  # Still unhealthy
```

## Usage Example

```python
manager = AIProviderManager(PROVIDERS)

# Automatic fallback - tries Gemini first, then OpenAI, then Claude
result = await manager.generate("Explain quantum computing in Thai")
print(f"Response from {result['provider']}: {result['text']}")

# Schedule health checks
asyncio.create_task(recover_providers_periodically(manager))
```

## Best Practices

- Set `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` as environment variables
- Use the free Gemini tier as primary to minimize costs
- Monitor error rates and adjust `MAX_ERRORS` threshold based on your SLA
- Implement exponential backoff for rate-limited providers
- Cache responses where appropriate to reduce API calls
