---
layout: post
date: 2025-06-08
title: How to build an MCP server and research agent
giscus_comments: true
related_posts: false
tags: 
  - llm
  - mcp
---

Inspired by the Huggingface MCP [course](https://huggingface.co/learn/mcp-course/unit1/introduction), I decided to build an MCP application. 

I started by replicating an example from the course to ensure that it works. The MCP server implements a sentiment analysis function and the client is an agent that has access to the tool. Below is how I got it to run. 

Note this will run on the Inference Providers on HF. You get a $ 0.10 credits with a free account which is enough to run this agent a few times but you will soon get an error "You have exceeded your monthly included credits for Inference Providers. Subscribe to PRO to get 20x more monthly included credits." You can purchase more in your settings [page](https://huggingface.co/settings/billing/subscription). I had to subscribe for a month to experiment with this. In a future post I plan to look at running this locally.  

# Toy Example of MCP Server - Client

Set up a [standard repo](https://bayesways.github.io/blog/2025/How-I-set-up-my-project-repositories/) and install the following packages

```bash
pip install "smolagents[mcp]" "gradio[mcp]" mcp fastmcp textblob
```

Save a `.env` file
```
HUGGINGFACE_API_TOKEN=<yourHuggingFaceToken>
```
you can get one from your [settings](https://huggingface.co/settings/tokens) page.

Save the  `mcp_server.py`

```python
import gradio as gr
from textblob import TextBlob

def sentiment_analysis(text: str) -> dict:
    """
    Analyze the sentiment of the given text.

    Args:
        text (str): The text to analyze

    Returns:
        dict: A dictionary containing polarity, subjectivity, and assessment
    """
    blob = TextBlob(text)
    sentiment = blob.sentiment
    
    return {
        "polarity": round(sentiment.polarity, 2),  # -1 (negative) to 1 (positive)
        "subjectivity": round(sentiment.subjectivity, 2),  # 0 (objective) to 1 (subjective)
        "assessment": "positive" if sentiment.polarity > 0 else "negative" if sentiment.polarity < 0 else "neutral"
    }

# Create the Gradio interface
demo = gr.Interface(
    fn=sentiment_analysis,
    inputs=gr.Textbox(placeholder="Enter text to analyze..."),
    outputs=gr.JSON(),
    title="Text Sentiment Analysis",
    description="Analyze the sentiment of text using TextBlob"
)

# Launch the interface and MCP server
if __name__ == "__main__":
    demo.launch(mcp_server=True)
```

<s>Note that the app.py in the MCP course contains a small mistake as of today (6/11/25). The `sentiment_analysis` tool is returning a string that _looks_ like a dictionary (`"root={'polarity': 1.0, 'subjectivity': 1.0, 'assessment': 'positive'}"`) instead of returning an actual Python dictionary object.</s>
I opened a PR for this and it was accepted so this has now been corrected. 


Save the `mcp_client.py` 
```python
import gradio as gr
import os

from mcp import StdioServerParameters
from smolagents import InferenceClientModel, CodeAgent, ToolCollection, MCPClient


try:
    mcp_client = MCPClient(
        {"url": "http://localhost:7860/gradio_api/mcp/sse"} # This is the MCP Server we created in the previous section
    )
    tools = mcp_client.get_tools()

    model = InferenceClientModel(token=os.getenv("HUGGINGFACE_API_TOKEN"))
    agent = CodeAgent(tools=[*tools], model=model)

    demo = gr.ChatInterface(
        fn=lambda message, history: str(agent.run(message)),
        type="messages",
        examples=["Analyze the sentiment of the following text 'This is awesome'"],
        title="Agent with MCP Tools",
        description="This is a simple agent that uses MCP tools to answer questions.",
    )

    demo.launch()
finally:
    mcp_client.disconnect()
```

To run the agent first open a terminals and start the server
```bash
source venv/bin/activate
python mcp_server.py
```

Then open an another terminal and start the client
```bash
source venv/bin/activate
python mcp_client.py
```

Open the client from a browser at http://127.0.0.1:7861/ and ask it for a sentiment, e.g. Analyze the sentiment of the following text "This is awesome"

Then inspect the terminal running the client. It tells you how it's trying to use the tool. Interestingly the agent made an error but corrected itself and got the right answer. 
<div class="col-sm mt-3 mt-md-0">{% include figure.html path="assets/img/2025-06-15-How to build your own MCP server and research agent/Screenshot 2025-06-10 at 21.15.10.png" class="img-fluid rounded z-depth-1" %} </div>

Pretty cool.

# How does it work? 

Setting up the server is made easy with Gradio via the `mcp_server=True` flag on line 34 of `mcp_server.py`.
When you call `demo.launch(mcp_server=True)`, Gradio does several MCP-related things for you automatically:

 - Function Discovery: It scans your Gradio interface and finds the function we passed to `gr.Interface(fn=sentiment_analysis, ...)` on lines 24-25.
 - Tool Registration: It automatically registers this function as an MCP tool with:
    - The function name as the tool name
    - The function's docstring as the tool description
    - The function's parameters and return type as the tool schema
 - MCP Server Creation: It creates an MCP server that exposes these tools via the standard MCP protocol

Gradio makes it easy to turn a function into an MCP server if you know how to set it as a Gradio interface, which is simple. Plus you get a helpful UI in your browser to explore the MCP server (see the little option "Use via API or MCP" at the bottom of the Gradio UI and find the MCP tab).

Now, the client connects to the MCP endpoint (lines 9-11 in `mcp_client.py` ) and retrieves all available tools (line 12). This is made easy for us by the `smolagents` library that implements the specific `get_tools()` method we use here. 

If instead of the server we wrote we wanted to connect to another MCP server we would only need to change line 10 of the `mcp_client.py`. 

# Basic Research Agent 
Now it's time to build something a little more useful. While I was reading the course I found myself needing to search for information I had read but could not remember in which specific chapter. A perfect job for an agent. So let's see how we can build this. You can find the code [here](https://github.com/bayesways/my-mcp-app).

Drawing inspiration by this [repo](https://github.com/willccbb/research-agent-lesson) by Will Brown, I decided to implement the idea based on a simple fetching tool which takes in a url and iteratively explores the links in it to search for the answer to our question. For example if I wanted to know what packages we need to install for the Hugging Face course I can give the agent the url of the course and let it find the answer. 

The MCP server is a relatively simple function. The function does two things: extracts the text of the webpage into a clean markdown and collects all the links within the page. It provides the output in a structured format to be readable by the agent. The agent reads the output and has two options: answer our question based on the info so far or call the tool at its disposal with a new url. Doing this iteratively the agent can collect info on the websites that it has visited until it has enough context to answer. 

I asked it 
```
Go to https://huggingface.co/learn/mcp-course/unit1/introduction
and tell me what are the prerequisites for this course
```

and it answered correctly 
```
- Basic understanding of AI and LLM concepts
- Familiarity with software development principles and API concepts
- Experience with at least one programming language (Python or TypeScript examples will be shown)
```

The answer comes from a segment of [unit 0](https://huggingface.co/learn/mcp-course/unit0/introduction) which is different that the link I provided in my question. So the agent must have found its way to unit 0 to give the right answer. Interestingly, when I asked it again the same question it answered with "No specific prerequisites mentioned, but basic understanding of AI and programming would be helpful." In this case case the agent did not visit another url so it answered based on the info in unit 1 which is incorrect.

Other usecases would be looking for some specific information at a website, or looking to find a specific item from an online store. For example I asked to 
```
Give me a list of 10 art studios in Red Hook NY from this website
https://newyork.craigslist.org/search/hhh?query=art%20studio%20redhook#search=2~gallery~0
```
it returned a decent list. The full logs are available [here](https://raw.githubusercontent.com/bayesways/my-mcp-app/refs/heads/main/server_output_3.md).

# Q&A

## I tried `demo.launch(mcp_server=True)` but it doesn't work. 

For some reason I don't totally understand I had to explicitly set `export GRADIO_MCP_SERVER=True` in the terminal to get it to actually set up the MCP server when I run it for the first time. 


## How to define more than one tools per server? 

Note that argument `fn` on line 25 of `mcp_server.py` does not support a list. You can still expose multiple tools in one server though. You create one interface per tool and then combine them before launching as shown below. 

```python
tool_1_interface = gr.Interface(
    fn=sentiment_analysis,
    inputs=gr.Textbox(placeholder="Enter text to analyze..."),
    outputs=gr.JSON(),
    title="Text Sentiment Analysis",
    description="Analyze the sentiment of text using TextBlob"
)
tool_2_interface = gr.Interface(
    fn=text_summarizer,
    inputs=gr.Textbox(placeholder="Enter text to summarize..."),
    outputs=gr.JSON(),
    title="Text Summary Generator",
    description="Summarize a text"
)

# Combine all interfaces into a tabbed interface
demo = gr.TabbedInterface(
    [tool_1_interface, tool_2_interface],
    ["Sentiment Analysis", "Text Summarizer"]
)

# Launch with MCP server enabled
if __name__ == "__main__":
    demo.launch(mcp_server=True)
```


## What does `get_tools()` actually do? 

When we call `get_tools()`, here's what happens behind the scenes:
 - Client Request: The MCP client sends a `tools/list` message to the server
 - Server Response: The server responds with a list of available tools in this format:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "sentiment_analysis",
        "description": "Analyze the sentiment of the given text",
        "inputSchema": {
          "type": "object",
          "properties": {
            "text": {"type": "string", "description": "The text to analyze"}
          },
          "required": ["text"]
        }
      }
    ]
  }
}
```
  - Client Processing: The `MCPClient.get_tools()` method parses this JSON response and converts each tool definition into tool objects that your agent can use.

The MCPClient class (from `smolagents`) implements the MCP client specification, which includes:

 - Connection management: Connecting to MCP servers via different transports (HTTP SSE, stdio, etc.)
 - Protocol handling: Sending/receiving MCP messages in the correct JSON-RPC format
 - Tool discovery: The `get_tools()` method that sends `tools/list` requests
 - Tool execution: Methods to call tools via `tools/call` requests

 and these should work with any MCP server, not just the Gradio server we wrote. That's the point of MCP, to provide a standardized way to connect AI models to external tools and data sources. Which means: 
 - Standardization: Any MCP client can work with any MCP server
 - Discoverability: Clients can automatically discover what tools are available
 - Flexibility: Tools can be added/removed dynamically
 - Security: The protocol includes mechanisms for safe tool execution


## What's the significance of `sse`?

SSE stands for Server-Sent Events - it's the transport protocol that our MCP client is using to communicate with the Gradio MCP server in this example.

More generally MCP specifies how messages are transported between Clients and Servers. Two primary transport mechanisms are supported:
 - **stdio (Standard Input/Output)** used for local communication, where the Client and Server run on the same machine
 - **HTTP + SSE (Server-Sent Events) / Streamable HTTP** used for remote communication, where the Client and Server might be on different machines.

There are more protocols supported but exploring them further is a topic for another post. The MCP course on Hugging Face contains more info on this. 

SSE/Streamable HTTP is the standard choice for most AI/ML applications because:  

 - Long-running operations: AI models can take time, SSE allows progress updates
 - Streaming responses: Perfect for streaming text generation or real-time analysis
 - Resource monitoring: Server can notify clients when resources change
 - Connection persistence: Maintains connection for multiple interactions

One last thing, if you run the code you will get a warning 

_"specifying the 'transport' key is deprecated. For now, it defaults to the legacy 'sse' (HTTP+SSE) transport, but this default will change to 'streamable-http' in version 1.20. Please add the 'transport' key explicitly."_

The streamable-http transport will become the default in version 1.20 of the `mcp` library (as of today we are on 1.9) so to avoid the warning use
```python
mcp_client = MCPClient(
    {
        "url": "http://localhost:7860/gradio_api/mcp/sse", 
        "transport": "sse" # "streamable-http" is also fine
    }
)
```


## References

 - [Hugging Face MCP Course](https://huggingface.co/learn/mcp-course/unit0/introduction)
 - [How to Build an MCP Server in 5 Lines of Python](https://huggingface.co/blog/gradio-mcp)
 - Will Brown's research agent [repo](https://github.com/willccbb/research-agent-lesson)
 