# Diversity Enhancer

Diversity Enhancer is a Python toolkit designed to enhance text diversity through various methods. It supports operations for both fixed-format questions (multiple choice, open-ended, or true/false) and non-format-specific text.

## Recommended Usage: Batch File Processing

The recommended way to use Diversity Enhancer is through `file_handle.py` for batch processing, which is the most efficient method for handling large datasets.

### Example Folder Structure

```
your_dataset/
├── file_config.json
├── data_1.json
├── data_2.json
└── data_3.json
```

### Configuration File Example (file_config.json)

```json
[
    {
        "file_name": "data_1.json",
        "question_format": "open_ended",
        "transformation_method": [
            "paraphrase_sentence",
            "transform_to_multiple_choice"
        ]
    },
    {
        "file_name": "data_2.json",
        "question_format": "multiple_choice",
        "transformation_method": [
            "paraphrase_sentence",
            "modify_sentence_length",
            "transform_to_true_false"
        ]
    }
]
```

### Input File Format Example (data_1.json)

```json
[
    {
        "prompt": "What is the capital of France?",
        "ground_truth": "Paris",
        "extra_instructions": "Optional instructions for the model"
    },
    {
        "prompt": "Which planet is known as the Red Planet?",
        "ground_truth": "Mars"
    }
]
```

### Output File Format Example (data_1_enhanced.json)

```json
[
    {
        "prompt": "What is the capital of France?",
        "ground_truth": "Paris",
        "original_format": "open_ended",
        "enhanced_prompt": "What is the capital of France? A) Berlin B) Madrid C) Rome D) Paris",
        "enhanced_ground_truth": "D",
        "enhancement_method": "transform_to_multiple_choice",
        "format": "multiple_choice"
    },
    {
        "prompt": "Which planet is known as the Red Planet?",
        "ground_truth": "Mars",
        "original_format": "open_ended",
        "enhanced_prompt": "Mars is known as the Red Planet, answer true or false.",
        "enhanced_ground_truth": "True",
        "enhancement_method": "transform_to_true_false",
        "format": "true_false"
    }
]
```

### Usage

1. Prepare your dataset folder containing:
   - Configuration file `file_config.json`
   - One or more data files (in .json format)

2. Run the processing script:

```bash
python file_handle.py --dataset_folder path/to/your/dataset
```

### Multi-turn Dialogue Support

For multi-turn dialogue data, transformation_method should be a 2D list where each sublist corresponds to transformation methods for one turn:

```json
{
    "file_name": "dialogue_data.json",
    "question_format": "open_ended",
    "transformation_method": [
        ["paraphrase_sentence"],
        ["transform_to_multiple_choice"],
        ["modify_sentence_length"]
    ]
}
```

Multi-turn dialogue data format:
```json
[
    {
        "prompt": [
            "First turn question",
            "Second turn question",
            "Third turn question"
        ],
        "ground_truth": "The answer",
        "extra_instructions": "Optional instructions"
    }
]
```

## Individual Usage

If you only need to process single sentences or questions, you can also use the DiversityEnhancer class directly.

### Initialization

First, import the DiversityEnhancer class:

```python
from diversity_enhancer import DiversityEnhancer
```

Then, create a DiversityEnhancer instance:

```python
enhancer = DiversityEnhancer()
```

DiversityEnhancer initialization accepts an optional parameter - a list of supported operations for your queries. If not specified, it initializes with all available operations and randomly selects one strategy for enhancement.

```python
supported_operations = [
    "paraphrase_sentence",
    "modify_sentence_length",
    "transform_to_multiple_choice",
    "transform_to_true_false",
    "transform_to_open_ended"
]
```

You can also limit it to specific operations:
```python
operations = [
    "transform_to_multiple_choice"
]
enhancer = DiversityEnhancer(operations)
```

### Important Notes

#### Basic Usage

- Fixed format refers to queries/sentences that are one of `["multiple_choice", "true_false", "open_ended"]`. When using enhance methods with fixed formats, you must provide the original query's current format as the `current_format` parameter. The `answer` parameter is optional - if provided, it will return the answer transformed to match the new question format.
- Non-fixed format queries only require the original query/sentence.

**Operations Supported for Fixed Format**

```python
format_operations = [
    "paraphrase_sentence",
    "modify_sentence_length",
    "transform_to_multiple_choice",
    "transform_to_true_false",
    "transform_to_open_ended"
]
```

**Operations Supported for Non-Fixed Format**

```python
non_format_operations = [
    "paraphrase_sentence",
    "modify_sentence_length"
]
```

### enhance_diversity Method

The enhance method parameters depend on the operations you specified when initializing DiversityEnhancer. If you included the fixed format operations `["transform_to_multiple_choice","transform_to_true_false","transform_to_open_ended"]`, you must specify both **current_format** and **answer** parameters. Otherwise, it will only use the general enhance methods `"paraphrase_sentence"` and `"modify_sentence_length"`.

- The `keep_original` parameter defaults to `True`. When `True`, there's an equal probability of keeping the original text unchanged (operation method will be "keep_original"). Set to `False` to disable this option.
- The `extra_instructions` parameter can provide additional guidance to the model. This is an optional string parameter.

Example:

```python
import json
from diversity_enhancer import DiversityEnhancer
enhancer = DiversityEnhancer()

async def main():
    # non_format query/sentence
    result = await enhancer.enhance_diversity("This is a test sentence.")
    print(json.dumps(result, indent=4))

    # offer 'current_format' and 'answer'
    result = await enhancer.enhance_diversity(
        "What is the capital of France?", 
        current_format="Open ended question",
        answer="Paris"
    )
    print(json.dumps(result, indent=4))

    # only offer 'current_format'
    result = await enhancer.enhance_diversity(
        "What is the meaning of life?", 
        current_format="Open ended question",
    )
    print(json.dumps(result, indent=4))

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

Example output:

```json
{
    "sentence": "Let me rephrase this test sentence for you.",
    "enhancement_method": "paraphrase_sentence"
}
{
    "sentence": "What is the capital of France?\nA) Berlin\nB) Madrid\nC) Paris\nD) Rome",
    "answer": "Paris",
    "format": "Multiple choice question",
    "enhancement_method": "transform_to_multiple_choice"
}
{
    "sentence": "What is the meaning of life? a) Happiness b) Success c) 42 d) Love",
    "format": "Multiple choice question",
    "enhancement_method": "transform_to_multiple_choice"
}
```

## Specific Operations

### 1. Paraphrase Sentence

```python
sentence = "Life is like a box of chocolates."
result = await enhancer.paraphrase_sentence(sentence)
print(result)
```

Output format:
```json
{
  "sentence": "Life resembles a box of chocolates; you never know what you're going to get."
}
```

### 2. Modify Sentence Length

```python
sentence = "The quick brown fox jumps over the lazy dog."
result = await enhancer.modify_sentence_length(sentence)
print(result)

# Specify length modification
result = await enhancer.modify_sentence_length(sentence, "lengthen")
print(result)
```

Available length modification operations:
- "lengthen"
- "shorten"

Output format:
```json
{
  "sentence": "The swift and agile brown fox gracefully leaps over the indolent and sluggish canine.",
  "operation": "lengthen"
}
```

### 3. Transform Question Format

```python
current_format = "Multiple choice question"
current_question = "What is the capital of France? a) Berlin b) Madrid c) Paris d) Rome"
answer = "c) Paris" # Optional, provide if ground truth exists

# Random format selection
result = await enhancer.transform_question_format(current_format, current_question=current_question, answer=answer)
print(result)

# Specify target format
result = await enhancer.transform_question_format(
    current_format, 
    target_format="True/False question", 
    current_question=current_question, 
    answer=answer
)
print(result)
```

Available question formats:
- "Multiple choice question"
- "True/False question"
- "Open ended question"

Output format:
```json
{
    "sentence": "What is the capital of France?",
    "answer": "Paris.",
    "format": "Open ended question"
}

{
  "sentence": "What is the capital of France? a) Berlin b) Madrid c) Paris d) Rome",
  "format": "Multiple choice question",
  "answer": "Paris"
}

{
  "sentence": "Paris is the capital of France. True or False",
  "answer": "True",
  "format": "True/false question"
}
```

When randomly selecting formats, it will choose from the two formats that aren't the current format. If no ground truth is provided, the output won't include an `answer` field.