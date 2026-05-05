# week-07-Flowise-Chatbot

## Project Overview

This project is a Retrieval-Augmented Generation (RAG) chatbot that answers cybersecurity questions by searching through real MITRE ATT&CK framework documents rather than relying on the LLM's general training data. The chatbot was built entirely with free-tier cloud tools using Flowise Cloud's visual, no-code drag-and-drop interface — no local software installation required.

RAG works by first converting your documents into searchable numerical representations (embeddings), storing them in a vector database, and then retrieving the most relevant passages whenever a user asks a question. Those passages are handed to the LLM as context, which then generates an answer grounded in your actual documents. This approach dramatically reduces hallucination and enables domain-specific expertise that no general-purpose LLM possesses on its own.

## Live Chatbot

**Try it here:** [https://cloud.flowiseai.com/chatbot/3da86c52-496e-4b36-b5fa-91f4427830aa](https://cloud.flowiseai.com/chatbot/3da86c52-496e-4b36-b5fa-91f4427830aa)

The chatbot can answer questions about Initial Access, Credential Access, and Lateral Movement techniques from the MITRE ATT&CK Enterprise framework. Try asking things like "What is a Golden Ticket attack?" or "What are the different types of spearphishing?" The first message after inactivity may take 30–60 seconds as Flowise re-processes the documents; subsequent messages respond much faster.

## Technology Stack

The RAG pipeline connects six components in Flowise Cloud: three **Text File** document loaders ingest the knowledge base, a **Recursive Character Text Splitter** (chunk size 1000, overlap 200) breaks them into searchable passages, **HuggingFace Inference Embeddings** (`sentence-transformers/all-MiniLM-L6-v2`) converts each chunk into a 384-dimensional vector, an **In-Memory Vector Store** holds and searches those vectors via cosine similarity, **Groq's Llama 3.3 70B Versatile** (temperature 0.3) generates answers from retrieved context, and a **Conversational Retrieval QA Chain** orchestrates the full pipeline with source document attribution enabled.

Groq was chosen for LLM inference because its custom LPU hardware delivers extremely fast response times. HuggingFace provides the embedding model separately because Groq specializes in text generation and does not offer embedding capabilities. Both services are used on their free tiers.

## Knowledge Base

Three documents covering MITRE ATT&CK Enterprise tactics, each containing technique IDs, names, full descriptions, and sub-technique details extracted from [attack.mitre.org](https://attack.mitre.org):

**mitre-initial-access.txt** — Tactic TA0001 with 11 techniques covering how adversaries establish their first foothold, including Phishing (4 sub-techniques), Drive-by Compromise, Supply Chain Compromise, and Valid Accounts.

**mitre-credential-access.txt** — Tactic TA0006 with 17 techniques covering how adversaries steal credentials, including Brute Force (4 sub-techniques), OS Credential Dumping, Kerberos Ticket attacks (Golden/Silver Ticket, Kerberoasting), and Unsecured Credentials.

**mitre-lateral-movement.txt** — Tactic TA0008 with 9 techniques covering how adversaries move through networks, including Remote Services (RDP, SSH, SMB, VNC, WinRM), Pass the Hash, Pass the Ticket, and Software Deployment Tools.

## Repository Structure

```
week-07-Flowise-Chatbot/
├── README.md
├── report.md
├── screenshots/
│   ├── flowise-dashboard.png
│   ├── groq-test.png
│   ├── rag-chatflow.png
│   ├── rag-response-with-sources.png
│   └── chatbot-share-link.png
├── rag-documents/
│   ├── mitre-initial-access.txt
│   ├── mitre-credential-access.txt
│   └── mitre-lateral-movement.txt
└── flowise-export/
    └── Security Knowledge Assistant.json
```

## Evaluation and Findings

Refer to [report.md](report.md) for the full evaluation, including detailed test results for all five questions, edge case observations on hallucination behavior, settings experiment notes, and reflections on how RAG could be applied to the capstone Email Triage & Auto-Responder project.
