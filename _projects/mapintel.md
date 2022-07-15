---
title: "MapIntel: a system for extracting knowledge from large textual corpora"
excerpt: "MapIntel is a system for efficiently exploring and searching large textual corpora. The system uses embeddings to encode the semantics of documents."
header:
  image: /assets/images/mapintel-header.png
  teaser: /assets/images/mapintel-teaser.png
---
<h3>{{ page.excerpt }}</h3>

***

Efficient knowledge extraction from large bodies of text is a challenge for multiple application domains.
Scientific research, policy analysis, and competitive intelligence are all areas that demand extense scanning and searching through large corpora of documents and that can benefit from a tool that facilitates these processes.
<!-- Add a summary statistic or reference an article or fact fundamenting this argument -->

In this post, we will look at **MapIntel**: a system for visualizing and searching information over large textual corpora by encoding each document as a multidimensional vector that captures its own semantics. 
For a more detailed display of the system, its design decisions and components, refer to the paper ["MapIntel: Enhancing Competitive Intelligence Acquisition Through Embeddings and Visual Analytics"]().

MapIntel differentiates from previously designed systems by promoting serendipity - the finding of useful or potentially useful information not directly sought for. 
In addition, we use sentence embeddings and topic models in our system that leverage the [Transformer architecture]().
This is a somewhat recent arquitecture that has provided significant advancements in multiple Natural Language Processing (NLP) subdomains.
If you want to understand better this architecture I highly advise the blog post ["The Illustrated Transformer"](https://jalammar.github.io/illustrated-transformer/), by Jay Alammar.

# The Architecture
The MapIntel architecture is composed by an *Indexing Pipeline*, which gets documents and their metadata from a source to a database, a *Query pipeline*, which retrieves the most relevant results for a natural language query, and a *Visualization pipeline*, which produces an interactive interface for exploring the document collection.

{% include figure image_path="/assets/images/mapintel-architecture.png" alt="MapIntel architecture" caption="Three main pipelines: Indexing, Query, and Visualization" %}

The system takes a set of documents and computes their **embeddings** - a vectorial representation of a document.
We used [SBERT](), a transformer-based sentence embedding model to produce vectors that encode the essence of each document.
This is the core of our work, as **the system relies on how well the semantics of the documents are captured** to successfuly organize, search and explore the corpus.
All necessary computations, including the embeddings, dimensionality reduction through [UMAP](), and topic modeling through [BERTopic]() are offloaded to not affect interaction with the system.

We use a [Open Distro for ElasticSearch](https://opendistro.github.io/for-elasticsearch/) database as the central repository for our documents, metadata and computational outputs. 
This database was chosen due to the efficient full-text search capabilities it provides. 
Namely, we use its [Approximate Nearest Neighbors (ANN)](https://opendistro.github.io/for-elasticsearch-docs/docs/knn/approximate-knn/) module to retrieve documents that have its embeddings close to a natural language query embedding. 
This is what we call the "Retrieval Bi-Encoder + ANN" module in our architecture.
We subsequently apply a [Cross-Encoder]() model that individually compares all the retrieved documents with the query, filtering out the most relevant documents and returns them to the user.

To provide an interface for exploration and quick identification of the main topics of the corpus, we also produce an interactive 2-dimensional scatter plot of the corpus. 
This map uses UMAP to produce an accurate low-dimensional topological representation of the high-dimensional vector space and BERTopic to capture its main topical cohorts, thus providing a way to visually identify groups of documents that share the same semantical characteristics.

# Evaluation and Results
One of the biggest challenges in our work was how to evaluate such a complex system.

We thought about using the [20 newsgroup dataset]() as it contains labeled documents, which we can use to compare with the identified topics.
In this way, since BERTopic relies on the sentence embeddings, we are also evaluating how well the semantics of each document are captured by the vectors.
We also tried several models both for capturing the topics and embedding the documents.
The model comparison was guided through the optimization of the following metrics: 
- [*kNN Classifier Accuracy*](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html#sklearn.metrics.accuracy_score) to evaluate how well the UMAP projection can preserve the original neighborhood
- [*Normalized Mutual Information (NMI)*](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.normalized_mutual_info_score.html) to assess the match between original labels and the topics assigned
- [*Topic Coherence Cv*](https://radimrehurek.com/gensim/models/coherencemodel.html) to measure the quality of the words that describe each topic

Below we present the best trials for each of the Topic and Embedding model combinations according to the average of the three MinMax scaled objective metrics.

| Topic Model | Embedding Model | MinMax Average | NMI             | Topic Coherence Cv | kNN Classifier Accuracy |
|-------------|-----------------|----------------|-----------------|--------------------|-------------------------|
| BERTopic    | Doc2Vec         | 0.5            | 0.11 ± 0.01     | 0.72 ± 0.02        | 0.16 ± 0.01             |
| BERTopic    | SBERT           | **0.94**       | **0.36 ± 0.03** | **0.76 ± 0.08**    | 0.36 ± 0.01             |
| CTM         | Doc2Vec         | 0.56           | 0.23 ± 0.02     | 0.55 ± 0.02        | 0.24 ± 0.03             |
| CTM         | SBERT           | 0.70           | 0.33 ± 0.02     | 0.58 ± 0.02        | 0.28 ± 0.04             |
| LDA         | Doc2Vec         | 0.58           | 0.25 ± 0.03     | 0.52 ± 0.03        | 0.25 ± 0.01             |
| LDA         | SBERT           | 0.71           | 0.26 ± 0.03     | 0.53 ± 0.03        | **0.37 ± 0.05**         |

The **best model** combination uses both BERTopic and SBERT, having significantly higher values in NMI and Topic Coherence Cv.

We also present in the next figure the comparison of the original labels with the topic labels captured by the best model above.
It is clear that the original topics are correctly captured, proving that the embeddings have learned the original topical labels in a fully unsupervised way.

{% include figure image_path="/assets/images/mapintel-umap.png" alt="Comparison of original and topic labels" caption="Comparison between UMAP planes of **train data** with original (left) and topic labels (right)." %}

# The Prototype
We built a Python web app using [Streamlit]() to demonstrate the capabilities of our system. 
The app allows a user to search a corpus of documents and explore it through a 2-dimensional interactive scatter plot where you can zoom, pan, and hover specific documents to access their content.

Watch the video below to get to know better the MapIntel system and its features.

You can find the code for the Python app at [https://github.com/NOVA-IMS-Innovation-and-Analytics-Lab/mapintel_project](https://github.com/NOVA-IMS-Innovation-and-Analytics-Lab/mapintel_project).

