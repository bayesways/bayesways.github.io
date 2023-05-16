---
layout: post
date: 2023-05-16
title: How to get started with LLM App building and Langchain? Part 2 of n
giscus_comments: true
related_posts: false
tags: 
---

# Overview 

In [Part 1 of n](https://bayesways.github.io/blog/2023/How-to-get-started-with-LLM-App-building-and-Langchain-Part-1-of-n/) we got a general overview of the Langchain framework for building LLM apps. We are now going deeper into some of the issues we will face as we start building our app. As a reminder, our goal is to build an app that can answer questions on a personal library of information, and provide citations for its answers. The fancy term for this is *retrieval augmented generation*. The high level structure of a chain appropriate for this task is described in [this](https://docs.langchain.com/docs/use-cases/qa-docs) use case from the Langchain documentation. I am copying it here directly from the docs: 

**Ingestion**
In order use a language model to interact with your data, you first have to get in a suitable format. That format would be an `Index`. By putting data into an Index, you make it easy for any downstream steps to interact with it.

There are several types of indexes, but by far the most common one is a Vectorstore. Ingesting documents into a vectorstore can be done with the following steps:

1.  Load documents (using a Document Loader)
2.  Split documents (using a Text Splitter)
3.  Create embeddings for documents (using a Text Embedding Model)
4.  Store documents and embeddings in a vectorstore

**Generation**
Now that we have an Index, how do we use this to do generation? This can be broken into the following steps:

1.  Receive user question
2.  Lookup documents in the index relevant to the question
3.  Construct a PromptValue from the question and any relevant documents (using a PromptTemplate).
4.  Pass the PromptValue to a model
5.  Get back the result and return to the user.

# Implementation

We will use python so let's go to the documentation [page](https://python.langchain.com/en/latest/use_cases/question_answering.html) for this use case from Langchain.
This doc provides a quick start where the *ingestion* and *generation* parts are all combined together. Later on we will break it down to understand the components one by one. But first let's see the big picture.

Note that the document we use here is the State of the Union Address, so we need to first create a txt file containing the text in question and save in `../docs/state_of_the_union.txt` -  copy past from [here](https://www.whitehouse.gov/state-of-the-union-2023/). The firefox read mode comes handy to give us the pure text without the extra stuff on the page. If you want a clean version to play with you can find an older state of the union address text file [here](https://github.com/hwchase17/langchain/blob/master/docs/modules/state_of_the_union.txt). If you use the latter, modify your query below to ask a question relevant to that text.

```python
# requires: pip install openai langchain chromadb 
import os
os.environ['OPENAI_API_KEY'] = <your_api_key>

from langchain.document_loaders import TextLoader
loader = TextLoader('../docs/state_of_the_union.txt')
  
from langchain.indexes import VectorstoreIndexCreator
qa_index = VectorstoreIndexCreator().from_loaders([loader])
```

We will ask a question about a topic from the text. The president said the following (quoting from the text):

>	For example, I — I should have known this, but I didn’t until two years ago: Thirty million workers have to sign non-compete agreements for the jobs they take. Thirty million. So a cashier at a burger place can’t walk across town and take the same job at another burger place and make a few bucks more.

Let's ask the bot a question about it.

```python
query = "What did the president say about non-compete agreements?"
qa_index.query(query)
```

The response: 
```
" The president said that they have banned non-compete agreements so companies have to compete for workers and pay them what they're worth."
```

A pretty reasonable response. 

Let's see what's going on under the hood - have been waiting to say this. 

# Implementation break down

### Ingestion

We first address the **ingestion** part of the steps. This means preparing the structure around the documents that we want to use when a user asks a question. The outcome of this part is a vector database, that we can pass to a chain during the **generation** part.


#### VectorIndex
The key functionality we need to understand is the `VectorstoreIndexCreator`, and specifically the following command. 

```python
VectorstoreIndexCreator().from_loaders([loader])
```

What this does is the following: 

1. split the text in chunks (we will discuss the specifics but it doesn't matter to much
2. specify the LLM model which we will use to compute the embeddings (vector values assigned to each chunk)
3. create a database where we store the vectors, we will use `Chroma`

```python 
# 1
documents = loader.load()
from langchain.text_splitter import CharacterTextSplitter
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)

# 2
from langchain.embeddings import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()

# 3
from langchain.vectorstores import Chroma
db = Chroma.from_documents(texts, embeddings)
retriever = db.as_retriever()
```

If we want to preview the **generation** part we can add a fourth step
4. create a chain that uses the vectors to answer questions

```python
# 4
qa_index = RetrievalQA.from_chain_type(llm=OpenAI(), chain_type="stuff", retriever=retriever)

qa_index.run(query) #same query as before
```



## Generation 

Once we have our vector database we can use it for various tasks, aka create different kinds of chains.

#### Retrieve relevant documents
The simplest thing we can do is to retrieve a list of relevant documents to a specific query.  The following code will return 4 documents, i.e. text chunks, ordered by relevance to the query we made.  

```python
docsearch = db.as_retriever()
query = "What did the president say about non-compete agreements?"
docs = docsearch.get_relevant_documents(query)
```

#### Ask questions based on the documents
Once we have our docs, we can ask questions on them with a simple LLM chain, which is made easy using the `load_qa_chain` template. 
```python
from langchain.chains.question_answering import load_qa_chain
from langchain.llms.openai import OpenAI

chain = load_qa_chain(llm=OpenAI(), chain_type="stuff")
query = "What did the president say about non-compete agreements?"
chain.run(input_documents=docs, question=query)
```

From here on we can use the docs we retrieved as we like and combine it with other chains. The most common use case is passing the docs as part of the prompt input, called *context*, and asking the bot to answer using this context. See more examples [here](https://python.langchain.com/en/latest/modules/chains/index_examples/qa_with_sources.html)


## References
 - Langchain [overview](https://python.langchain.com/en/latest/use_cases/question_answering.html) of question and answering with sources
 - Langchain [docs](https://python.langchain.com/en/latest/modules/indexes/getting_started.html) for Indexes.
 - Examples of