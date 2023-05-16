---
layout: post
date: 2023-05-15
title: How to get started with LLM App building and Langchain? Part 1 of n
giscus_comments: true
related_posts: false
---

I will use this space to document my effort to build LLM applications, starting pretty much from scratch. In order to have a concrete goal, I will aim to understand how to build an app that can answer questions and provide citations, utilizing a personal library of information. However, the lessons will apply equally to other use cases, some of which are listed in the references. 

## Background
It goes without saying that any LLM application is going to be based on **Langchain**. 
The [conceptual guide](https://docs.langchain.com/docs/)  is a good place to start. 

LLMs are game changing, partly because, they can simulate semantic understanding. This means that they can perform tasks that would typically require someone to understand the meaning of text, even if they don't really "understand" the text. Text is an example of an unstructured data format, as opposed to a structured dataset such as a database for example. Querying a database is simple because the data is stored in a way that's optimized for querying. Text on the other hand is words on a page without any pre-specified structure imposed on them. One can (correctly) object this point as human language is in fact at least partially structured, otherwise any random set of words would be meaningful. However for data retrieval purposes, a newspaper article is highly unstructured compared to a database. LLMs fill the gap and make retrieving information from unstructured text data possible, performing at the level we would expect someone with human language understanding to perform. 

The way LLM applications impose "structure" on text is via indexes (see the conceptual documentation [here](https://docs.langchain.com/docs/components/indexing/)). These are emerging specialized data structures to facilitate LLM application building. Currently the main type of index used is a *vector databases* which stores *embeddings*, a numerical vector associated with a chunk of text.


## Basic Elements of Langchain

### Models
Three kinds of models (quoting directly the documentation [page](https://docs.langchain.com/docs/components/models/))
- Large Language Models (LLMs) are the first type of models we cover. These models take a text string as input, and return a text string as output.
- Chat Models are the second type of models we cover. These models are usually backed by a language model, but their APIs are more structured. Specifically, these models take a list of Chat Messages as input, and return a Chat Message.
- Text Embedding Models. These models take text as input and return a list of floats.

### Input/Output
Any LLM output can be post-processed to fit specific requirements. Langchain provides the specialized output parsers for this reason (see [here](https://docs.langchain.com/docs/components/prompts/output-parser))


### Indexes
The database structure needed to facilitate LLM building. As we mentioned above, LLM applications building requires us to put some structure around text and store it, aka *index* text data. To do that Langchain provides specialized tools and integrates with the providers of such vector databases, such as  `pinecone` and `chroma`. Note that to create a text embedding, typically we need to use a pre-trained LLM. 


### Memory
Referes to the memory used by a chat agent. It's important to distinguish between short and long memory. *Short memory* captures the context of a singular conversation with an agent, whereas *long memory* refers to the rest of information that persists between conversations. 
The main concrete example of memory usage is for storing the context of a conversation between a user and a chatbot, and expanding that context as the conversation evolves. Langchain provides tools to handle this scenario. 


### Chains

Chain is the name given to an object that captures smaller components pieced together. 
The main type of a chain is an **LLM-Chain** which typically contains three components: 
 - an LLM model (such as [GPT3](https://platform.openai.com/docs/models/gpt-3) or [Flan](https://ai.googleblog.com/2021/10/introducing-flan-more-generalizable.html))
 - a prompting template
 - an output parser
 Each component might be broken down into further specialized pieces to achieve the desired goal of the application.  

The other common type of chain is an **Index-related chain** which uses the power of an LLM to interact with a specific text that the user chooses. The simplest way to pass the text to the LLM is to "stuff" it in the prompt of an LLM-Chain, hence defaulting back to the framework described above. However, that's not always the best choice and Langchain provides some additional methods for interacting with (indexed) text using LLMs. 

### Prompting

Prompting refers to the text input given to an LLM or chat agent. Although it might sound simple, it is a vital piece of LLM app building.  Langchain provides some frameworks for selecting the "right" prompt depending on the use case. The general task of prompting is more general, so we will have to come back to this.

### Agents

This refers to chains that do not have a pre-determined order of components. Instead, these chains consists of an "agent" and a pre-determined list of tools. The user interacts with the "agent" which has access to a suite of tools, and it's up to the "agent" to decide what tool to use when, depending on the user input. We will leave this for a next post. 


## References
- [Conceptual guide to langchain](https://docs.langchain.com/docs/)
- [List of LLM use cases](https://docs.langchain.com/docs/category/use-cases)