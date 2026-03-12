---
name: gemini-integration
description: 'Integrate Google Gemini AI models into applications with support for text generation, multimodal inputs (images, audio, video), function calling, streaming responses, and context caching. Includes Gemini CLI usage and Google AI Studio patterns.'
---

# Gemini Integration (ผสานรวม Google Gemini AI)

Integrate Google Gemini AI into your applications for text generation, multimodal understanding, and agentic capabilities.

## Setup

```bash
# Python
pip install google-generativeai

# Node.js
npm install @google/generative-ai

# Gemini CLI
npm install -g @google/gemini-cli
```

```python
import google.generativeai as genai
import os

genai.configure(api_key=os.environ["GEMINI_API_KEY"])
```

## Text Generation

```python
# Basic text generation
model = genai.GenerativeModel("gemini-1.5-flash")

response = model.generate_content("อธิบาย Quantum Computing ให้เข้าใจง่าย")
print(response.text)

# With system instruction
model = genai.GenerativeModel(
    model_name="gemini-1.5-flash",
    system_instruction="คุณคือผู้เชี่ยวชาญด้านเทคโนโลยี ตอบเป็นภาษาไทยที่เข้าใจง่าย"
)
response = model.generate_content("อธิบาย Docker ให้นักพัฒนามือใหม่")
print(response.text)
```

## Streaming Responses

```python
# Stream for real-time output
model = genai.GenerativeModel("gemini-1.5-flash")

for chunk in model.generate_content("เขียน essay เกี่ยวกับ AI", stream=True):
    print(chunk.text, end="", flush=True)
print()  # Newline after streaming
```

## Multimodal: Image + Text

```python
import PIL.Image

model = genai.GenerativeModel("gemini-1.5-flash")

# Analyze an image
image = PIL.Image.open("screenshot.png")
response = model.generate_content([
    "อธิบายสิ่งที่เห็นในภาพนี้เป็นภาษาไทย",
    image
])
print(response.text)

# Multiple images
images = [PIL.Image.open(f) for f in ["before.png", "after.png"]]
response = model.generate_content([
    "เปรียบเทียบสองภาพนี้ อะไรเปลี่ยนแปลงไปบ้าง?",
    *images
])
print(response.text)
```

## Multi-turn Chat

```python
model = genai.GenerativeModel("gemini-1.5-flash")
chat = model.start_chat(history=[])

# Maintain conversation context
while True:
    user_input = input("คุณ: ")
    if user_input.lower() in ["exit", "ออก"]:
        break

    response = chat.send_message(user_input)
    print(f"Nangnoy: {response.text}\n")
```

## Function Calling

```python
# Define tools for Gemini to use
tools = [
    genai.protos.Tool(
        function_declarations=[
            genai.protos.FunctionDeclaration(
                name="get_weather",
                description="Get current weather for a location",
                parameters=genai.protos.Schema(
                    type=genai.protos.Type.OBJECT,
                    properties={
                        "location": genai.protos.Schema(
                            type=genai.protos.Type.STRING,
                            description="City name, e.g. Bangkok, Thailand"
                        )
                    },
                    required=["location"]
                )
            )
        ]
    )
]

model = genai.GenerativeModel("gemini-1.5-flash", tools=tools)
response = model.generate_content("อากาศกรุงเทพฯ วันนี้เป็นอย่างไร?")

# Check if function was called
if response.candidates[0].content.parts[0].function_call:
    fc = response.candidates[0].content.parts[0].function_call
    print(f"Called: {fc.name}({dict(fc.args)})")
    # Execute the actual function and send result back
```

## Context Caching (for large documents)

```python
import google.generativeai as genai
from google.generativeai import caching
import datetime

# Cache large content for reuse
large_document = open("large_codebase.txt").read()

cache = caching.CachedContent.create(
    model="gemini-1.5-flash-001",
    display_name="My Codebase",
    system_instruction="You are an expert code reviewer.",
    contents=[large_document],
    ttl=datetime.timedelta(minutes=60),
)

# Use cached content
model = genai.GenerativeModel.from_cached_content(cached_content=cache)
response = model.generate_content("Review this code for security issues")
```

## Gemini CLI Usage

```bash
# Interactive chat
gemini chat

# One-shot query
gemini "อธิบาย Kubernetes ให้เข้าใจง่าย"

# Analyze file
gemini "Review this code" --file main.py

# Generate with specific model
gemini --model gemini-1.5-pro "Complex analysis task"

# Pipe input
cat error.log | gemini "What's causing this error?"
```

## Node.js Integration

```javascript
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

// Async generation
async function generateContent(prompt) {
  const result = await model.generateContent(prompt);
  return result.response.text();
}

// Streaming
async function streamContent(prompt) {
  const result = await model.generateContentStream(prompt);
  for await (const chunk of result.stream) {
    process.stdout.write(chunk.text());
  }
}
```

## Best Practices

1. **Model selection** - Use `gemini-1.5-flash` for speed/cost, `gemini-1.5-pro` for complex tasks
2. **Safety settings** - Configure appropriate safety thresholds for your use case
3. **Rate limits** - Free tier: 15 RPM, 1M TPM. Implement retry with exponential backoff
4. **Context window** - Gemini 1.5 Pro supports 2M tokens - use caching for large inputs
5. **Streaming** - Always stream for user-facing chat to improve perceived performance
6. **System instructions** - Set role and language instructions for consistent outputs
