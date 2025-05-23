# GNN-Based Phishing Detection System with LLaMA Auxiliary Classifier

This comprehensive solution integrates graph neural networks (GNNs), a LoRA fine-tuned LLaMA large language model, and a robust preprocessing API to detect phishing URLs with high precision and explainability. Below is a full technical breakdown of each component and the reasoning behind its inclusion.

---

## 🎯 Project Overview
We aim to build an AI system that:
- Accepts a URL and scrapes metadata, text, security headers, and threat intel.
- Converts this structured JSON into a graph format.
- Enhances the graph with historical phishing data and domain linkage behavior.
- Trains a GNN (specifically Graph Attention Network) to classify phishing vs. benign URLs.
- Uses a fine-tuned LLaMA model to verify, explain, or rerank the classification via textual understanding.
- Deploys the system as a FastAPI backend and optionally TorchServe for scalable model serving.

---

## 🔍 Input Data Pipeline: URL Preprocessing API
We build a FastAPI server that ingests a URL and returns a JSON with:
- Redirect behavior
- HTML headers and content
- Text embeddings
- SSL & threat intelligence (e.g., has_ssl, CSP, XSS headers, threat labels)
- Timing data (load time, redirect count)

Each field becomes a node feature in the GNN graph or auxiliary input to the LLaMA classifier.

---

## 🔗 Graph Construction
### 🧱 Nodes:
- **URL Node**: features like original/final URL, entropy, length, redirect count
- **Content Node**: BERT embedding of body text
- **Header Node**: Encoded HTTP headers (categorical, one-hot, or pre-trained tokens)
- **Security Node**: binary features for headers like CSP, HSTS, etc.
- **Threat Intel Node**: contains threat source info, `is_malicious`, confidence
- **Domain Graph Node**: connected to a historical domain co-occurrence knowledge graph

### 🔄 Edges:
- URL → Content
- URL → Headers
- URL → Security
- URL → Threat Intel
- URL ↔ Domain Graph node (learned from past phishing attacks)

This heterogeneous graph is built using **PyTorch Geometric**.

---

## 🧠 GNN Model Architecture: Graph Attention Network (GAT)
### Why GAT?
- Captures edge-type and node importance via attention.
- Well-suited for heterogeneous graphs.

### Architecture:
- Input: GraphData object with node types and edges
- GATConv layers with 8-head attention
- Global mean pooling
- Dense output with softmax (phishing vs non-phishing)

We train using cross-entropy loss and Adam optimizer.

---

## 🦙 Auxiliary Scoring: LLaMA with LoRA Fine-Tuning
### Why LLaMA?
- Strong few-shot and instruction-tuned performance
- Better domain reasoning than GPT-Neo or Flan-T5

### LoRA Fine-Tuning:
- Fine-tune only adapter layers (saves compute)
- Trained on prompts like:
  "Given this metadata, is the site likely phishing?"
  - Text: extracted text_content
  - Headers
  - Security info
  - Threat report summary

### Auxiliary Use:
- Validate or rerank GNN predictions
- Add natural-language rationale
- Flag high-risk predictions for manual review

---

## 🚀 Deployment
### FastAPI:
- `/preprocess`: Takes a URL, returns structured metadata
- `/predict_gnn`: Converts metadata into graph and returns GNN prediction
- `/score_llama`: Uses LLaMA to provide natural-language phishing score

### Optional TorchServe:
- TorchServe model handler for scalable GNN inference
- LLaMA can be containerized with text prompts using HuggingFace pipeline

---

## 🧪 Evaluation Metrics
- Precision, Recall, F1 (focus on high precision)
- ROC-AUC for GNN
- Human-in-the-loop evaluation for LLaMA explanations

---

## 🛠 Future Work
- Incorporate JavaScript behavior analysis
- Temporal graph extensions (URL behavior over time)
- Expand knowledge graph with WHOIS, IP history
- Prompt optimization using Reinforcement Learning (RLHF) for LLaMA

---

## 📦 Deliverables
- FastAPI server
- Graph construction script
- GNN model training pipeline (PyG)
- LoRA fine-tuning script for LLaMA
- Inference and explanation API
