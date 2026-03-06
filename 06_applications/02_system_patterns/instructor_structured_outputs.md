---
layer: 06_applications
type: application
status: seed
tags: [instructor, structured-outputs, pydantic, data-extraction, llm]
created: 2026-03-06
---

# Instructor: Structured LLM Outputs

## Purpose

Implementation patterns for using Instructor to extract validated, type-safe structured data from LLM responses. Instructor wraps any LLM client (OpenAI, Anthropic, Ollama, etc.) and uses Pydantic models as the return type schema — automatically retrying with the validation error as feedback when the model produces malformed output. It is the standard tool for building reliable data extraction, classification, and structured generation pipelines.

### Examples

- Entity extraction from unstructured text
- Document classification with confidence scores
- Structured reasoning outputs (chain-of-thought → typed answer)
- Multi-entity batch extraction pipelines

## Architecture

**Installation:**
```bash
pip install instructor pydantic
# Provider extras:
pip install "instructor[anthropic]"  # Claude
pip install "instructor[openai]"     # OpenAI
```

**Basic extraction pattern:**
```python
import instructor
from pydantic import BaseModel, Field
from anthropic import Anthropic

client = instructor.from_anthropic(Anthropic())

class UserProfile(BaseModel):
    name: str = Field(description="Full name")
    age: int  = Field(ge=0, le=120, description="Age in years")
    email: str = Field(description="Email address")

profile = client.messages.create(
    model="claude-3-5-haiku-latest",
    max_tokens=512,
    messages=[{
        "role": "user",
        "content": "Jane Doe is 28 years old, her email is jane@example.com"
    }],
    response_model=UserProfile
)
print(profile.name)   # "Jane Doe"
print(profile.age)    # 28
```

**With OpenAI:**
```python
from openai import OpenAI

client = instructor.from_openai(OpenAI())

result = client.chat.completions.create(
    model="gpt-4o-mini",
    response_model=UserProfile,
    messages=[{"role": "user", "content": "Extract: Bob Smith, 35, bob@co.com"}]
)
```

**Nested models and enums:**
```python
from enum import Enum
from typing import Optional

class Sentiment(str, Enum):
    POSITIVE  = "positive"
    NEGATIVE  = "negative"
    NEUTRAL   = "neutral"

class Address(BaseModel):
    city:    str
    country: str

class ReviewAnalysis(BaseModel):
    sentiment:       Sentiment
    confidence:      float = Field(ge=0.0, le=1.0)
    key_points:      list[str] = Field(description="Main points, max 5 items")
    address_mention: Optional[Address] = None

analysis = client.messages.create(
    model="claude-3-5-haiku-latest",
    max_tokens=512,
    messages=[{"role": "user", "content": "Analyze: 'Great product, shipped fast from London!'"}],
    response_model=ReviewAnalysis
)
```

**Custom validators with automatic retry:**
```python
from pydantic import field_validator
import re

class StructuredCode(BaseModel):
    language: str
    code:     str
    
    @field_validator("code")
    def no_placeholder_comments(cls, v):
        if "# TODO" in v or "pass" == v.strip():
            raise ValueError("Code must be complete, not a stub")
        return v

# Instructor sends the ValueError message back to the LLM and retries
snippet = client.messages.create(
    model="claude-3-5-haiku-latest",
    max_tokens=512,
    max_retries=3,
    messages=[{"role": "user", "content": "Write a Python hello world function"}],
    response_model=StructuredCode
)
```

**Streaming partial results:**
```python
from pydantic import BaseModel

class Report(BaseModel):
    title:    str
    sections: list[str]
    summary:  str

# Stream as fields populate — good for real-time UI updates
for partial in client.messages.create_partial(
    model="claude-3-5-haiku-latest",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a short report on transformers"}],
    response_model=Report
):
    print(f"\rTitle: {partial.title or '...'}", end="")
```

**Streaming iterable of items:**
```python
class Task(BaseModel):
    title:    str
    priority: str  # "high" | "medium" | "low"

# Returns items one by one as LLM generates them
for task in client.messages.create_iterable(
    model="claude-3-5-haiku-latest",
    max_tokens=512,
    messages=[{"role": "user", "content": "List 5 ML project tasks"}],
    response_model=Task
):
    print(f"- [{task.priority}] {task.title}")
```

**Batch extraction pipeline:**
```python
from concurrent.futures import ThreadPoolExecutor

def extract_one(text: str) -> UserProfile:
    return client.messages.create(
        model="claude-3-5-haiku-latest",
        max_tokens=256,
        messages=[{"role": "user", "content": text}],
        response_model=UserProfile
    )

texts = ["Alice, 30, alice@co.com", "Bob Smith 25 bob@co.com"]
with ThreadPoolExecutor(max_workers=4) as pool:
    profiles = list(pool.map(extract_one, texts))
```

**Using local models via Ollama:**
```python
from openai import OpenAI

client = instructor.from_openai(
    OpenAI(base_url="http://localhost:11434/v1", api_key="ollama"),
    mode=instructor.Mode.JSON
)

result = client.chat.completions.create(
    model="llama3.1",
    response_model=UserProfile,
    messages=[{"role": "user", "content": "Extract: ..."}]
)
```

**Key design guidelines:**
- Use `Field(description=...)` — the description is injected into the JSON schema the LLM sees
- Use `Field(ge=..., le=..., min_length=...)` to constrain values and reduce retries
- Keep models flat when possible; deeply nested models increase generation difficulty
- For long batches, set `max_retries=1` to fail fast and log failures separately

## Links
- [[structured_outputs|Structured Outputs]]
- [[prompting_strategies|Prompting Strategies]]
- [[function_calling|Function Calling]]
- [[model_serving_with_fastapi|Model Serving with FastAPI]]
