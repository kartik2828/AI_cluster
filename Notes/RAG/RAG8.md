### Summary: Corrective RAG (C-RAG) Explained and Implemented

This section explores **Corrective RAG (C-RAG)**, an advanced variant of the traditional Retrieval-Augmented Generation (RAG) system used in natural language processing tasks. The content covers the conceptual foundation of RAG, issues with conventional RAG workflows, and step-by-step implementation of C-RAG using LangGraph and OpenAI's LLMs, closely following a recent 2024 research paper.

---

### Core Concepts and Workflow of Traditional RAG

- **Traditional RAG Workflow**:
  1. **Query Embedding**: User query $x$ is converted to a vector via an embedding model.
  2. **Retrieval**: Semantic search over a vector database retrieves relevant documents $d_1, d_2, ...$.
  3. **Augmentation**: Retrieved documents and query are combined and sent as context to a Large Language Model (LLM).
  4. **Generation**: LLM generates an answer strictly based on the documents provided.

- **Problem with Traditional RAG**:
  - LLM **blindly trusts** retrieved documents.
  - If documents are irrelevant or partially related, the answer can be **incorrect or hallucinated**.
  - Example: If query is "What is an LLM?" but documents cover only machine learning without LLM info, irrelevant documents (e.g., Random Forest) may be retrieved, causing wrong answers.
  - This can have serious consequences in business or critical domains.

---

### Introduction to Corrective RAG (C-RAG)

- **C-RAG's Key Innovation**: It introduces a **Retrieval Evaluator model** that assesses the **relevance and quality of retrieved documents** before passing them to the LLM.
  
- **Evaluator Model Functionality**:
  - Takes the user query $x$ and retrieved documents $d_i$.
  - Outputs a **relevance score** and a **judgment** for each document.
  - Classifies retrieval results into **three cases**:
    1. **Correct**: Relevant documents found.
    2. **Incorrect**: No relevant documents found.
    3. **Ambiguous**: Mixed relevance; some documents partially useful.

- **Three Corresponding C-RAG Paths**:
  - **Correct Case**: Proceed with normal RAG — refine documents and generate answer.
  - **Incorrect Case**: Perform a **web search** (external knowledge source) to fetch relevant documents, then refine and generate answer.
  - **Ambiguous Case**: Combine retrieved and web documents, refine combined context, then generate answer.

- **Critical Difference**: C-RAG **does not blindly trust retrieval**; it **actively evaluates** and **corrects** retrieval before generation.

---

### Stepwise Implementation Highlights

- **Dataset and Setup**:
  - Uses three classic ML/DL books as document sources.
  - Documents are split into chunks (~900 characters) to form the vector database.
  - OpenAI embedding and LangGraph for workflow orchestration.

- **Knowledge Refinement Module**:
  - Breaks down retrieved documents into **sentence-level strips**.
  - Uses an LLM (ChatGPT in this case, as the fine-tuned T5 model is unavailable) to **filter** strips by relevance to the query.
  - Only relevant strips are retained and recombined into a refined context to improve answer quality.

- **Retrieval Evaluation Module**:
  - Sends each retrieved document with query to LLM evaluator.
  - Evaluator returns:
    - A **numerical relevance score** between 0 and 1.
    - A **reason** explaining the score.
  - Scores compared against **thresholds**:
    - Lower threshold $\theta_{low} = 0.3$ (example)
    - Upper threshold $\theta_{high} = 0.7$ (example)
  - Judgments:
    - If any document score $> \theta_{high}$ → Correct
    - If all scores $< \theta_{low}$ → Incorrect
    - Otherwise → Ambiguous

- **Graph-based Workflow Routing**:
  - Based on evaluator verdict, routes query through:
    - Refined retrieval only (correct)
    - Web search followed by refinement (incorrect)
    - Combined documents refinement (ambiguous)

- **Web Search Integration**:
  - Uses **Tawtly API** for web search when retrieval is insufficient.
  - Web documents are filtered similarly to internal docs.
  - Query rewriting is performed before web search to improve search relevance:
    - LLM rewrites vague user query into a keyword-rich, search-engine-optimized query.
    - Example: "Who was the screenwriter for Death of a Batman?" → "Death of a Batman screenwriter Wikipedia"
  - Rewritten query yields better search results, improving retrieval in external knowledge case.

---

### Timeline and Iterative Development of C-RAG Features

| Iteration | Feature/Step                                           | Description                                                                            |
|-----------|-------------------------------------------------------|----------------------------------------------------------------------------------------|
| 1         | Traditional RAG baseline                              | Retrieval → Augmentation → Generation with blind trust in retrieved documents          |
| 2         | Knowledge Refinement                                  | Document chunk decomposition, filtering irrelevant parts, recombining refined context  |
| 3         | Retrieval Evaluation                                  | LLM-based evaluator scoring retrieved docs, classifying retrieval quality              |
| 4         | Routing based on evaluation                           | Branching workflow for correct, incorrect, ambiguous retrieval cases                    |
| 5         | Web Search integration                                | External retrieval when internal docs insufficient                                    |
| 6         | Query Rewriting                                      | Rewriting user queries before web search to optimize search results                    |
| 7         | Ambiguous case handling                               | Combining retrieved and web docs for best context and generation                       |

---

### Key Insights and Conclusions

- **Corrective RAG significantly improves reliability** of RAG systems by **monitoring and correcting retrieval quality**, addressing hallucination risks in LLM outputs.

- **Retrieval Evaluator model** is central to C-RAG, enabling selective use of documents and fallback to external knowledge sources when needed.

- **Knowledge Refinement** filters noisy or partially relevant chunks, improving answer precision.

- **Web Search and Query Rewriting** add robustness for queries not covered by internal documents.

- The entire architecture can be built incrementally, starting from traditional RAG and adding C-RAG features step-by-step.

- Due to lack of access to fine-tuned T5 models, OpenAI's ChatGPT is used as a practical alternative for filtering and evaluation, illustrating flexibility in implementation.

- The design leverages **LangGraph** for modular, node-based workflow management with clear routing logic.

- The presented code and architecture are closely aligned with the original 2024 research paper on Corrective RAG, providing a practical blueprint for real-world deployment.

---

### Summary Table: Document Evaluation Thresholds and Outcomes

| Condition                                            | Threshold Criteria                         | Outcome          | Action                                 |
|-----------------------------------------------------|--------------------------------------------|------------------|--------------------------------------|
| **Correct Retrieval**                                | At least one document score $> \theta_{high}$ (e.g., 0.7) | Correct          | Use good docs for refinement & generation |
| **Incorrect Retrieval**                              | All document scores $< \theta_{low}$ (e.g., 0.3)           | Incorrect        | Trigger web search and external refinement |
| **Ambiguous Retrieval**                              | Scores fall between $\theta_{low}$ and $\theta_{high}$     | Ambiguous        | Combine retrieved & web docs; refine and generate |

---

### Keywords

- **RAG (Retrieval-Augmented Generation)**
- **Corrective RAG (C-RAG)**
- **Retrieval Evaluation**
- **Knowledge Refinement**
- **Query Rewriting**
- **Web Search Integration**
- **Vector Database**
- **Semantic Search**
- **LangGraph**
- **Large Language Models (LLM)**
- **T5 Transformer**
- **OpenAI API**
- **Thresholding**
- **Hallucination**
- **Information Retrieval**

---

