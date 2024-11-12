# Diversity Enhancer

Diversity Enhancer 是一个 Python 类，旨在通过多种方式增强文本的多样性。它提供了对有固定format (选择题、开放性问答或者判断题) 以及没有固定fomat的一系列enhance的操作，下面来进行详细介绍。

## 使用方法

首先，导入 DiversityEnhancer 类：

```python
from diversity_enhancer import DiversityEnhancer
```

然后，创建一个 DiversityEnhancer 实例：

```python
enhancer = DiversityEnhancer()
```

DiversityEnhancer 初始化的时候需要接受一个可选参数，这个可选参数是一个list，包含了所有在你的query上支持的操作方式（你可以自己指定，也可以不指定），以此来限制对query的处理。如果没有指定的话，默认初始化为下面的列表（所有的操作方式），并会进行随机选择一种策略进行enhance。

```python
supported_operations = [
    "transform_expression",
    "paraphrase_sentence",
    "modify_sentence_length",
    "transform_to_multiple_choice",
    "transform_to_true_false",
    "transform_to_open_ended"
]
```
您也可以只让他支持部分的操作
```python
operations = [
    "transform_to_multiple_choice"
]
enhancer = DiversityEnhancer(operations)
```

### 0. 注意事项

#### 基本用法

- 固定format是指你的原始query/sentence是 `["multiple_choice", "true_false", "open_ended"]`的其中之一，此时你在使用enhance方法时必须提供原始query的current format 作为`current_format`的参数。`answer`参数是可选提供的，如果提供就会根据问题形式的变化给出新问题下的answer，如果未提供就不会返回`answer`键。
- 非固定format就只用提供原始query/sentence即可

**固定format支持的操作**

```python
format_operations = [
    "transform_expression",
    "paraphrase_sentence",
    "modify_sentence_length",
    "transform_to_multiple_choice",
    "transform_to_true_false",
    "transform_to_open_ended"
]
```

**非固定format支持的操作**

```python
non_format_operations = [
    "transform_expression",
    "paraphrase_sentence",
    "modify_sentence_length"
]
```

使用 `enhance_diversity` 方法：

enhance方法针对你在实例化DiversityEnhancer时设置的方式来进行参数传递，如果你包含了固定format专属的三种操作`[transform_to_multiple_choice","transform_to_true_false","transform_to_open_ended"]`，你必须指定**current_format**以及**answer**两个参数。否则只会调用`"transform_expression", "paraphrase_sentence", "modify_sentence_length"`三种通用enhance方法。
- 同时我们可以传入一个keep_original参数，默认为`True`，如果为`True`的话则有与其他操作等概率出现一个操作方法为keep_original的操作，输出的结果与原来保持不变。为`False`则关闭这个操作。

`e.g`

```python
import json
from diversity_enhancer import DiversityEnhancer
enhancer = DiversityEnhancer()
async def main():

    # non_format query.sentence
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

`output`

```json
{
    "sentence": "I can't believe this is just a test sentence!",
    "structure_type": "emotion",
    "enhancement_method": "transform_expression"
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

#### 参数说明

- `sentence`: 要增强多样性的句子或问题（字符串）
- `current_format`: 当前问题的格式（可选，仅在处理固定format时需要）。对于问题格式转换，必须提供 `current_format` 参数
- `answer`: 可选，如果当前任务为一个问题，如果存在的话可以给 `answer`（ground truth），输出的时候也会将 ground truth 转化为对应的问题形式下的答案
- `extra_instructions` 可选，如果你想要给模型一些额外的指导，可以传入这个参数，这个参数是一个字符串，默认为空
#### 输出格式
所有输出的 JSON 结构与每个详细输出结构类似，可以参考下文1、2、3、4小节中的输出部分，只是会多出一个 `enhancement_method` 来说明随机选择了哪种方法来增强多样性。

### 1. 转换句子结构
`transform_expression` 函数可以将输入的句子转换为不同的结构或风格，同时保持原始含义。

#### 函数参数
- `sentence`: 要转换的原始句子 (必需)
- `structure_type`: 指定转换的结构类型 (必需)
- `example`: 用于 "example_type" 结构类型的示例句子 (可选)
- `custom_structures`: 自定义结构类型字典 (可选)

#### 使用方法
1. LLM自动选择模式(默认模式,推荐) 
使用 "select" 模式,让模型从可用结构中选择最合适的类型。此时会随机将可用模型列表中的**一半**在**打乱顺序**后传给LLM,让它自行决定哪个修改方式更合适。
```python
result = await enhancer.transform_expression(sentence, structure_type="select")
```
2. 随机转换 
使用 "random" 模式,函数会从默认的9种句式类型中随机选择一种进行转换。
```python
result = await enhancer.transform_expression(sentence,structure_type="random")
```
3. 指定结构类型
通过 `structure_type` 参数指定特定的转换类型。
```python
result = await enhancer.transform_expression(sentence, structure_type="passive_voice")
```
4. 示例类型转换
使用 `example_type` 结构类型,并提供 `example` 参数。
```python
result = await enhancer.transform_expression(sentence, structure_type="example_type", example="As the saying goes, 'The early bird catches the worm.'")
```
5. 自定义结构列表
提供 `custom_structures` 参数来使用自定义的结构类型。
```python
custom_structures = {
    "metaphor": "Rewrite the sentence as a metaphor that conveys the same meaning.",
    "alliteration": "Rewrite the sentence using alliteration while preserving its meaning.",
    "personification": "Rewrite the sentence using personification to convey the same idea."
}
result = await enhancer.transform_expression(sentence,  structure_type="select",custom_structures=custom_structures)
```


默认结构类型:
- declarative
<!-- - rhetorical_question -->
- imperative
- conditional
- passive_voice
- active_voice
<!-- - double_negative -->
- emphasize
- emotion

#### 输出格式
函数返回一个包含转换后句子和结构类型的JSON对象:
```json
{
  "sentence": "转换后的句子",
  "structure_type": "使用的结构类型"
}
```
对于 "select" 模式,输出还会包含所选的结构类型:
```json
{
  "selected_structure": "模型选择的结构类型",
  "sentence": "转换后的句子",
  "structure_type": "select"
}
```
注意事项: 

- 使用 "example_type" 时必须提供 `example` 参数。
- 提供 `custom_structures` 时,随机选择将从这些自定义结构中进行,而非默认结构。
- "select" 模式会自动选择最合适的结构类型,并在输出中指明。
- 所有转换都会尽可能保持原始句子的核心含义。

### 2. 改写句子

```python
sentence = "Life is like a box of chocolates."
result = await enhancer.paraphrase_sentence(sentence)
print(result)
```

#### 输出格式：
```json
{
  "Sentence": "Life resembles a box of chocolates; you never know what you're going to get."
}
```

### 3. 修改句子长度

```python
sentence = "The quick brown fox jumps over the lazy dog."
result = await enhancer.modify_sentence_length(sentence)
print(result)

# 指定增加或减少长度
result = await enhancer.modify_sentence_length(sentence, "lengthen")
print(result)
```
可选的长度修改操作：
- "lengthen"
- "shorten"

#### 输出格式：
```json
{
  "Sentence": "The swift and agile brown fox gracefully leaps over the indolent and sluggish canine.",
  "operation": "lengthen"
}
```

### 4. 转换问题格式

```python
current_format = "Multiple choice question"
current_question = "What is the capital of France? a) Berlin b) Madrid c) Paris d) Rome"
answer="c) Paris" #可选，如果存在ground truth则可以传入，没有可以省略。

# 随机选择新的问题格式
result = await enhancer.transform_question_format(current_format, current_question=current_question, answer=answer)
print(result)

# 指定特定的问题格式
result = await enhancer.transform_question_format(current_format, target_format="True/False question", current_question=current_question, answer=answer)
print(result)
```

可选的问题格式：
- "Multiple choice question"
- "True/False question"
- "Open ended question"

#### 输出格式：
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
当你选择随机选择格式的时候，随机的问题格式会是非当前格式的另外两种形式之一。
如果你没有输入 ground truth，则输出也不会包含 `answer`。

## 注意事项

- 所有方法都是异步的，需要在异步环境中使用（使用 `async/await`）。
- 当不指定特定类型时，方法会随机选择一个可用的转换类型。
