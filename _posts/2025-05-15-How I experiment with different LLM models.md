---
layout: post
date: 2025-05-16
title: How to try different llm models
giscus_comments: true
related_posts: false
tags: 
  - llm
---


### Why LLM is a Great Package for Automation and Productivity

The [LLM package by Simon Willison](https://llm.datasette.io/en/stable/index.html) offers a unique combination of features that make it an ideal tool for research, automation and productivity. Kudos to Simon for making such a powerful piece of software. In this article, I'll explore three reasons to use it.

### Reason 1: Easy Experimentation with Different LLMs

One of the most significant benefits of the LLM package (LLM) is its ability to make it easy to experiment with different LLM models. With LLM, you can store all your results and revisit them later to examine differences or find answers to questions you've previously asked. This feature is particularly useful for exploring the capabilities of different LLMs and identifying the best one for your specific use case. 

There are other good ways to run different llm models online, such as [hyperbolic](https://app.hyperbolic.xyz) or hugging face [inference playground](https://huggingface.co/playground), and some to run them locally, such as [LM Studio](https://lmstudio.ai/). And of course there are all the chatbot interfaces from OpenAI, Anthropic, Google AI Studio etc. which are truly fantastic products. But if you are interested in trying them all out, there are actually too many places which makes tracking your usage too hard. The llm-package solves this problem for me to a good degree. I use it to interact with all the API services and open source models including all local models. This way I am capturing all the interactions automatically. All prompts and outputs are [stored automatically](https://llm.datasette.io/en/stable/logging.html) in a local database together with all the metadata.  LLM has a variety of plugins for many API services and local inference frameworks. For the rare occasion that there is an API that is not covered, you can write your own plugin (Simon has kindly provided a detailed guide on how to do exactly that - and if you do other will benefit too!). What I cannot capture are my interactions with ChatGPT or other chatbot interfaces, but at least for those there is a history saved on the interface.

### Reason 2: Easy Local Model Running

LLM also makes it very easy to run models locally. With LLM, you can access a wide range of local inference frameworks, all within a single package. Each one is made available via an `llm`-plugin that can be easily installed via `pip`. You can find the full list here. This feature is particularly useful for those who want to run models on their own computer without incurring costs or relying on external cloud services. What is more, all data stays with the user, which is something I appreciate a lot. Below I am showing how to run [Llama3-3B-4bit](https://huggingface.co/mlx-community/Llama-3.2-3B-Instruct-4bit) locally. This model is powerful enough for common tasks, yet small and very performant. Combined with the mlx inference framework it is possible to run this model easily on the background without having your computer slow down if you are running your regular apps in parallel.

**Running Lama3 Locally on Your Mac**
**System Requirements**
* Mac Pro (or a MacBook Pro machine)
* Local folder with virtual environment and local python version 3.12. We specifically need 3.12 and not higher. See my notes on ["How I set up my project repositories"](<2025-05-14-How I set up my project repositories.md>). 
* Enter your project repo and activate the virtual environment

**Step 1: Install llm**
Follow the [instructions](https://llm.datasette.io/en/stable/setup.html) for setting up llm. Inside the virtual environment we can use pip. 
```bash
pip install llm
```

**Step 2: Install llm-mlx**
Follow the [instructions](https://github.com/simonw/llm-mlx?tab=readme-ov-file) of the llm-mlx plugin. Since we have setup our virtual environment with python 3.12 already we don't need to follow the special instructions listed. The following one line will do. 
```bash
llm install llm-mlx
```

**Step 3: Install the Llama model**
This will take a minute or two to download. 
```bash
llm mlx download-model mlx-community/Llama-3.2-3B-Instruct-4bit
```

**Step 4: Test it out**
Try prompting the model  with a simple query
```bash
llm -m mlx-community/Llama-3.2-3B-Instruct-4bit 'Capital of France?'
```


- You can run other local models as listed on the [repo](llm -m mlx-community/Llama-3.2-3B-Instruct-4bit 'Capital of France?' -s 'you are a pelican') of the plugin on github
- To delete a downloaded model amd free space on your computer follow the instructions [here](https://github.com/simonw/llm-mlx/issues/14)

### Reason 3: Customizable Workflows in the Terminal

Finally, the fact that LLM is integrated into the terminal, combined with the fact that Simon has built in certain features that are very helpful, allows us to create custom little workflows to automate tasks such as writing blog posts. With LLM, you can create a custom terminal command that automates the process of writing blog posts, saving time and effort.

### Example 

**Using llama3B to Automate Tasks**

How I wrote this blog post. First I ran the following command to create a chat template (a great LLM feature). This stores a preference for a model and a system prompt I want to reuse in the future. 

```bash 
llm llm -m mlx-community/Llama-3.2-3B-Instruct-4bit -s "You are my helpful copyright editor that writes for my AI blog. You can take in some quick notes that I give you and you turn it into clear text. If you don't have all the details to complete the article you put placeholders for me to fill in later. You use simple language in a neutral and professional tone. You do not hype the topics or get too excited. You output only markdown files." --save copywriter
```

Then I started a chat with this template

```bash
llm chat -t copywriter
```

Then I went back a forth a few times with the model to write the pieces of the post, one prompt at a time. I didn't get a perfect response each time, but within 5 iterations I got what I needed to put the post together. I had to edit it manually for 10 mins before completing it. 

**Tips and Modifications**
By combining LLM features with a dictation app, you can replace typing by speaking when prompting which makes the process a lot faster. This can be particularly useful for tasks that require a lot of text input, such as writing emails, creating reports, or drafting documents. I use [Macwhisperer](https://goodsnooze.gumroad.com/l/macwhisper) which runs locally. This way I have a pipeline that runs on my laptop, for free, requires no internet access and no data leaves my computer. 

Simon has added a ton of useful features to the LLM package which make it easy to automate various workflows. You can take some ideas from his video demo, or check out the list of tools and plugins available on the documentation. 

