---
title: RAG with Ollama & LangChain
---
Retrieval Augmented Generation or RAG, is a technology developed to further the capabilities of LLM's by giving them access to a database full of context that they can use to formulate responses.

You could effectively expand what the LLM 'knows' and give it topic specific information to better answer questions of that topic

This is achieved by taking large amounts of data that you want your Model to use and embedding them, or transposing them into vector space

Embedding can be complicated, but the basics are as follows. An embedding model starts with a large column of keywords or statuses a word could have, so something like: 

	is_noun, is_alive, is_occupation, is_male 

Then, each keyword is weighted by how much it relates to the word you are embedding. So for a test word like 'king' with the keywords above: 

	0, 0.99, 0.7, 0.99

To accurately embed a word you will need very many of these keywords

Then you are left with the embedding of each individual word, but how does that reflect the meaning of an entire sentence? Good question! It doesn't.

You first need a way to reflect the context of each word in the sentence, and then aggregate them. Models like BERT are used for this by generating different embeddings for a word based on its surrounding words, then these are 'pooled' together to create the final sentence embedding. This process can be repeated for larger scopes but can also lose accuracy. Each embedding is stored in a Vector Database for later reference.

The exact math behind this process involves multi-variable calculus and statistics, but after all of this you now have a database full of text that has its semantic and contextually meaning graphed into an N-dimensional vector space where N is the number of keywords mentioned earlier

Now how do we use this to augment an LLM?

Before sending a query to your model, first embed it with the same function we used to embed our vector database earlier (this part is important). Then, compare that embedding with our vector database and pick which parts are the most similar.

This is typically done via cosine similarity, a function that calculates how 'similar' two vectors are by the angle that separates them in space

The reason that functions like Euclidean Distance cannot be used for this is due to a phenomenon known as the 'Curse of Dimensionality', and is also the reason why we can't increase the accuracy of embeddings just by adding millions of qualifying keywords.


A vector database that was produced by a keyword set 784 words long (each word gets 784 other 'words' to describe its meaning) has 784 dimensions in its output vector space, and 784-D space is calculator breaking huge. As the number of dimensions goes up: 1) the data stored within the space gets more and more sparse, and 2) the average distance between points approaches the maximum distance

Essentially, it becomes impossible to graph how similar the embeddings are to each other anymore because they are too far apart


Then we can send the extra data we retrieved as context for a model to answer the query. You can literally pass it directly into a query with something like:

	"Answer this question '{query}' Using the following context: {context}"

The reason that the function you use to embed your database and your queries must be the same is because of the differences in output between different functions. There are a lot of different sophisticated ways to graph words into high dimensional vector space and these different methods or keyword sets produce slightly different embeddings. So using different methods at different links of the chain makes it impossible to relate semantic meaning between two embeddings of different vector spaces

This process alone can still lack accuracy with fetching data related to your query, especially in larger databases. There are other ways to refine the accuracy of our retrieval, such as Re-ranking.

Re-ranking is the process of comparing the data retrieved from the vector database back to the original query. 

During the initial retrieval process you were calculating the similarity between potentially thousands of vectors, and say you wanted to feed your model the top 5 most similar passages to your query. You would just retrieve the the top 5 and feed it to the model.

With Re-ranking we would first retrieve a larger number of similar data like the top 15, and then use those 15 as a second more precise vector database for another retrieval. 

That is the quickest and easiest way to implement a re-ranker and there are obviously 50 sophisticated different ways to increase the accuracy of an LLM, but this solution above could be implemented without much code or effort as the tools to embed and store vector databases exist within LangChain and can be used with free locally-run LLM's like Ollama's 3.1:8b