# Responses

## LLM & LVM Response Generation

The `LLM & LVM Response Generation` module is designed to generate responses from text prompts using various models. It supports both local and API-based models, allowing for flexibility in deployment and execution. The module can handle different request types such as `LLM` and `LVM`, processing data accordingly.

## Running the Script

```bash
python lm_response.py
```

## Configuration
- **Request Type**: The script processes data based on different request types such as `llm` (language models) and `lvm` (vision models). You can specify the request type by modifying the `request_type` variable in the `main()` function.

- **Model Selection**: The script supports both local and API models. You can specify which models to use by modifying the `async_list` (for API models) and `sync_list` (for local models) in the `main()` function.

- **Prompt and Result Keys**: The script processes JSON files containing prompts and generates responses based on the specified keys. You can configure the prompt and result keys by modifying the `prompt_key` and `result_key` variables.

## Example Usage
### LLM Response Generation

```python
if __name__ == '__main__':
    
    request_type = ['llm',]
    data_folder = '/path/to/your/data_folder'
    
    async_list = [
        'gpt-4o', 'gpt-4o-mini', 'gpt-3.5-turbo',
        'claude-3.5-sonnet', 'claude-3-haiku',
        'gemini-1.5-pro', 'gemini-1.5-flash', 'gemma-2-27B',
        'llama-3.1-70B', 'llama-3.1-8B',
        'glm-4-plus', 'qwen-2.5-72B', 
        'mistral-8x7B', 'mistral-8x22B', 
        'yi-lightning', 'deepseek-chat'
    ]
    
    sync_list = []
    
    prompt_key = ['prompt',]
    result_key = ['responses',]
    image_key = ['image_urls']
    file_name_extension = '_responses_test'

    for request_type, prompt_key, result_key in zip(request_type, prompt_key, result_key):
        asyncio.get_event_loop().run_until_complete(
            process_datafolder(data_folder, request_type, async_list, sync_list, prompt_key, result_key, file_name_extension, image_key=image_key)
        )
    print("All files processed.")
```

### LVM Response Generation

```python
if __name__ == '__main__':
    
    request_type = ['lvm',]
    data_folder = '/path/to/your/data_folder'
    
    async_list = [
        'gpt-4o', 'gpt-4o-mini', 
        'claude-3.5-sonnet', 'claude-3-haiku', 
        'gemini-1.5-pro', 'gemini-1.5-flash', 
        'glm-4v-plus', 'qwen-2-vl-72B', 
        'llama-3.2-90B-V', 'llama-3.2-11B-V'
    ]
    
    sync_list = []
    
    prompt_key = ['enhanced_prompt',]
    result_key = ['responses',]
    image_key = ['image_urls']
    file_name_extension = '_responses'

    for request_type, prompt_key, result_key in zip(request_type, prompt_key, result_key):
        asyncio.get_event_loop().run_until_complete(
            process_datafolder(data_folder, request_type, async_list, sync_list, prompt_key, result_key, file_name_extension, image_key=image_key)
        )
    print("All files processed.")
```

## Notes

- Ensure that the directory structure for saving responses exists or is created by the script.
- The script supports both LLM and LVM request types, allowing for flexibility in handling text and image-based prompts.

## Functions Overview

### `ResponseProcessor`

This class handles the processing of both asynchronous and synchronous models. It manages the generation of responses based on the request type (`llm` or `lvm`).

- **`process_async_service`**: Handles asynchronous API-based model requests.
- **`process_sync_service`**: Handles synchronous local model requests.
- **`get_single_response`**: Processes a single prompt and generates responses from all specified models.
- **`get_responses`**: Processes multiple prompts concurrently, respecting the maximum number of concurrent tasks.

### `MultiProcessor`

This class manages the processing of multiple request types and models. It handles loading data, processing responses, and saving the results.

- **`process_all`**: Processes all data files and generates responses for each prompt.
- **`count_invalid_responses`**: Counts invalid responses (e.g., empty or `None` responses) for each model.

### `process_datafolder`

This function processes all files in a specified data folder, generating responses for each file based on the request type and models.

## Output Example

The output is saved in JSON format, with the following structure:

```json
[
    {
        "prompt": "Describe the safety features of this product.",
        "responses": {
            "gpt-4o": "This product has several safety features...",
            "claude-3-haiku": "The product ensures safety by...",
            "gemini-1.5-pro": "Safety is a key aspect of this product..."
        }
    },
    ...
]
```

## T2I Response Generation

The `t2i_response` module is designed to generate images from text prompts using various models. It supports both local and API-based image generation, allowing for flexibility in deployment and execution. The module can handle different aspects such as robustness, fairness, safety, privacy, and truthfulness, generating images accordingly.


### Running the Script

```bash
export HF_TOKEN='your_hugging_face_token'
python t2i_response.py
```

### Configuration

- **Aspect Selection**: The script processes data based on different aspects such as `robustness`, `fairness`, `safety`, `privacy`, and `truthfulness`. You can change the aspect by modifying the `aspect` variable in the `main()` function.

- **Model Selection**: The script supports both local and API models. You can specify which models to use by modifying the `local_model` and `api_model` lists in the `main()` function.

### Data Processing

The script processes JSON files containing descriptions and generates images based on the specified aspect. The paths to these JSON files are defined in the `aspect_dict` dictionary.

### Output

Generated images are saved to specified output paths, which are determined based on the aspect and model used. The output paths are stored in the JSON data for reference.

## Notes

- Ensure that the directory structure for saving images exists or is created by the script.
- The script uses a Hugging Face token for authentication when using API-based models. Make sure to set this token correctly in your environment.