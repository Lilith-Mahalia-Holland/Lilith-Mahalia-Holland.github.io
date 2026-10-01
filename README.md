# Lilith Holland's Portfolio Projects
## Master's Capstone
### [Predictive Modeling of Weather Station Data](https://lilith-mahalia-holland.github.io/IDC6940_Graph_Neural_Network/)
**Project Overview**

This Master's capstone project investigates whether a Graph Neural Network (GNN) can outperform a univariate time-series regression model for short-term weather forecasting. Using real-world weather station data from southwestern Kansas, we compare a spatiotemporal GNN against a traditional regression baseline for next-step temperature prediction.

## Current Projects
### [Research Gap Identification Through Machine Learning and Network Graphs](https://github.com/Lilith-Mahalia-Holland/gapmap)
**Project Overview**

An end-to-end document intelligence pipeline that ingests unstructured PDFs, extracts spatial layouts via [PP-DocLayoutV3](https://huggingface.co/PaddlePaddle/PP-DocLayoutV3_safetensors), and structures the data into rigid JSON schemas. The framework leverages [SciBERT](https://huggingface.co/allenai/scibert_scivocab_uncased), [BERTopic](https://huggingface.co/MaartenGr/BERTopic_ArXiv), and network graphs to map intersections between research domains and automatically identify literature gaps from user-defined anchor terms.

## Paused Projects
### Distributed LLM Directed Graph Orchestration
**Project Overview**

The goal of this project is to design and build an asynchronous, graph-based LLM orchestration pipeline inspired by [LangChain](https://www.langchain.com/). Developed as a practical implementation of microservice-driven architecture, the project features a decoupled system consisting of a TypeScript front-end, a FastAPI back-end, and a Taskiq distributed task broker.

**Architecture & Technology Stack**
 - **Core Services:** TypeScript (User Interface), FastAPI (REST API Gateway), and Taskiq (Asynchronous broker).
 - **Data & State Management:** Neo4j (Graph Database) and Redis (Caching & Task Queue backend).
 - **Storage:** MinIO (S3-compatible Object Storage for unstructured data).
 - **Inference:** vLLM (Large Language Model hosting) and Text Embeddings Inference (TEI) (Vector embeddings generation).

## Future Projects
 - Finish Diabetic Blood Sugar Tool
 - Implement GPT 2 in PyTorch
 - Update Masters Capstone
