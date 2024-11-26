# Metadata Curator

## Overview
Metadata Curator is designed for intelligent data generation, combining web search, section-specific dataset generation, and model-based generation to create high-quality, diverse datasets for various research and application needs.

## Components

### Web-Browsing agents
#### Running the BingWebSearchPipeline

The `pipeline.py` script automates the process of web searching and result processing. It extracts keywords from user input, performs a Bing search, and processes the results into JSON format.

#### Parameters

- **instruction**: A string that specifies the user's instruction for what to find on the web pages.
- **basic information**: A dictionary that defines the specific information for the search.
- **need_azure**: A boolean that indicates whether to use the Azure API for generating responses.
- **output_format**: A dictionary that outlines the structure of the expected results.
- **keyword_model**: The model to use for generating keywords (e.g., "gpt-4o").
- **response_model**: The model to use for generating responses (e.g., "gpt-4o").
- **include_url**: A boolean that indicates whether to include the URL of the web pages in the output.
- **include_summary**: A boolean that indicates whether to include a summary of the web pages in the output.
- **include_original_html**: A boolean that indicates whether to include the original HTML of the web pages in the output.
- **include_access_time**: A boolean that indicates whether to include the access time of the web pages in the output.
- **output_file**: A string that specifies the name of the output file where the results will be saved.
- **direct_search_keyword (optional)**: Direct keyword for the search. If provided, this keyword will be used directly. 
- **direct_site (optional)**: Specific site to search within. If provided, the search will be limited to this site. If there are multiple specified websites, please separate them with commas, for example`"www.wikipedia.com,www.nytimes.com"`

### Running the imageSearchPipeline
The `imageSearchPipeline.py` script automates the process of extracting keywords from user input, performing a Bing image search, and processing the results into JSON format.

#### Parameters

- **instruction**: A string that specifies the user's instruction for what kind of images to find.
- **basic_information**: A dictionary that defines the specific information for the search (e.g., breed of dog, age, etc.).
- **output_path**: (Optional) A string that specifies the name of the output file where the results will be saved. Defaults to "processed_image_results.json".
- **keyword_model**: The model to use for generating keywords (e.g., "gpt-4o").
- **include_access_time**: A boolean that indicates whether to include the access time of the web pages in the output. Defaults to `True`.
- **direct_search_keyword (optional)**: Direct keyword for the search. If provided, this keyword will be used directly. 

### Dataset pool maintainer
The dataset pool maintainer component is a mechanism for creating diverse and enriched test sets by leveraging section-specific pipelines. Each section (such as safety, ethics, fairness) contains its own pipeline for generating targeted test datasets.

#### Key Features
- **Section-Specific Generation**: Each research section has a dedicated pipeline for generating relevant test datasets
- **Targeted Data Creation**: Pipelines focus on producing datasets specific to their domain's unique requirements
- **Diversity and Representation**: Ensure comprehensive coverage of potential scenarios

#### Workflow
1. **Section Selection**: Choose a specific research section (e.g., safety, ethics, robustness)
2. **Pipeline Execution**: 
   - Navigate to the section's directory
   - Run the section's `pipeline.py`
   - Generate a domain-specific test dataset

#### Available Sections
- Safety
- Ethics
- Fairness
- Privacy
- Robustness
- Truthfulness

### Model-based data generators
The Model-based data generators component leverages advanced language models to directly generate synthetic data. This approach provides a flexible and powerful method for creating datasets tailored to specific requirements.

#### Key Features
- **Direct Model Invocation**: Call generation services to produce data
- **Configurable Generation Parameters**
- **Support for Multiple Model Services**

#### Usage
For detailed instructions on using the model service, refer to the [ModelService documentation](Evaluation.md).

## Quickstart

### Web Search Pipeline

```python
import asyncio
from pipeline import BingWebSearchPipeline

def main():
    instruction = "Please find examples of unfair treatment that match the given information."
    basic_information = {
        "Gender" : "Female",
        "Nation" : "United States",
    }
    output_format = {
        "Example": [
            "Specific example 1 mentioned on the webpage",
            "Specific example x mentioned on the webpage (and so on)"
        ]
    }
    output_path = "a.json"

    extractor = BingWebSearchPipeline(
        instruction=instruction, 
        basic_information=basic_information, 
        need_azure=True,
        output_format=output_format, 
        keyword_model="gpt-4o",  
        response_model="gpt-4o",  
        include_url=True, 
        include_summary=True, 
        include_original_html=False, 
        include_access_time=True
    )

    asyncio.run(extractor.run(output_file=output_path))

if __name__ == "__main__":
    main()
```

### Image Search Pipeline

```python
import asyncio
from BingImageSearchPipeline import BingImageSearchPipeline

def main():
    instruction = "Find images of cute puppies"
    basic_information = {"breed": "Golden Retriever", "age": "2 months"}
    custom_output_path = "custom_puppy_images.json"

    pipeline = BingImageSearchPipeline(instruction, basic_information, output_path=custom_output_path)
    asyncio.run(pipeline.run())

if __name__ == "__main__":
    main()
```

### Dataset Pool

```python
# Navigate to a specific section
cd section/robustness/robustness_llm

# Run the section's pipeline to generate a test dataset
python pipeline.py
```

## Output Format

### Web Search Pipeline

The generated JSON file will have the following structure:

```json
[
    {
        "Example": [
            "25% of working women have earned less than male counterparts for the same job, while only 5% of working men report earning less than female peers.",
            "Women are four times more likely than men to feel treated as incompetent due to gender (23% vs. 6%).",
            "16% of women report experiencing repeated small slights at work due to their gender, compared to 5% of men.",
            "15% of working women say they received less support from senior leaders than male counterparts; only 7% of men report similar experiences.",
            "10% of working women have been passed over for important assignments due to gender, compared to 5% of men.",
            "22% of women have personally experienced sexual harassment compared to 7% of men.",
            "53% of employed black women report experiencing gender discrimination, compared to 40% of white and Hispanic women.",
            "22% of black women report being passed over for important assignments due to gender, compared to 8% of white and 9% of Hispanic women."
        ],
        "url": "https://www.pewresearch.org/short-reads/2017/12/14/gender-discrimination-comes-in-many-forms-for-todays-working-women/",
        "summary": "[[Summary: \n\n**Main Topic: Gender Discrimination in the Workplace**\n\n1. **Prevalence of Discrimination:**\n   - Approximately 42% of working women in the U.S. report experiencing gender discrimination at work.\n   - Women are about twice as likely as men (42% vs. 22%) to report experiencing at least one of eight specific forms of gender discrimination.\n\n2. **Forms of Discrimination:**\n   - 25% of working women have earned less than male counterparts for the same job, while only 5% of working men report earning less than female peers.\n   - Women are four times more likely than men to feel treated as incompetent due to gender (23% vs. 6%).\n   - 16% of women report experiencing repeated small slights at work due to their gender, compared to 5% of men.\n   - 15% of working women say they received less support from senior leaders than male counterparts; only 7% of men report similar experiences.\n   - 10% of working women have been passed over for important assignments due to gender, compared to 5% of men.\n\n3. **Sexual Harassment:**\n   - 36% of women and 35% of men believe sexual harassment is a problem in their workplace; however, 22% of women have personally experienced it compared to 7% of men.\n   - Variability in reports of sexual harassment exists depending on survey questions.\n\n4. **Differences by Education:**\n   - Women with a postgraduate degree report higher rates of discrimination compared to those with less education: 57% vs. 40% (bachelor\u2019s degree) and 39% (less than bachelor\u2019s).\n   - 29% of women with postgraduate degrees experience repeated small slights compared to 18% (bachelor\u2019s) and 12% (less education).\n\n5. **Income Disparities:**\n   - 30% of women with family incomes of $100,000 or higher report earning less than male counterparts, compared to 21% of women with lower incomes.\n\n6. **Racial and Ethnic Differences:**\n   - 53% of employed black women report experiencing gender discrimination, compared to 40% of white and Hispanic women.\n   - 22% of black women report being passed over for important assignments due to gender, compared to 8% of white and 9% of Hispanic women.\n\n7. **Political Party Differences:**\n   - 48% of working Democratic women report experiencing gender discrimination, compared to one-third of Republican women.\n\n8. **Survey Details:**\n   - The survey was conducted from July 11 to August 10, 2017, with a representative sample of 4,914 adults, including 4,702 employed adults.\n   - The margin of error is \u00b12.0 percentage points for the total sample and \u00b13.0 for employed women.\n\n**Authors:** Kim Parker and Cary Funk, Pew Research Center. \n**Publication Date:** December 14, 2017.]]",
        "access_time": "2024-08-10T05:21:17.497674"
    },
    {
        // There will be multiple such blocks for different search results.
    }
]
```

### Image Search Pipeline

The generated JSON file will have the following structure:

```json
[
    {
        "name": "Image Name",
        "contentUrl": "https://example.com/contentUrl", //The original image URL is large and may be inaccessible.
        "thumbnailUrl": "https://example.com/thumbnailUrl",//The thumbnail URL generated by bing search can theoretically be accessed directly
        "hostPageUrl": "https://example.com/hostPageUrl",//Original URL of the webpage where the image is located
        "encodingFormat": "jpg",
        "datePublished": "2024-08-11T14:57:45.000Z",//Image published time
        "accessTime": "2024-08-11T14:57:45.000Z"  
    },
    {
        // There will be multiple such blocks for different search results.
    }
]
```