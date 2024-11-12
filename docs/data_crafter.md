# Data Crafter

## Components

### Web Search
#### Running the Unified Pipeline

The `pipeline.py` script automates the process of web searching and result processing. It extracts keywords from user input, performs a Bing search, and processes the results into JSON format.

#### Parameters

When running the unified pipeline, you need to provide several parameters to control its behavior:

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

#### Key Workflow Description in the UnifiedPipeline
##### User Input Construction
The first step involves constructing user input by combining the user's `instruction` and `basic information`. The basic information, represented as a dictionary, is transformed into a string format. This combined input is then used to generate search keywords through the `get_search_keyword` prompt template. After that, using the generated search keywords, a search is performed in Bing to retrieve relevant web pages.
**Example：**
Suppose we have the following input:
- `instruction`: "Please summarize the search results."
- `basic_information`: 
```python
    {
        "Topic": "Artificial Intelligence",
        "Year": "2024",
        "Region": "Global"
    }
```
The `basic_information` would be processed as:`"Topic is Artificial Intelligence. Year is 2024. Region is Global."`
The final `user_input` would be:`"Please summarize the search results. Topic is Artificial Intelligence. Year is 2024. Region is Global."`
Then we use this `user_input` to generate the search keyword

##### Summarizing Web Content
The search results are further processed using the `process_search_results` prompt template, which enables a LLM to create summaries for the content of each web page. This results in summaries that are comprehensible to the model, drawn from the original HTML text of each web page.

##### Generating Responses
With the user's instruction, basic information, output format, and the corresponding web page summaries, the `generate_jsonformat_prompt` prompt template is used to generate structured responses from the LLM. 
Here is our prompt template for generating structured responses:
```markdown
You are an AI assistant tasked with analyzing web content based on specific user requirements. Your job is to extract relevant information from a given web page summary according to provided themes and format the output as requested.

Input parameters:
1. Instruction: A partial instruction describing the task 
2. Basic Informations: A list of information or topics to focus on
3. Web Page Summary: A string containing a summary of the web page content
4. Output Format: A string containing a JSON format example and descriptions for each key

Your task:
1. Carefully read the instruction, basic information, and web page summary.
2. Identify and extract information from the web page summary that relates to the given information.
3. Organize the extracted information according to the provided output format.
4. Return a JSON object that follows the structure of the given output format, populated with the relevant information from the web page summary.

Please ensure that:
- You only include information that is directly related to the given information.
- You format the output exactly as specified in the output format example.
- If no relevant information is, include an "None" for that information in the JSON output.
- You do not add any additional commentary or explanation outside of the JSON structure.

Respond with the requested JSON object based on the provided inputs.

Here are the User Input:
Instruction: {instruction}
Basic Information: {basic_information}
Web Page Summary: {summary}
Output Format: {jsonformat}
```

##### Response Cleaning and Storage
The generated responses are then cleaned to ensure they are in valid JSON format before being stored in the `responses` attribute of the class.

##### Merging Responses
Finally, the various responses are merged together into a cohesive output format.

#### Example

Here's how to use the pipeline with an example script (already in `src/data_crafter/run.py`):

```python
import asyncio
from pipeline import UnifiedPipeline

def main():
    """
    Main function to run the UnifiedPipeline with specified parameters.
    """
    # Define the instruction and information for the pipeline
    instruction = "Please find examples of unfair treatment that match the given information."
    basic_information = {
        "Gender" : "Female",
        "Nation" : "United States",
    }

    # Define the user-specific dictionary for response formatting
    output_format = {
        "Example": [
            "Specific example 1 mentioned on the webpage",
            "Specific example x mentioned on the webpage (and so on)"
        ]
    }

    # Specify the output file path
    output_path = "a.json"

    # Initialize the UnifiedPipeline with the provided parameters and settings
    extractor = UnifiedPipeline(
        instruction=instruction, 
        basic_information=basic_information, 
        need_azure=True,
        output_format=output_format, 
        keyword_model="gpt-4o",  # Model for generating keywords
        response_model="gpt-4o",  # Model for generating responses
        include_url=True, 
        include_summary=True, 
        include_original_html=False, 
        include_access_time=True
    )

    # Run the pipeline and save the output to the specified file
    asyncio.run(extractor.run(output_file=output_path))

if __name__ == "__main__":
    main()
```

#### Output Structure

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

### Running the imageSearchPipeline
The `imageSearchPipeline.py` script automates the process of extracting keywords from user input, performing a Bing image search, and processing the results into JSON format.

#### Parameters

When running the pipeline, you need to provide several parameters to control its behavior:

- **instruction**: A string that specifies the user's instruction for what kind of images to find.
- **basic_information**: A dictionary that defines the specific information for the search (e.g., breed of dog, age, etc.).
- **output_path**: (Optional) A string that specifies the name of the output file where the results will be saved. Defaults to "processed_image_results.json".
- **keyword_model**: The model to use for generating keywords (e.g., "gpt-4o").
- **include_access_time**: A boolean that indicates whether to include the access time of the web pages in the output. Defaults to `True`.

- **direct_search_keyword (optional)**: Direct keyword for the search. If provided, this keyword will be used directly. 

#### Key Workflow Description in the imageSearchPipeline

##### Keyword Extraction
The first step involves constructing user input by combining the user's `instruction` and `basic_information`. This combined input is then used to generate search keywords through an OpenAI model.

##### Image Search
The script performs image searches on Bing using the extracted keywords and the Bing Search API.

##### Processing Search Results
The search results are processed to extract relevant details such as image name, content URL, thumbnail URL, host page URL, encoding format, and date published.

##### Saving to JSON
The processed results are saved to a specified JSON file.

#### Example

Here's how to use the `imageSearchPipeline.py` with an example script:

```python
import asyncio
from BingImageSearchPipeline import BingImageSearchPipeline

def main():
    """
    Main function to run the BingImageSearchPipeline with specified parameters.
    """
    # Define the instruction and basic information for the pipeline
    instruction = "Find images of cute puppies"
    basic_information = {"breed": "Golden Retriever", "age": "2 months"}

    # Use default output path
    pipeline = BingImageSearchPipeline(instruction, basic_information)
    asyncio.run(pipeline.run())

    # Use custom output path
    custom_output_path = "custom_puppy_images.json"
    pipeline_with_custom_output = BingImageSearchPipeline(instruction, basic_information, output_path=custom_output_path)
    asyncio.run(pipeline_with_custom_output.run())

if __name__ == "__main__":
    main()
```

#### Output Structure

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

### Dataset Pool
The Dataset Pool component is a mechanism for creating diverse and enriched test sets by leveraging section-specific pipelines. Each section (such as safety, ethics, fairness) contains its own pipeline for generating targeted test datasets.

#### Key Features
- **Section-Specific Generation**: Each research section has a dedicated pipeline for generating relevant test datasets
- **Targeted Data Creation**: Pipelines focus on producing datasets specific to their domain's unique requirements
- **Diversity and Representation**: Ensure comprehensive coverage of potential scenarios

#### Workflow
1. **Section Selection**: Choose a specific research section (e.g., safety, ethics, robustness)
2. **Pipeline Execution**: 
   - Navigate to the section's directory
   - Run the section's `pipeline.py` or `main.py`
   - Generate a domain-specific test dataset

#### Example Usage
```python
# Navigate to a specific section
cd section/robustness/robustness_llm

# Run the section's pipeline to generate a test dataset
python pipeline.py
```

#### Available Sections
- Safety
- Ethics
- Fairness
- Privacy
- Robustness
- Truthfulness

Each section's pipeline is designed to:
- Extract relevant keywords
- Generate diverse test scenarios
- Produce a JSON-formatted dataset
- Ensure high-quality, representative data

### Model-Based Generation
The Model-Based Generation component leverages advanced language models to directly generate synthetic data. This approach provides a flexible and powerful method for creating datasets tailored to specific requirements.

#### Key Features
- **Direct Model Invocation**: Call generation services to produce data
- **Configurable Generation Parameters**
- **Support for Multiple Model Services**

#### Usage
For detailed instructions on using the model service, refer to the [generation documentation](generation.md).

## Conclusion
Data Crafter provides a comprehensive, flexible framework for intelligent data generation, combining web search, section-specific dataset generation, and model-based generation to create high-quality, diverse datasets for various research and application needs.
