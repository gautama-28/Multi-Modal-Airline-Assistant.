# FlightAI Assistant

## Overview

FlightAI Assistant is a simple conversational airline chatbot built using:

- Python
- OpenAI API
- Function Calling (Tools)
- Gradio
- sqllite3

The primary goal of this project was to learn how Large Language Models interact with external functions and how tool calling works internally.

The assistant can:

- Answer airline-related queries
- Maintain conversation history
- Detect when a user is asking for ticket prices
- Call a Python function automatically
- Return the function result as a natural language response

---

# Learning Objectives

This project was built to understand:

1. How chat-based LLM applications are structured
2. How OpenAI Function Calling works
3. How conversation history is maintained
4. How external tools can be connected to an LLM
5. How Gradio can be used to build a quick user interface

---

# Project Architecture

```text
User
  |
  v
Gradio UI
  |
  v
chat()
  |
  v
OpenAI API
  |
  +------------------+
  |                  |
  | Tool Needed?     |
  |                  |
  +--------+---------+
           |
          Yes
           |
           v
handle_tool_call()
           |
           v
get_ticket_price()
           |
           v
Tool Response
           |
           v
OpenAI API
           |
           v
Final Answer
```

---

# Project Components

## 1. Environment Setup

```python
from dotenv import load_dotenv
from openai import OpenAI
import os
```

### Purpose

- Loads API keys securely
- Creates an OpenAI client
- Avoids hardcoding sensitive credentials

---

## 2. System Prompt

```python
system_message = "You are a helpful airline assistant"
```

### Purpose

The system message defines the assistant's behavior.

Think of it as permanent instructions given to the model before every conversation.

Example:

```text
You are a helpful airline assistant.
```

This influences tone, style, and behavior.

---

## 3. Ticket Price Function

```python
def get_ticket_price(destination_city):
```

### Purpose

This is a normal Python function.

The LLM cannot access databases or APIs directly.

Instead, it requests that this function be executed.

Example:

```python
get_ticket_price("Tokyo")
```

Returns:

```python
"₹132314"
```

---

## 4. Tool Definition

```python
tools = [...]
```

### Purpose

The model must know:

- Function name
- Parameters
- Parameter types
- Description

Without this definition, the model cannot call the function.

Think of it as a contract between the LLM and your Python code.

---

## 5. Building Conversation History

```python
history = [
    {
        "role": h["role"],
        "content": h["content"]
    }
    for h in history
]
```

### Purpose

Converts Gradio chat history into OpenAI message format.

Example:

```python
[
    {
        "role": "user",
        "content": "Hello"
    }
]
```

---

## 6. Building Messages

```python
messages = (
    [{"role": "system", "content": system_message}]
    + history
    + [{"role": "user", "content": message}]
)
```

### Purpose

Creates the complete conversation sent to the model.

Structure:

```python
[
    system message,
    previous messages,
    current user message
]
```

This allows the model to understand context.

---

## 7. First OpenAI Request

```python
response = openai.chat.completions.create(
    model=MODEL,
    messages=messages,
    tools=tools
)
```

### Purpose

Send:

- Conversation
- Available tools

to the model.

The model decides:

- Answer directly
- Call a tool

---

## 8. Detecting Tool Calls

```python
if response.choices[0].finish_reason == "tool_calls":
```

### Purpose

Checks whether the model wants to execute a function.

Possible outcomes:

```python
"stop"
```

Normal response.

or

```python
"tool_calls"
```

Function execution required.

---

## 9. Tool Execution

```python
response = handle_tool_call(message)
```

### Purpose

Runs the requested Python function.

Example:

Model requests:

```python
get_ticket_price("London")
```

Python executes:

```python
price = get_ticket_price("London")
```

Returns:

```python
"75513"
```

---

## 10. Parsing Arguments

```python
arguments = json.loads(
    tool_call.function.arguments
)
```

### Purpose

Converts tool arguments from JSON text into a Python dictionary.

Before:

```python
'{"destination_city":"London"}'
```

After:

```python
{
    "destination_city": "London"
}
```

---

## 11. Returning Tool Output

```python
response = {
    "role": "tool",
    "content": price_details,
    "tool_call_id": tool_call.id
}
```

### Purpose

Sends function results back to the model.

Example:

```python
{
    "role": "tool",
    "content": "75513"
}
```

---

## 12. Second OpenAI Request

```python
response = openai.chat.completions.create(
    model=MODEL,
    messages=messages
)
```

### Purpose

The model now sees:

1. User question
2. Tool request
3. Tool result

and generates the final natural language response.

Example:

```text
The ticket price for London is ₹75,513.
```

---

## 13. Gradio Interface

```python
gr.ChatInterface(
    fn=chat,
    type="messages"
)
```

### Purpose

Creates a browser-based chat application.

Workflow:

```text
User
 ↓
Gradio
 ↓
chat()
 ↓
OpenAI
 ↓
Response
 ↓
Gradio UI
```

---

# Example Tool Calling Flow

User:

```text
What is the ticket price to Tokyo?
```

Model:

```text
Call get_ticket_price("Tokyo")
```

Python:

```python
get_ticket_price("Tokyo")
```

Returns:

```text
132314
```

Model:

```text
The ticket price for Tokyo is ₹132,314.
```

---

# Key Concepts Learned

- OpenAI Chat Completions API
- Function Calling
- Tool Definitions
- JSON Parsing
- Conversation Memory
- System Prompts
- Gradio Interfaces
- LLM Orchestration
- API Key Management

---

# Future Improvements

- Real airline APIs
- Flight search
- Flight booking
- User authentication
- Flight status tracking
- Database integration
- Multi-tool support
- Streaming responses
- Error handling for invalid destinations

---

# Author

Shivam Gautam

Built as a learning project to understand OpenAI Function Calling and conversational AI application development.
