# Cleantech ReAct Agentic AI System

## Overview
A production-grade multi-tool Agentic AI system built for specialized research in the cleantech sector. The system moves beyond standard RAG pipelines by implementing a LlamaIndex ReAct agent capable of dynamic tool orchestration — combining local historical knowledge, real-time web search, complex reasoning, and ethical guardrailing in a single, hierarchical workflow.

## System Architecture

The agent follows a strict, prioritized tool hierarchy to maximize efficiency and answer quality:

```
Internal RAG → Web Search → Reasoning Engine → Safety Guardrail
```

The agent reasons about which tool to invoke at each step, iterating until it produces a verified, comprehensive answer — functioning less like a search engine and more like a research project manager.

## Technical Implementation

### Part 1: RAG Foundation
- **Dataset:** `cleantech_media_dataset_v3_2024-10-28` — domain-specific cleantech articles and reports
- **Embedding Model:** HuggingFace BGE-Small (`BAAI/bge-small-en-v1.5`) — chosen for high semantic search performance with low latency
- **Chunking Strategy:** `SentenceSplitter` with 3,072-token chunks to preserve technical context integrity and prevent fragmentation of cross-referenced regulations or multi-paragraph specifications
- **Retrieval Optimization:** Two-stage LLMRerank postprocessor — retrieves k=50 candidates via k-NN vector search, then uses GPT-4o-mini to re-score and select the top 5 most semantically precise chunks

### Part 2: ReAct Agent and Custom Tool Extensions
Three custom tools built from scratch using LlamaIndex's `FunctionTool` wrapper:

| Tool | Purpose |
|---|---|
| `cleantech_media_search` | Core RAG retriever — queries the local vector index for historical domain knowledge |
| `web_search_tool` | Real-time web search for current events and out-of-scope queries |
| `reasoning_tool` | Routes complex synthesis, comparison, and calculation tasks to a dedicated GPT-4o reasoning engine |
| `safety_check_tool` | Post-generation guardrail — checks final responses for safety, toxicity, and topic adherence before output |

### Part 3: Evaluation
- **Methodology:** LLM-as-Judge using GPT-4o to score responses across three metrics: Faithfulness, Relevance, and Completeness
- **Test Set:** 23 complex, multi-faceted questions from the CleanTech evaluation dataset
- **Design Targets:** Faithfulness 0.85+, Relevance 0.90+, Completeness 0.80+

**Note on Evaluation Results:** Final metric generation was blocked by an `InvalidStateError` stemming from asynchronous execution conflicts in the LlamaIndex ReAct loop within Google Colab's dynamic runtime environment. Root cause analysis confirmed this was an environmental race condition — not a logic flaw in the system. The evaluation CSV is included to show full execution trace data. Re-running in a Dockerized environment with fixed dependency versioning is the identified fix.

## Key Design Decisions
- **Why BGE-Small over larger embedding models?** Superior computational efficiency with competitive semantic search quality — optimizes latency for real-time RAG
- **Why 3,072-token chunks?** Cleantech documents contain dense, cross-referenced technical content — large chunks prevent the "lost-in-the-middle" problem
- **Why LLMRerank over standard retrieval?** Standard k-NN retrieval maximizes recall but not precision. LLMRerank adds a second-stage semantic filter to eliminate marginal context before synthesis
- **Why LLM-as-Judge over ROUGE/BLEU?** Token-overlap metrics cannot measure factual grounding or semantic utility — both critical for RAG evaluation

## Tools and Technologies
- **Language:** Python
- **Frameworks:** LlamaIndex (ReAct Agent, VectorStoreIndex, LLMRerank)
- **LLMs:** GPT-4o-mini (agent reasoning), GPT-4o (evaluation judge)
- **Embeddings:** HuggingFace BGE-Small
- **Environment:** Google Colab

## Repository Structure
```
cleantech-react-agent/
├── cleantech_react_agent.ipynb          # Full implementation (Parts 1, 2, 3)
├── cleantech_evaluation_results.csv     # Evaluation execution trace
├── cleantech_react_agent.pdf            # Full technical report
└── README.md
```

> **Note:** API keys have been removed from the notebook. Set your own `OPENAI_API_KEY` environment variable before running.
> The cleantech dataset is not included. It can be sourced from cleantech media archives or substituted with a domain-specific document corpus.

## Author
**Vaishnavi Tiwari**  
M.S. Data Science, DePaul University  
[LinkedIn](https://www.linkedin.com/in/vaishnavi-tiwari-1a18971ab) · [GitHub](https://github.com/tiwari-vaish)
