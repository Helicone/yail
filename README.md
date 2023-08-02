# DRAFT - V0 (08/02/2023)

# yail - Yet Another AI Logger

Let's create 1 logger to rule them all.

# Proposal

Let's define a single normalization schema and pattern to represent queries to LLMs so that they can be easily logged, monitored, debugged and captured. And let's all share it and keep it up to date as new models and providers come onto the market.

# Philosophy

I think the way we beat this and make sure we don't get swallowed up by larger corporations is for all us startups work together to have a shared base.

# User Experience

Typically it is best to understand how this will impact our users and make their lives better...

```python
from langchain import lc_loggers # Llama Index | Gaurdrails | AutoGPT
from helicone import HeliconeLogger # PromptLayer | HoneyHive | LangFuse


lc_loggers.push(HeliconeLogger("api_key"))
```

## Problem

There are a ton of different ways people are interfacing with these apps, let's generate a unified standard to log completions for all inference calls.

## Basics

At it's most fundamental level there is some input, and output that needs to be captured and pushed upwards to a logger.

## Schema V0 proposal

I am going to use python, but this schema will use design patterns found in most programming languages.

Let's keep the interface as simple as possible like this. Then let's build mappers/decorators and whatever else is needed to get the data to fit this.

Main interface:

```python
def log_request(request: YAILRequest) -> YAILRequestResult:
    pass

def log_response(response: YAILResponse) -> YAILResponseResult:
    pass
```

Possible Request implementation

```python
@abc
class YAILInputs:
    def body() -> dict:
        raise UnimplementedError()

    def meta() -> dict:
        raise UnimplementedError()

class OpenAIInputs(YAILInputs):
    pass # TODO implementation for OpenAI inputs

@dataclass
class YAILRequest:
    def __init__(inputs: YAILInputs, LoggerVistor):
        pass
```

Possible ux for base packages

```python
from yail import openai # we can copy Helicone or PromptLayer's implementation here
from yail import loggers

loggers.push(HeliconeLogger) # Or any other logger
```

## Where it get's complex

Streaming:
When requests are being streamed there is a lot of different implementations and the actual packets look different. Normalizing the body into one unified string and representing that correctly when `log_response` is called can be tricky. We also want to make sure we log any partially received data if the stream falls off.

Large payloads:
With Clause we are already getting 100k tokens and soon will have possibly 1 Million + tokens by the end of the year and we are talking about mega bytes of data. Many serverless providers restrict payloads larger than 4mbs, and we should be cautious about this and build our loggers in a way that we can chunk data

Webhooks for responses:
For replicate and other providers sometimes the response is sent back within a webhook. We should provide tooling and instrumentation to link the request and response in cases where the `request_id` that links the two is not within the same stack frame when the response is captured.

## Extensions with Vistors

For many loggers we want to be able to extend the schema and send our own meta data. We each logger should be able to provide a vistor that can be attached to the logger.

Possible ux can look like...

```python
from yail import openai
openai.ChatCompletion.create(
    model=model,
    messages=chat_log,
    temperature=0.7,
    yail_visit= lambda a: pass # Do something like append your own custom meta data sent to the logger
)
```

or for the LangChain...

```python
chat = ChatOpenAI(temperature=0, yail_visit= lambda a: pass)
```

## Why should we get onboard

## Who we should get onboard

| Loggers | Helicone | PromptLayer | HoneyHive |
| ------- | -------- | ----------- | --------- |

| Providers | Replicate | Hugging Face |
| --------- | --------- | ------------ |

| Libraries | LangChain | LlamaIndex | GaurdRails |
| --------- | --------- | ---------- | ---------- |
