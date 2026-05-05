# Week 7: RAG Security Knowledge Assistant — Evaluation Report

## 1. Setup Summary

- **LLM:** Llama-3.3-70b-versatile (via Groq) with a temperature of 0.3 and streaming enabled
- **Embeddings:** sentence-transformers/all-MiniLM-L6-v2 using the HuggingFace Inference API
- **Vector Store:** In-memory vector store with Top K set to 4
- **Text Splitter:** Recursive Character Text Splitter (chunk size: 1000, overlap: 200)
- **Documents loaded:**
  1. `mitre-initial-access.txt` — MITRE ATT&CK Tactic TA0001: Initial Access (11 techniques including Phishing sub-techniques, Supply Chain Compromise, Valid Accounts) — approximately 3 pages
  2. `mitre-credential-access.txt` — MITRE ATT&CK Tactic TA0006: Credential Access (17 techniques including Brute Force, OS Credential Dumping, Kerberos Tickets, Unsecured Credentials) — approximately 4 pages
  3. `mitre-lateral-movement.txt` — MITRE ATT&CK Tactic TA0008: Lateral Movement (9 techniques including Remote Services, Pass the Hash, Remote Service Session Hijacking) — approximately 3 pages

## 2. Test Results

| # | Question | Used Documents? | Quality | Notes |
|---|----------|----------------|---------|-------|
| 1 | What is spearphishing and what are its different sub-techniques in the ATT&CK framework? | Yes | Good | All four sub-techniques (Attachment, Link, via Service, Voice) correctly identified with accurate descriptions matching the T1566 entries in the Initial Access document. The chatbot prefaced its answer with "according to the provided context," indicating RAG grounding. |
| 2 | What are the differences between brute force techniques like password guessing, password spraying, and credential stuffing? | Yes | Good | Accurately distinguished all three T1110 sub-techniques and synthesized a comparative summary covering target scope, password list source, and account lockout behavior. The specific example 'Password01' for password spraying matched the source document verbatim. |
| 3 | How do adversaries use valid accounts for both initial access and lateral movement? | Yes | Good | Successfully retrieved T1078 (Valid Accounts) content spanning both the Initial Access and Lateral Movement documents. Minor hallucination observed: the chatbot added security recommendations (multi-factor authentication, least privilege) not present in the source documents, though these were contextually appropriate. |
| 4 | What is a Golden Ticket attack and how does it relate to Kerberos authentication? | Yes | Good | Precise retrieval of T1558.001 (Golden Ticket) details including the KRBTGT password hash, TGT forging, and Active Directory access. Correctly contextualized within the broader T1558 Kerberos ticket-stealing technique family from the Credential Access document. |
| 5 | What techniques can adversaries use to move laterally through a network after gaining initial access? | Yes | Partial | Retrieved several correct Lateral Movement techniques (Exploitation of Remote Services, SSH, RDP, Windows Admin Shares) from the source document. However, the chatbot hallucinated PsExec and WMI (not present in any uploaded document) and incorrectly attributed Credential Access techniques (AiTM, ARP Cache Poisoning, LLMNR/NBT-NS Poisoning) as lateral movement methods. This demonstrates that broad questions increase hallucination risk as the LLM supplements retrieved context with training data. |

## 3. Edge Case Observations

- **Unrelated question ("What is the weather like today?"):** The chatbot responded with “Hmm, I’m not sure,” which is the expected behavior. Even though the vector store returned unrelated chunks, the model correctly ignored them and didn’t generate a false answer. This shows the system stays controlled when questions are completely outside its domain.

- **Related topic not in documents ("What are the latest CVEs from 2026?"):** The chatbot again responded with “I’m not sure.” Even though this question is cybersecurity-related, the model recognized that the retrieved content didn’t actually answer the question and avoided guessing. This shows the system can handle domain-related but unsupported questions properly.
  
- **Closely related tactic not uploaded ("What techniques do adversaries use for data exfiltration?"):** Here, the chatbot gave a detailed but completely incorrect answer and claimed it came from the provided context. None of the listed techniques were in the uploaded documents. This is the most serious issue because the question was very close to the domain, making the model confident enough to rely on its training data instead of the retrieved content. This type of hallucination is risky and could be reduced by stricter prompts or similarity thresholds.

## 4. Settings Experiments

Settings experiments were not performed during this lab session due to time constraints. The default configuration (temperature 0.3, chunk size 1000, chunk overlap 200, Top K 4) produced strong results for most test cases, suggesting these are reasonable defaults for a cybersecurity knowledge assistant. Future experimentation could explore reducing temperature to 0.1 to further constrain hallucination on broad questions like Q5, increasing Top K to 6 for better cross-document retrieval, or testing smaller chunk sizes (500) to improve precision on specific technique lookups.

## 5. Reflection

**What surprised me about how RAG works:** The most surprising finding was the inconsistency of hallucination behavior across the three edge case questions. The chatbot correctly admitted uncertainty for completely unrelated questions (weather) and even domain-adjacent questions (CVEs), but hallucinated confidently when asked about a closely related MITRE ATT&CK tactic (exfiltration) that was not in the knowledge base. This revealed that hallucination risk in RAG is not binary — it exists on a spectrum determined by the semantic proximity between the question and the uploaded documents. The closer the question is to the domain of the knowledge base, the more likely the LLM is to blend its training data with retrieved context and produce a convincing but unsupported answer.

**How I could improve this chatbot for real-world use:** To make this system more reliable, a few improvements are needed. Switching from an in-memory store to something like Pinecone or Chroma would improve scalability. Adding stricter prompts would ensure the model says “I don’t know” when needed. Setting a similarity threshold would prevent irrelevant data from being used. Expanding the knowledge base to include more MITRE ATT&CK tactics and frameworks like NIST CSF would also improve accuracy and coverage.

**How I might use RAG in my capstone project:** For my capstone project (Email Triage and Auto-Responder system), RAG could be used to pull in company policies, procedures, and guidelines when generating responses. This would make replies more accurate and aligned with organizational standards instead of relying only on general AI knowledge. It would be especially useful for handling sensitive or compliance-related emails. I could build on the Flowise setup from this lab and integrate it with my n8n workflow to create a complete system.
