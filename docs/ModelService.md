# ModelService

The `ModelService` class is the central utility for managing various models, including **Large Language Models (LLMs)**, **Large Vision Models (LVMs)**, and **Text-to-Image (T2I)** models. It abstracts the complexity of model initialization, request handling, and response generation, providing a unified interface for both text and image-based tasks.

---

## Quick Start

The `ModelService` is included in the `trustgen` package. To get started:

```python
from trustgen.src.generation import ModelService

# Initialize ModelService
model_service = ModelService(
    request_type="t2i",
    handler_type="local",
    model_name="sd-3.5-large",
    config_path="/path/to/config.yaml"
)

# Process a single prompt
response = model_service.process("Generate an image of a futuristic city.")
```

---

## Initialization

To create an instance of the `ModelService` class, you need to specify the following parameters:

### Constructor Parameters

| Parameter       | Type    | Description                                                                 |
|------------------|---------|-----------------------------------------------------------------------------|
| `request_type`   | `str`   | The type of request: `"llm"` (Language Model), `"lvm"` (Vision Model), or `"t2i"` (Text-to-Image). |
| `handler_type`   | `str`   | The handler type: `"api"` (remote API-based inference) or `"local"` (local inference).  |
| `model_name`     | `str`   | The name of the model to use, mapped internally.                            |
| `config_path`    | `str`   | Path to the YAML configuration file.                                       |
| `**kwargs`       | `dict`  | Additional parameters to customize behavior (e.g., temperature).           |

---

## Supported Models

The `ModelService` supports a wide range of models for text, vision, and text-to-image tasks. Below is a comprehensive table of the supported models, categorized by request type (`llm`, `lvm`, `t2i`), model name, and whether the model uses `api` or `local` inference.

| Request Type | Model Name                                     | Handler Type |
|--------------|------------------------------------------------|--------------|
| **LLM**      | `gpt-4o`                                       | `api`        |
| **LLM**      | `gpt-4o-mini`                                  | `api`        |
| **LLM**      | `gpt-3.5-turbo`                                | `api`        |
| **LLM**      | `text-embedding-ada-002`                       | `api`        |
| **LLM**      | `glm-4`                                        | `api`        |
| **LLM**      | `glm-4-plus`                                   | `api`        |
| **LLM**      | `llama-3-8B`                                   | `api`        |
| **LLM**      | `llama-3.1-70B`                                | `api`        |
| **LLM**      | `llama-3.1-8B`                                 | `api`        |
| **LLM**      | `qwen-2.5-72B`                                 | `api`        |
| **LLM**      | `mistral-7B`                                   | `api`        |
| **LLM**      | `mistral-8x7B`                                 | `api`        |
| **LLM**      | `claude-3.5-sonnet`                            | `api`        |
| **LLM**      | `claude-3-haiku`                               | `api`        |
| **LLM**      | `gemini-1.5-pro`                               | `api`        |
| **LLM**      | `gemini-1.5-flash`                             | `api`        |
| **LLM**      | `command-r-plus`                               | `api`        |
| **LLM**      | `command-r`                                    | `api`        |
| **LLM**      | `gemma-2-27B`                                  | `api`        |
| **LLM**      | `deepseek-chat`                                | `api`        |
| **LLM**      | `yi-lightning`                                 | `api`        |
| **LVM**      | `glm-4v`                                       | `api`        |
| **LVM**      | `glm-4v-plus`                                  | `api`        |
| **LVM**      | `llama-3.2-90B-V`                              | `api`        |
| **LVM**      | `llama-3.2-11B-V`                              | `api`        |
| **LVM**      | `qwen-vl-max-0809`                             | `api`        |
| **LVM**      | `qwen-2-vl-72B`                                | `api`        |
| **LVM**      | `internLM-72B`                                 | `api`        |
| **LVM**      | `claude-3-haiku`                               | `api`        |
| **LVM**      | `gemini-1.5-pro`                               | `api`        |
| **LVM**      | `gemini-1.5-flash`                             | `api`        |
| **T2I**      | `dall-e-3`                                     | `api`        |
| **T2I**      | `flux-1.1-pro`                                 | `api`        |
| **T2I**      | `flux_schnell`                                 | `api`        |
| **T2I**      | `playgroundai/playground-v2.5-1024px-aesthetic`| `local`        |
| **T2I**      | `cogview-3-plus`                               | `api`        |
| **T2I**      | `sd-3.5-large`                                 | `local`      |
| **T2I**      | `sd-3.5-large-turbo`                           | `local`      |
| **T2I**      | `HunyuanDiT`                                   | `local`      |
| **T2I**      | `kolors`                                       | `local`      |
| **T2I**      | `playground-v2.5`                              | `local`      |

---

## Pipeline Initialization

The `_initialize_pipeline` method sets up the appropriate pipeline based on the provided parameters. It automatically configures the model, handler, and other runtime options.

### Example: Initialize a Stable Diffusion Pipeline for Local Use

```python
model_service = ModelService(
    request_type="t2i",
    handler_type="local",
    model_name="sd-3.5-large",
    config_path="/path/to/config.yaml"
)
```

---

## Methods

### 1. `process(prompt: Union[str, List[str]], **kwargs) -> Any`

Processes a single prompt or a list of prompts synchronously. It supports both one-off interactions and multi-turn conversations.

#### Parameters:
- `prompt` (`str` or `List[str]`): The input prompt(s).
- `**kwargs`: Additional parameters for model customization.

#### Returns:
- Model-generated responses.

#### Example:
```python
# Single prompt
response = model_service.process("Generate an image of a peaceful forest.")

# Multi-turn interaction
prompts = [
    "What is the capital of France?",
    "What is the population of Paris?"
]
responses = model_service.process(prompts)
```

---

### 2. `process_async(prompt: Union[str, List[str]], **kwargs) -> Awaitable[Any]`

Handles requests asynchronously, enabling high concurrency for demanding applications.

#### Parameters:
- `prompt` (`str` or `List[str]`): The input prompt(s).
- `**kwargs`: Additional model parameters.

#### Returns:
- An awaitable object with the model's response.

#### Example:
```python
# Asynchronous prompt
response = await model_service.process_async("Describe a sunset over the ocean.")
```

---

## Configuration

The configuration file (`config.yaml`) specifies model settings, resource allocation, and runtime options. Below is an example configuration format:

### Example Configuration (`config.yaml`)

```yaml
openai_sdk_llms:
  gpt-4o: OPENAI
  gpt-3.5-turbo: OPENAI
  glm-4: ZHIPU
  llama-3.1-70B: DEEPINFRA

local_models:
  sd-3.5-large: LOCAL
  HunyuanDiT: LOCAL

OPENAI:
  OPENAI_API_KEY: sk-XXXXXX
  OPENAI_BASE_URL: xxx
```

Replace the API keys and configuration paths with your own values.
