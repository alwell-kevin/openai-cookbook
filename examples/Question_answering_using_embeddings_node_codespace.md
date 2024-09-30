# Question Answering using embeddings-based search with node in a GitHub Codespace

In a previous [popular article](https://cookbook.openai.com/examples/question_answering_using_embeddings) we learned how to leverage the Completions and Embeddings services to answer questions related to [a dataset](https://cookbook.openai.com/examples/embedding_wikipedia_articles_for_search) using embeddings-based search. This previous implementation was completed in python, using well known python libraries and best practices. 

From this example, you will achieve the same outcome using Node modules and [OpenAI's npm package "OpenAI"](https://www.npmjs.com/package/openai), which as of writing has > 2 million weekly downloads. 

In the example, we leverage a [GitHub CodeSpace](https://docs.github.com/en/codespaces/overview) - a lightweight, cloud hosted VSCode IDE directly intgrated with repositories on GitHub.com. You can use your editor of choice, but this cloud hosted editor allows you to get up and running quickly, with a consistent experience across your team through codified dependencies in a [.devcontainer file](https://docs.github.com/en/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/introduction-to-dev-containers) (we'll see more later). 

### Scope of learning

We do not revisit topics already addressed in [the original article](https://cookbook.openai.com/examples/question_answering_using_embeddings). However, we will learn about:

* When to use a Vector Database versus more traditional, third party search endpoints or [both](https://cookbook.openai.com/examples/question_answering_using_a_search_api)!
* How to implement question answering using embeddings-based search with Node.js.
* How to codify dependencies in a CodeSpace so teammates can get up and running quickly.

### Choosing a methodology

### Implementing search in node

#### Workflow

1. 
