# Improvement Records

Local modifications made during the RAGAS evaluation POC project:
https://github.com/Sheryl-shiyi/proj-poc-RAGAS

## 1. Switch LLM wrapper from Completion API to Chat Completion API

**Files changed:**
- `src/llama_stack_provider_ragas/compat.py`
- `src/llama_stack_provider_ragas/inline/wrappers_inline.py`

**What changed:**

The inline LLM wrapper (`LlamaStackInlineLLM`) was using the OpenAI-compatible **text completion** API (`/v1/completions` via `openai_completion()`). This was changed to use the **chat completion** API (`/v1/chat/completions` via `openai_chat_completion()`).

Specifically:
- `OpenAICompletionRequestWithExtraBody(prompt=...)` → `OpenAIChatCompletionRequestWithExtraBody(messages=[...])`
- `inference_api.openai_completion()` → `inference_api.openai_chat_completion()`
- Response parsing: `choice.text` → `choice.message.content`
- Added imports in `compat.py`: `OpenAIChatCompletionRequestWithExtraBody`, `OpenAIUserMessageParam`

**Why:**

When RAGAS evaluates answers using the judge LLM, it sends prompts through the wrapper to the vLLM backend. The original text completion API (`/v1/completions`) sends the raw prompt text directly to vLLM without applying the model's chat template. For instruct-tuned models (e.g., Gemma 3 27B, Gemma 4 31B IT), this means the model receives unformatted text instead of the structured `<start_of_turn>user\n...<end_of_turn>` format it was trained on, leading to lower quality and less reliable judge responses.

Switching to the chat completion API (`/v1/chat/completions`) causes vLLM to automatically apply the model's chat template, producing better structured outputs from the judge LLM during RAGAS evaluation.

**Impact:**

This change is required for correct RAGAS evaluation when using instruct-tuned models served by vLLM as the judge. Without it, the judge LLM may produce poorly formatted or lower-quality assessments.
