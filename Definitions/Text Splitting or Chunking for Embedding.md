---
title: Text Splitting or Chunking for Embedding
dateCreated: 2024-11-12
dateModified: 2025-04-02
---

# Text Splitting or Chunking for Embedding

## Overview

In the context of building LLM-related applications, **chunking** is the process of breaking down large pieces of text into smaller segments. It's an essential technique that helps optimize the relevance of the content we get back from a vector database in [[RAG]] systems once we use the [[LLM]] to embed content.

## Details

When a **sentence** is embedded, the resulting vector focuses on the sentence's specific meaning. The comparison would naturally be done on that level when compared to other sentence [[AI]]. This also implies that the embedding may miss out on broader contextual information found in a paragraph or document.

When a **full paragraph or document** is embedded, the embedding process considers both the overall context and the relationships between the sentences and phrases within the text. This can result in a more comprehensive vector representation that captures the broader meaning and themes of the text. Larger input text sizes, on the other hand, may introduce noise or dilute the significance of individual sentences or phrases, making finding precise matches when querying the index more difficult.

The length of the query also influences how the [[AI]] relate to one another. A shorter query, such as a single sentence or phrase, will concentrate on specifics and may be better suited for matching against sentence-level [[AI]]. A longer query that spans more than one sentence or a paragraph may be more in tune with [[AI]] at the paragraph or document level because it is likely looking for broader context or themes.

The index may also be non-homogeneous and contain [[AI]] for chunks of *varying* sizes. This may pose challenges in terms of query result relevance, but it may also have some positive consequences. On the one hand, the relevance of the query result may fluctuate because of discrepancies between the semantic representations of long and short content. On the other, a non-homogeneous index could potentially capture a wider range of context and information since different chunk sizes represent different levels of granularity in the text. This could accommodate different types of queries more flexibly.

**Which embedding model are you using, and what chunk sizes does it perform optimally on?**
A model like [text-embedding-ada-002](https://openai.com/blog/new-and-improved-embedding-model) performs better on chunks containing 256 or 512 tokens.

**What are your expectations for the length and complexity of user queries?**
If they are long and complex your chunking strategy should be contain more tokens.

### Chunking Methods

There are different methods for chunking, and each of them might be appropriate for different situations. By examining the strengths and weaknesses of each method, our goal is to identify the right scenario to apply them to.

### Fixed-size Chunking

This is the most common and straightforward approach to chunking: we simply decide the number of tokens in our chunk and, optionally, whether there should be any overlap between them. In general, we will want to keep some overlap between chunks to make sure that the semantic context doesn't get lost between chunks. Fixed-sized chunking will be the best path in most common cases. Compared to other forms of chunking, fixed-sized chunking is computationally cheap and simple to use since it doesn't require the use of any NLP libraries.

### "Content-aware" Chunking

These are a set of methods for taking advantage of the nature of the content we're chunking and applying more sophisticated chunking to it. Here are some examples:

#### Sentence Splitting

As we mentioned before, many models are optimized for embedding sentence-level content. Naturally, we would use sentence chunking, and there are several approaches and tools available to do this, including:

- **Naive splitting:** The most naive approach would be to split sentences by periods (".") and new lines. While this may be fast and simple, this approach would not take into account all possible edge cases. Here's a very simple example:

```python
text = "..." # your text
docs = text.split(".")
```

- [**NLTK**](https://www.nltk.org/): The Natural Language Toolkit (NLTK) is a popular Python library for working with human language data. It provides a sentence tokenizer that can split the text into sentences, helping to create more meaningful chunks. For example, to use NLTK with LangChain, you can do the following:

```python
text = "..." # your text
from langchain.text_splitter import NLTKTextSplitter
text_splitter = NLTKTextSplitter()
docs = text_splitter.split_text(text)
```

- [**spaCy**](https://spacy.io/): spaCy is another powerful Python library for NLP tasks. It offers a sophisticated sentence segmentation feature that can efficiently divide the text into separate sentences, enabling better context preservation in the resulting chunks. For example, to use spaCy with LangChain, you can do the following:

```python
text = "..." # your text
from langchain.text_splitter import SpacyTextSplitter
text_splitter = SpaCyTextSplitter()
docs = text_splitter.split_text(text)
```

#### Recursive Chunking

Recursive chunking divides the input text into smaller chunks in a hierarchical and iterative manner using a set of separators. If the initial attempt at splitting the text doesn't produce chunks of the desired size or structure, the method recursively calls itself on the resulting chunks with a different separator or criterion until the desired chunk size or structure is achieved. This means that while the chunks aren't going to be exactly the same size, they'll still "aspire" to be of a similar size.

Here's an example of how to use recursive chunking with [LangChain](https://python.langchain.com/en/latest/modules/indexes/text_splitters/examples/recursive_text_splitter.html):

```python
text = "..." # your text
from langchain.text_splitter import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
    # Set a really small chunk size, just to show.
    chunk_size = 256,
    chunk_overlap  = 20
)

docs = text_splitter.create_documents([text])
```

### Figuring out the Best Chunk Size for Your Application

Here are some pointers to help you come up with an optimal chunk size if the common chunking approaches, like fixed chunking, don't easily apply to your use case.

- **Preprocessing your Data** - You need to first pre-process your data to ensure quality before determining the best chunk size for your application. For example, if your data has been retrieved from the web, you might need to remove HTML tags or specific elements that just add noise.
- **Selecting a Range of Chunk Sizes** - Once your data is preprocessed, the next step is to choose a range of potential chunk sizes to test. As mentioned previously, the choice should take into account the nature of the content (e.g., short messages or lengthy documents), the embedding model you'll use, and its capabilities (e.g., token limits). The objective is to find a balance between preserving context and maintaining accuracy. Start by exploring a variety of chunk sizes, including smaller chunks (e.g., 128 or 256 tokens) for capturing more granular semantic information and larger chunks (e.g., 512 or 1024 tokens) for retaining more context.
- **Evaluating the Performance of Each Chunk Size** - In order to test various chunk sizes, you can either use multiple indices or a single index with multiple [namespaces](https://docs.pinecone.io/docs/namespaces). With a representative dataset, create the Artificial Intelligence for the chunk sizes you want to test and save them in your index (or indices). You can then run a series of queries for which you can evaluate quality, and compare the performance of the various chunk sizes. This is most likely to be an iterative process, where you test different chunk sizes against different queries until you can determine the best-performing chunk size for your content and expected queries.

## Key Points

- Chunking or text splitting is a process of breaking down large texts into smaller segments which helps in optimizing the relevance of content from a vector database.
- There are several factors to consider when deciding the size of chunks, including the specific embedding model used and expectations for user query length and complexity.
- Common methods for chunking include fixed-size chunking and content-aware chunking like sentence splitting and recursive chunking.
- The choice of method depends on specific requirements such as computational efficiency, simplicity, or need for more sophisticated context preservation.
- To determine optimal chunk size one must preprocess data, select range of potential sizes based on content nature and model capabilities, evaluate performance using multiple indices/namespaces with representative dataset.

## References / Sources

1. [pinecone.io - learn chunking strategies](https://www.pinecone.io/learn/chunking-strategies/)
2. [OpenAI's New & Improved Embedding Model](https://openai.com/blog/new-and-improved-embedding-model/)
3. [Natural Language Toolkit (NLTK)](https://www.nltk.org/)
4. [spaCy - Python Library for NLP tasks](https://spacy.io/)
5. [LangChain Text Splitting Tool](https://python.langchain.com/en/latest/modules/indexes/text_splitters/examples/recursive_text_splitter.html)
6. [Pinecone Documentation - Namespaces](https://docs.pinecone.io/docs/namespaces)

## Reflections / Thoughts

While there is no 'one-size-fits-all' solution to determine the ideal chunk size or text splitting strategy due to varying demands across different applications, having an understanding about these strategies offers a solid starting point in this aspect of machine learning workflow.

One should be mindful that while larger chunks may provide richer context information, they might also introduce noise which could dilute significance of individual sentences within it affecting precision when querying indexes later on.

Therefore it becomes crucial to not just understand your input data well but also anticipate how users might interact with it during queries before deciding upon suitable parameters like token limit per segment etc., ultimately aiming at maintaining balance between retaining useful contextual information while minimizing unnecessary noise resulting through unwanted parts.

The mention about using separate indices or namespaces based testing approach highlights another important concept where ML engineers can design experiments involving varied parameter combinations iteratively refining them over time against set benchmarks thus enabling us optimize models better serving unique needs each project poses accurately meeting its specifications precisely!

# Text Splitting

This notebook covers text [5 Levels Of Text Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/main/tutorials/LevelsOfTextSplitting/5_Levels_Of_Text_Splitting.ipynb)

## Levels of Text Splitting

- **Level 1: [Character Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/8a30b5710b3dd99ef2239fb60c7b54bc38d3613d/tutorials/LevelsOfTextSplitting/#CharacterSplitting)** - Simple static character chunks of data
- **Level 2: [Recursive Character Text Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/8a30b5710b3dd99ef2239fb60c7b54bc38d3613d/tutorials/LevelsOfTextSplitting/#RecursiveCharacterSplitting)** - Recursive Artificial Intelligence based on a list of separators
- **Level 3: [Document Specific Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/8a30b5710b3dd99ef2239fb60c7b54bc38d3613d/tutorials/LevelsOfTextSplitting/#DocumentSpecific)** - Various Artificial Intelligence methods for different document types (PDF, Python, Markdown)
- **Level 4: [Semantic Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/8a30b5710b3dd99ef2239fb60c7b54bc38d3613d/tutorials/LevelsOfTextSplitting/#SemanticChunking)** - Embedding walk based Artificial Intelligence
- **Level 5: [Agentic Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/8a30b5710b3dd99ef2239fb60c7b54bc38d3613d/tutorials/LevelsOfTextSplitting/#AgenticChunking)** - Experimental method of splitting text with an agent-like system. Good for if you believe that token cost will trend to $0.00
- ***Bonus Level:*** **[Alternative Representation Chunking + Indexing](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/8a30b5710b3dd99ef2239fb60c7b54bc38d3613d/tutorials/LevelsOfTextSplitting/#BonusLevel)** - Derivative representations of your raw text that will aid in retrieval and indexing
