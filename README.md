# PDF Manuals RAG Bot

## Introduction
The purpose of chatbots backed up with retrieval-augmented generation (RAG) is to give the ability to “talk with your data.” This mainly helps to communicate with the bot about more specific information without hoping that it was trained on it and lowering the chance that it will hallucinate on the topic. The general structure of the RAG application is shown in Figure 1 [1].

![Figure 1](https://miro.medium.com/v2/resize:fit:1400/0*WYv0_CaBmCTt7FXc)

The main steps are the following:
1. Define the data extraction method
2. Find an effective chunking method
3. Choose an embedding model (or create one :D)
4. Choose a vector DB that matches the requirements (Depends on time efficiency, cost, complexity, etc.)
5. Pick an LLM
6. Connect everything and test

The lazy, short way to come up with a RAG model is to use the LangChain Framework to cover most of the steps, but let’s see what else we can do.

## Extraction Method
For the PDF processing, I thought we could use pyPDF. Here appears the first problem. As we are working with instruction manuals, most of those will probably contain some illustrations and sketches with important information. After consulting with GPT, I have this code that will extract PDF info including text and images and also a code that will be able to decode images from text (we will need that later for response generation).

I think that this code most probably wouldn’t work as expected but this is just for proof of concept.

## Chunking
Generally, for text documents, page-based chunking is very common. In the case of instruction manuals, those are very structured and clearly divided into parts. From that, I assume that section-based chunking will best fit this problem. Here is the second problem. To split it by sections, we need to understand the sections. We can use SpaCy to find headings, but we need to specify what type of text is considered a heading. For effective management of heading tracking, it will require some time and effort to write a different range of cases when considering text as a heading or subheading. Also, that should be tested to see if it worked out more effectively compared to regular page-based chunking.

After dividing into section chunks, depending on the size of the chunks, those might be divided into smaller chunks by semantic methods. All of the chunks will be indexed in the metadata as (filename)/(# heading)/(# piece) for checking their existence in the future.

## Embedding Model
Taking into account that we have images in our data that we need, the first priority when looking for the embedding model is its ability to process images with text. The first thing that comes up with that ability is the CLIP by OpenAI, which creates a common space for text and images, allowing them to find similarities between images and texts. Here is a GitHub that allows the use of CLIP as an API, as OpenAI doesn’t provide a CLIP endpoint in their APIs: [CLIP as Service](https://github.com/jina-ai/clip-as-service). I think there are also open-source models available that can be used for complete local implementation.

## Vector Database
Nowadays, the Chroma DB is very popular, but I don’t know the exact reason. The first database that comes to my mind is Pinecone because it is pretty easy to use and has a good interface to understand what is going on in your database. However, it is not open source and has a price to it which is a drawback. As an alternative, I would consider Weaviate as an open-source option, which was recommended to me by Dr. Hrant Davidyan from Metric.

## LLM
Again there are two options that I would consider in choosing LLM. It also depends if we want the model to work locally or not. For online implementation, I would most probably use the OpenAI APIs. The GPT-3.5-turbo should work fine for this purpose. Why not GPT-4? Because it is slower and costly. We don’t need the brain of GPT-4 as GPT-3 is already instruction-trained and can complete most of the tasks in our context. It will be connected to a chatbot. When it receives a PDF, it will call a function to the PDF location, call a function to read the data, and check if it is already in the DB or not. If not, then embed it and populate the database. After reading the query, it will find the most relevant chunks. The number of chunks and amount of “surrounding chunks” that would be included in the response context should be decided during testing, but it might return the two most relevant ones (that don’t fall into the neighboring chunk range of each other), and for example, three chunks before and after each matched chunk. Otherwise, if we have a powerful computer, we can use LLAMA 2 or BLOOM.

## Connect and Test
First, we connect everything, and the main intersection happens in writing the logic of LLM to understand what to do when it receives a PDF document and the question related to it. I think there also might be a problem with the relevance of the questions. During the testing, we can yield a similarity search threshold that will define if the PDF has that information, or the chatbot will say, “I didn’t find anything about that in the PDF…” and optionally give their own trained opinion on the matter. After connecting all these components and writing a regulatory prompt for the LLM to let it know what kind of chatbot it is, we can go for the testing and tuning. For testing, we can write some ground truth cases for several documents and run tests. It must include different types of information, like finding exact numbers or measurements, as well as some kind of instructions on how to do something. Based on test results, we will be tuning different parameters, like chunking parameters, ‘not found’ threshold, context size, etc.

## Resources
[1] B. L. ([online]. Available: https://gradientflow.com/techniques-challenges-and-future-of-augmented-language-models/). "Techniques, Challenges, and Future of Augmented Language Models". Gradient Flow. Accessed: Jun. 19, 2024.
