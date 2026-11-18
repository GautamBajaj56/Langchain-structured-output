# LangChain Structured Output

A small, example-driven project for learning how to make language models return
structured, predictable data instead of free-form text.

The examples use a product review as input and extract fields such as themes,
summary, sentiment, pros, cons, and reviewer name. They compare three common
schema styles:

- **Pydantic models** - runtime validation and convenient serialization.
- **`TypedDict`** - lightweight type hints for dictionary-shaped data.
- **A JSON Schema dictionary** - a provider-neutral schema representation.

The project also includes a local Pydantic example and a Hugging Face version
of the LangChain example.

## What you will learn

1. How Pydantic converts and validates input values.
2. The difference between type hints and runtime validation.
3. How `with_structured_output(...)` maps model responses to a schema.
4. How optional fields, lists, descriptions, and `Literal` values affect a
   structured response.
5. How to switch between an OpenAI chat model and a Hugging Face endpoint.

## Project structure

| File | Purpose |
| --- | --- |
| [`pydantic_demo.py`](./pydantic_demo.py) | Defines and validates a `Student` Pydantic model. |
| [`typeddict_demo.py`](./typeddict_demo.py) | Demonstrates a dictionary shape using `TypedDict`. |
| [`with_structured_output_pydantic.py`](./with_structured_output_pydantic.py) | Extracts review data into a validated Pydantic model using OpenAI. |
| [`with_structured_output_typeddict.py`](./with_structured_output_typeddict.py) | Extracts review data into a `TypedDict` using OpenAI. |
| [`with_structured_output_json.py`](./with_structured_output_json.py) | Extracts review data using an inline JSON Schema dictionary. |
| [`with_structured_output_llama.py`](./with_structured_output_llama.py) | Runs the Pydantic example through a Hugging Face TinyLlama endpoint. |
| [`json_schema.json`](./json_schema.json) | A simple JSON Schema learning artifact. |
| [`students_dataset.csv`](./students_dataset.csv) | Placeholder CSV for future student-data exercises. |

## Requirements

- Python 3.9 or newer
- An API key for the provider used by the example:
  - OpenAI examples: `OPENAI_API_KEY`
  - Hugging Face example: `HUGGINGFACEHUB_API_TOKEN`

The code uses:

- [Pydantic](https://docs.pydantic.dev/)
- [python-dotenv](https://pypi.org/project/python-dotenv/)
- [LangChain OpenAI](https://pypi.org/project/langchain-openai/)
- [LangChain Hugging Face](https://pypi.org/project/langchain-huggingface/)

## Setup

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell, activate it with:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install the dependencies:

```bash
pip install pydantic[email] python-dotenv langchain-openai langchain-huggingface
```

Create a `.env` file in the project root. Add only the key for the provider
you intend to use:

```dotenv
OPENAI_API_KEY=your_openai_api_key
HUGGINGFACEHUB_API_TOKEN=your_huggingface_token
```

Do not commit `.env` or expose API keys in source code.

## Running the examples

Run the local examples first; they do not call an LLM:

```bash
python pydantic_demo.py
python typeddict_demo.py
```

Then run any OpenAI structured-output example:

```bash
python with_structured_output_pydantic.py
python with_structured_output_typeddict.py
python with_structured_output_json.py
```

To try the Hugging Face version:

```bash
python with_structured_output_llama.py
```

The model and provider must support structured output. Hosted model behavior
can vary, especially for smaller models such as TinyLlama.

## Key concepts

### Pydantic validates at runtime

In [`pydantic_demo.py`](./pydantic_demo.py), Pydantic converts the string
`"32"` to an integer because the field is declared as `int`. It also validates
the email and enforces the CGPA range with `Field(gt=0, lt=10)`.

This is different from a type annotation alone: annotations document the
expected shape, but they do not automatically validate values at runtime.

### `TypedDict` describes a dictionary shape

`TypedDict` is useful when the result should remain a normal dictionary and
you mainly want editor and static type-checker support. It does not perform
runtime validation or conversion. For example, the `age` value in
[`typeddict_demo.py`](./typeddict_demo.py) is a string even though the
annotation says `int`.

### Structured output is schema-guided generation

`model.with_structured_output(Review)` gives the model a schema and asks
LangChain to parse the response into that shape. Pydantic is usually the
strongest option in this project because it combines schema generation,
validation, and a convenient Python object.

Descriptions in `Field(...)` and `Annotated[...]` are part of the schema
guidance. Make them specific: they help the model understand what belongs in
each field.

### `Literal` values must match the declared values

The examples declare sentiment as `Literal["pos", "neg"]`. That means
`"neutral"` is not a valid result, even though some field descriptions mention
neutral sentiment. If neutral sentiment is required, update the schema to:

```python
sentiment: Literal["pos", "neg", "neutral"]
```

This is an important lesson: the type definition is the actual contract; the
description should agree with it.

## Suggested learning exercises

- Add `"neutral"` to the sentiment values and test a neutral review.
- Add a numeric `rating` field constrained to a range such as 1 through 5.
- Change the review input into a reusable function parameter.
- Add error handling for validation and provider/API failures.
- Load rows from `students_dataset.csv` and validate them with a Pydantic
  model.
- Compare the returned Pydantic object with the dictionary returned by the
  `TypedDict` example.
- Move the repeated review text into a separate file or prompt template.

## Notes about the learning artifacts

- The examples intentionally repeat the same review so the schema approaches
  can be compared easily.
- [`json_schema.json`](./json_schema.json) is a starting point for learning
  JSON Schema. A standard JSON Schema uses property definitions such as
  `"type": "string"` rather than `"name": "string"`; improve the file before
  using it as a strict provider schema.
- The scripts execute immediately when run. They are teaching examples rather
  than a packaged application or production-ready extraction service.

## Troubleshooting

- **`OPENAI_API_KEY` is missing:** check that `.env` is in the project root and
  that `load_dotenv()` can find it.
- **Email validation fails:** install the email extra with
  `pip install "pydantic[email]"`.
- **A Hugging Face request fails:** verify the token, endpoint availability,
  model permissions, and whether the selected model supports the requested
  task.
- **The output does not match the schema:** try a model with stronger
  structured-output/tool-calling support and make the field descriptions more
  explicit.

## License

This is a personal learning project. Add a license if you plan to distribute
or reuse it publicly.
