# Evaluation

## Overview

The `ModelService` class provides a flexible and powerful interface for interacting with various language models (LLM), vision models (LVM), and text-to-image models (T2I). It supports synchronous and asynchronous requests, as well as handling multi-turn conversations. This document outlines the usage, input formats, and output formats of the `ModelService` class.

## Features
- **Synchronous and Asynchronous Processing**: The class can handle both synchronous and asynchronous requests.
- **Multi-Turn Conversations**: The service supports multi-turn conversations, maintaining the context of the conversation history.
- **Customizable Request and Handler Types**: The class allows customization of request and handler types to suit various deployment scenarios.
- **Extensible for Future Implementations**: The class is designed to be extensible, with plans to include additional functionalities in future updates.

## Usage

### Initialization
Initialize the `ModelService` with the required parameters:
```python
from generation import ModelService

service = ModelService(
    request_type='llm',         # The type of request (e.g., 'llm' for language models)
    handler_type='api',         # The type of handler (e.g., 'api' for API-based requests)
    model_name='gpt-4o',        # The name of the model to use
    config_path='src/config/config.yaml',  # Path to the configuration file
    **kwargs                    # Additional keyword arguments
)
```

### Synchronous Processing

#### Single Prompt
To process a single prompt synchronously:
```python
response = service.process("What is the capital of France?")
print(response)
```

#### Multi-Turn Conversation
To process multiple prompts in a conversation synchronously:
```python
prompts = [
    "Hello, who are you?",
    "What can you do?",
    "Tell me a joke."
]
responses = service.process(prompts)
for response in responses:
    print(response)
```

### Asynchronous Processing

#### Single Prompt
To process a single prompt asynchronously:
```python
import asyncio

async def main():
    response = await service.process_async("What is the capital of France?")
    print(response)

asyncio.get_event_loop().run_until_complete(main())
```

#### Multi-Turn Conversation
To process multiple prompts in a conversation asynchronously:
```python
import asyncio

async def main():
    prompts = [
        "Hello, who are you?",
        "What can you do?",
        "Tell me a joke."
    ]
    responses = await service.process_async(prompts)
    for response in responses:
        print(response)

asyncio.get_event_loop().run_until_complete(main())
```

### Concurrent Processing
To apply a function concurrently to a list of elements with a maximum concurrency limit:
```python
import asyncio
import json
from generation import ModelService, apply_function_concurrently
from typing import List, Dict, Any

# process_element function is defined by you
async def process_element(element: Dict[str, Any]) -> Dict[str, Any]:
    text = element.get("prompt", "")
    result = await some_async_api_call(text)  # your customized asynchronous logic here
    element["result"] = result
    return element

async def main():
    with open('path_to_your_dataset.json', 'r', encoding='utf-8') as file:
        elements = json.load(file)
    results = await apply_function_concurrently(
        process_element, elements, max_concurrency=5
    )
    with open('path_to_generation_result.json', 'w') as file:
        json.dump(results, file, indent=4)

if __name__ == "__main__":
    asyncio.get_event_loop().run_until_complete(main())
```

### Specialized Model Usage

#### Text-to-Image (T2I) Models

The T2I models require additional keys such as `save_folder` and `file_name` for saving generated images. Here is an example:
```python
from generation import ModelService

service = ModelService(
    request_type="t2i",
    handler_type="api",
    model_name="flux_pro",
    config_path="src/config/config.yaml",
    # save_folder="./output", # the `save_folder` and `file_name` can either be set here or in the process method
    # file_name="output.jpg",
)

service.process("A little rabbit.", save_folder="./output", file_name="output.jpg")
```

#### Vision (LVM) Models

LVM models can handle multi-image inputs, requiring the `image_urls` keyword, which is a list of URLs for the images.

Example:
```python
from generation import ModelService

service = ModelService(
    request_type="lvm",
    handler_type="api",
    model_name="llama-3.2-90B-V",
    config_path="src/config/config.yaml",
)

print(service.process("What is the photo about?", image_urls=["output.png"]))
```

## Input and Output Formats
### Input
- **Single Prompt**: A single string.
- **Multi-Turn Conversation**: A list of strings, where each string is a user prompt.
- **T2I Specific Inputs**: Include `save_folder` and `file_name` for specifying where to save the generated images.
- **LVM Specific Inputs**: Include `image_urls`, a list of image URLs for multi-image processing.

### Output
- **Single Prompt**: A single response string.
- **Multi-Turn Conversation**: A list of response strings corresponding to each user prompt.
- **T2I Output**: None
- **LVM Output**: Response analysis based on the provided images.

## Supported Models
| Model Name             | LLM | LVM | T2I |
|------------------------|-----|-----|-----|
| gpt-4o                 | ✅  | ✅  |     |
| gpt-4o-mini            | ✅  | ✅  |     |
| text-embedding-ada-002 | ✅  |     |     |
| glm-4                  | ✅  |     |     |
| glm-4v                 |     | ✅  |     |
| claude-3.5-sonnet      | ✅  | ✅  |     |
| claude-3-opus          | ✅  |     |     |
| gemini-1.5-pro         | ✅  | ✅  |     |
| gemini-1.5-flash       | ✅  | ✅  |     |
| command-r-plus         | ✅  |     |     |
| command-r              | ✅  |     |     |
| llama-3-70B            | ✅  |     |     |
| llama-3.1-70B          | ✅  |     |     |
| llama-3.2-90B-V        |     | ✅  |     |
| qwen-2.5-72B           | ✅  |     |     |
| qwen-max-vl            |     | ✅  |     |
| mistral-8x22B          | ✅  |     |     |
| mistral-8x7B           | ✅  |     |     |
| deepseek-chat          | ✅  |     |     |
| dalle3                 |     |     | ✅  |
| flux-pro               |     |     | ✅  |
| playground-v2.5        |     |     | ✅  |
| cogView-3-plus         |     |     | ✅  |
| kolors                 |     |     | ✅  |
| HunyuanDiT             |     |     | ✅  |

Any additional questions or issues, please contact the coding team.