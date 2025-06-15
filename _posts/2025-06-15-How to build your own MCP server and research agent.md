---
layout: post
date: 2025-06-08
title: How to build an MCP server
giscus_comments: true
related_posts: false
tags: 
---

Inspired by the Huggingface MCP [course](https://huggingface.co/learn/mcp-course/unit1/introduction), I decided to build an MCP application. 

I started by replicating an example from the course to ensure that it works. The MCP server implements a sentiment analysis function and the client is an agent that has access to the tool. Below is how I got it to run. 

Note this will run on the Inference Providers on HF. You get a $ 0.10 credits with a free account which is enough to run this agent a few times but you will soon get an error "You have exceeded your monthly included credits for Inference Providers. Subscribe to PRO to get 20x more monthly included credits." You can purchase more in your settings [page](https://huggingface.co/settings/billing/subscription). I had to subscribe for a month to experiment with this. In a future post I plan to look at running this locally.  

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

Now it's time to build something a little more useful. While I was reading the course I found myself needing to search for information I had read but could not remember in which specific chapter. A perfect job for an agent. So let's see how we can build this. 

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
it returned a decent list. The full logs are available [here](https://raw.githubusercontent.com/bayesways/my-mcp-app/refs/heads/main/server_output_3.md) 

## References

 - [Hugging Face MCP Course](https://huggingface.co/learn/mcp-course/unit0/introduction)
 - Will Brown's research agent [repo](https://github.com/willccbb/research-agent-lesson)