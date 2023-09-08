# Introduction
LLM-feedback(https://www.llmfeedback.com/) is a tool designed for LLM app developers to analyzing User-AI interaction the :

- Detect and summarize user queries that resulted in poor AI responses, potentially caused by issues with prompts, models, or RAG indexing.
- Monitor feedback on AI-generated content, staying informed about how users are impacted by changes in the LLM configuration, be it prompts, models, or RAG indexing.
- Track every modification to your LLM configuration and aggregate feedback for each version, enabling efficient LLM A/B testing.
- Gain deeper insights into user-AI interactions and the context surrounding specific feedback.


With the help of the client SDK, integration will be easy. Client SDK: https://github.com/scy0208/llm-feedback-client-py 

# Getting Started (Cloud Host Solution)
We will use Langchain team's example code chat-langchain(https://github.com/langchain-ai/chat-langchain) as an example. The detail code change can be found at (https://github.com/langchain-ai/chat-langchain/pull/129/files)

## Register and create project
Go to https://www.llmfeedback.com/register, connect your github account, and create a new project in the dashboard

## Install llm-feedback-client Python SDK
````mdx
pip install llm-feedback-client

````
## Create a client in your app:
```python showLineNumbers {3}
from llmfeedback.client import Client as FeedbackClient

project_id = "YOUR_PROJECT_ID"
api_key = "YOUR_API_KEY"
feedback_client = FeedbackClient(project_id=project_id, api_key=api_key)
```

## Register your LLM config:
```python showLineNumbers
# You can record anything you believe that is critical to the performance 
# and register them together as a version of config
CONFIG_NAME = "CHAT-LANGCHAIN_09072023"
feedback_client.register_config(
        config_name=CONFIG_NAME, 
        config={
            # record type 
            "type": "OpenAIFunctionsAgent",
            # record system_prompt
            "prompt": system_message.content,
            # record llm setting
            "llm": llm._default_params,
            # record tools (including the RAG setting)
            "tools": [{
                "name": tool.name,
                "description": tool.description,
            } for tool in tools]
            # record the RAG setting
            "rag_setting": {
                "chunk_size": 200
                "k": 3
            }
        }
    )
```

## Log the User-AI interation:
`id` can be used in link a feedback to a specific content.
`groupId` can help group together multile rounds of user-AI interations (e.g. a conversation).  
```python showLineNumbers {3}
feedback_client.log_dialogue(
    config_name=CONFIG_NAME,
    instruction=question,
    response=content,
    id=str(run_id),
    group_id=data.get("conversation_id"),
    created_by="user",
)
```

## Create feedback on specific content
```python showLineNumbers {3}
feedback_client.create_feedback(
    content_id=str(run_id), 
    key="user_score",
    score=score,
    user="user" 
)
```
In your UI component:
```html
<button type="button" onClick={() => createFeedback(run_id, 1)}
    <ThumbUpIcon />
</button>
```
