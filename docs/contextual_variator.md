# Contextual Variator

## Overview
Contextual Variator is designed to enhance text diversity through various methods. It supports operations for both fixed-format questions (multiple choice, open-ended, or true/false) and non-format-specific text.



## Operations

### enhance_diversity method

The `enhance_diversity` method parameters depend on the operations you specified when initializing ContextualVariator. If you included the fixed format operations `["transform_to_multiple_choice","transform_to_true_false","transform_to_open_ended"]`, you must specify both **current_format** and **answer** parameters. Otherwise, it will only use the general enhance methods `"paraphrase_sentence"` and `"modify_sentence_length"`.

- The `keep_original` parameter defaults to `True`. When `True`, there's an equal probability of keeping the original text unchanged (operation method will be "keep_original"). Set to `False` to disable this option.
- The `extra_instructions` parameter can provide additional guidance to the model. This is an optional string parameter.

**Parameters**

- **sentence** (str): The input sentence or query.
- **current_format** (str, optional): The current format of the query. Required if using fixed format operations.
- **answer** (str, optional): The ground truth answer. Required if using fixed format operations and you want the answer transformed.
- **keep_original** (bool, optional): Whether to keep the original text unchanged. Defaults to `True`.
- **extra_instructions** (str, optional): Additional instructions for the model.



### paraphrase_sentence method

The `paraphrase_sentence` method is designed to generate a new version of the input sentence while maintaining its original meaning. This method is particularly useful for diversifying the phrasing of a given sentence without altering its core content. The paraphrased sentence will be structurally different from the original, potentially using different words, sentence structures, or grammatical constructs, but it will convey the same meaning.

**Parameters**

- **sentence** (str): The input sentence to paraphrase.



### modify_sentence_length method

The `modify_sentence_length` method is used to adjust the length of the input sentence. This method can either lengthen or shorten the sentence, depending on the specified or randomly selected length modification type. Lengthening a sentence involves adding more detail but ensuring that the core meaning of the sentence remains intact, while shortening a sentence involves condensing the information into fewer words. 

**Parameteras**

- **sentence** (str): The input sentence to modify.
- **length_modification** (str, optional): The type of length modification. Can be "lengthen" or "shorten". Defaults to randomly selecting one.



### transform_question_format method

The `transform_question_format` method is used to convert a question from one format to another. This method supports transforming questions between multiple-choice, true/false, and open-ended formats. If a ground truth answer is provided, the method will ensure that the transformed question retains the correct answer. 

**Parameters**

- **current_format** (str): The current format of the question.
- **current_question** (str): The current question to transform.
- **answer** (str, optional): The ground truth answer. Required if you want the answer transformed.
- **target_format** (str, optional): The target format to transform to. If not provided, a random format will be selected.

Available current_format:

- "Multiple choice question"
- "True/False question"
- "Open ended question"



## Example Usage

### enhance_diversity method

```python
import json
from contextual_variator import ContextualVariator
variator = ContextualVariator()

async def main():
    # non_format query/sentence
    result = await variator.enhance_diversity("This is a test sentence.")
    print(json.dumps(result, indent=4))

    # offer 'current_format' and 'answer'
    result = await variator.enhance_diversity(
        "What is the capital of France?", 
        current_format="Open ended question",
        answer="Paris"
    )
    print(json.dumps(result, indent=4))

    # only offer 'current_format'
    result = await variator.enhance_diversity(
        "What is the meaning of life?", 
        current_format="Open ended question",
    )
    print(json.dumps(result, indent=4))

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

**Output format:**

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



### paraphrase_sentence method

```python
sentence = "Life is like a box of chocolates."
result = await variator.paraphrase_sentence(sentence)
print(result)
```

**Output format:**
```json
{
  "sentence": "Life resembles a box of chocolates; you never know what you're going to get."
}
```



### modify_sentence_length method


```python
sentence = "The quick brown fox jumps over the lazy dog."
result = await variator.modify_sentence_length(sentence)
print(result)

# Specify length modification
result = await variator.modify_sentence_length(sentence, "lengthen")
print(result)
```

**Output format:**
```json
{
  "sentence": "The swift and agile brown fox gracefully leaps over the indolent and sluggish canine.",
  "operation": "lengthen"
}
```



### transform_question_format method

```python
current_format = "Multiple choice question"
current_question = "What is the capital of France? a) Berlin b) Madrid c) Paris d) Rome"
answer = "c) Paris" # Optional, provide if ground truth exists

# Random format selection
result = await variator.transform_question_format(current_format, current_question=current_question, answer=answer)
print(result)

# Specify target format
result = await variator.transform_question_format(
    current_format, 
    target_format="True/False question", 
    current_question=current_question, 
    answer=answer
)
print(result)
```

**Output format:**
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



## Recommended Usage: Batch File Processing

The recommended way to use Contextual Variator is through `file_handle.py` for batch processing, which is the most efficient method for handling large datasets.

**Example Folder Structure**

```
your_dataset/
├── file_config.json
├── data_1.json
├── data_2.json
└── data_3.json
```

**Configuration File Example (file_config.json)**

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

**Input File Format Example (data_1.json)**

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

**Output File Format Example (data_1_enhanced.json)**

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

**Usage**

1. Prepare your dataset folder containing:
   - Configuration file `file_config.json`
   - One or more data files (in .json format)

2. Run the processing script:

```bash
python file_handle.py --dataset_folder path/to/your/dataset
```


**Multi-turn Dialogue Support**

For multi-turn dialogue data, `transformation_method` should be a 2D list where each sublist corresponds to transformation methods for one turn:

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
