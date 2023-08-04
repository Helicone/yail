# DRAFT - V0 (08/02/2023)

# YAIL - Yet Another AI Logger

Introducing a universal logger to streamline all AI operations.

# Proposal

Our goal is to establish a single normalization schema and pattern to streamline queries to Large Language Models (LLMs) for simplified logging, monitoring, debugging, and capturing. As the AI landscape evolves with new models and providers, we commit to keeping this schema up-to-date and shared among all stakeholders.

# Philosophy

I think the way we beat this and make sure we don't get swallowed up by larger corporations is for all us startups work together to have a shared base.

# User Experience

Our priority lies in enhancing our users' experience. Here's a glimpse of a typical user interaction:

```python
from langchain import lc_loggers # Llama Index | Gaurdrails | AutoGPT
from helicone import HeliconeLogger # PromptLayer | HoneyHive | LangFuse

lc_loggers.push(HeliconeLogger("api_key"))
```

## Problem

With a myriad of interfacing methods across AI apps, it's crucial to create a unified standard for logging completions of all inference calls, promoting standardization and ease of use.

## Basics

At its core, YAIL aims to capture inputs and outputs efficiently, pushing this crucial data to a logger for further analysis.

## Schema V0 Proposal

We leverage Python for our proposed schema, but the design patterns used can be found in most programming languages, making this proposal widely applicable. Our focus is on simplicity, and we'll build mappers/decorators as needed to fit the data.

Main interface:

```python
def log_request(request: YAILRequest) -> YAILRequestResult:
    pass

def log_response(response: YAILResponse) -> YAILResponseResult:
    pass
```

Possible Request implementation:

```python
@abc
class YAILInputs:
    def body() -> dict:
        raise UnimplementedError()

    def meta() -> dict:
        raise UnimplementedError()

class OpenAIInputs(YAILInputs):
    pass # Implementation for OpenAI inputs pending

@dataclass
class YAILRequest:
    def __init__(inputs: YAILInputs, LoggerVistor):
        pass
```

Potential user experience for base packages:

```python
from yail import openai # Helicone or PromptLayer's implementation can be utilized
from yail import loggers

loggers.push(HeliconeLogger) # Or any other logger
```

## Complexities

- **Streaming:** For streaming requests with varying implementations, we aim to normalize the body into a unified string, accurately representing it during the `log_response` call. We also prioritize logging partially received data in the event of stream disconnection.

- **Large payloads:** As we handle large data volumes, with expectations of 1 Million + tokens by the end of the year, we recognize that many serverless providers restrict payloads larger than 4MBs. Thus, our loggers will be built to chunk data accordingly.

- **Webhooks for responses:** In cases where providers like Replicate send back responses within a webhook, we're developing tools to link requests and responses even when the `request_id` linking them is not within the same stack frame when capturing the response.

## Extensions with Visitors

To extend the schema and send custom metadata, each logger should be able to provide a visitor that can be attached to the logger.

The user experience could look like this:

```python
from yail import openai
openai.ChatCompletion.create(
    model=model,
    messages=chat_log,
    temperature=0.7,
    yail_visit= lambda a: pass # Append custom meta data sent to the logger
)
```

or for the LangChain...

```python
chat = ChatOpenAI(temperature=0, yail_visit= lambda a: pass)
```

## Why You Should Get Onboard

Participation in this project offers several benefits. You can contribute to the development of a standard that will shape the future of AI logging. This allows for better intercommunication between different platforms and streamlines the process of handling various AI models. By working together, we can keep up with the rapid changes in AI technology and maintain our competitiveness against larger corporations.

## Who We Should Get Onboard

We aim to bring together various stakeholders in the AI field, including loggers, providers, and libraries. Here are a few we have identified:

| Loggers | Helicone | PromptLayer | HoneyHive |
| ------- | -------- | ----------- | --------- |

| Providers | Replicate | Hugging Face |
| --------- | --------- | ------------ |

| Libraries | LangChain | LlamaIndex | GaurdRails |
| --------- | --------- | ---------- | ---------- |

We believe that these entities, with their unique strengths and capabilities, can contribute significantly to the development and success of YAIL. We look forward to fostering a dynamic and diverse community around this project.
