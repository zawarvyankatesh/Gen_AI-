# GenAI / RAG / Agentic AI — LogicMonitor Prep (Your Differentiator Round)

> This is where you stand out. You've BUILT this (DevPack + AIOps agent). Goal: explain crisply with the right vocabulary and connect every concept back to your projects.

---

## 1. What the JD asks (and what it really means)

| JD phrase | What they'll test |
|---|---|
| "Generative AI concepts, LLMs" | How LLMs work at a high level, tokens, context window, temperature |
| "prompt engineering" | System vs user prompts, few-shot, structured output, reducing hallucination |
| "embeddings" | What a vector embedding is, similarity search, vector DBs |
| "Retrieval-Augmented Generation (RAG)" | Full RAG pipeline end-to-end; RAG vs fine-tuning |
| "Agentic AI frameworks / AI orchestration" | Agents vs plain LLM calls, tool calling, ReAct, LangChain/LangGraph |
| "AI-assisted workflows" | How you actually used AI to automate ops (DevPack, AIOps) |
| (Edwin AI team) MCP, LangFuse, VectorDB | MCP protocol, observability/tracing of LLM apps, vector stores |

**Framing for the interviewer:** LogicMonitor builds **Edwin AI** (AIOps/event correlation). This team builds internal AI automation to run ops. So they want someone who can combine LLMs + cloud-native ops — which is literally your AIOps K8s agent. Say that.

---

## 2. Topics to learn (in order)

### Tier 1 — Must know cold (they WILL ask)
1. **LLM basics** — tokens, context window, temperature/top-p, what "generative" means, hallucination.
2. **Embeddings** — text → vector; cosine similarity; why they enable semantic search.
3. **RAG** — the full pipeline (ingest → chunk → embed → store → retrieve → augment → generate); why RAG vs fine-tuning.
4. **Prompt engineering** — system/user/assistant roles, few-shot, chain-of-thought, JSON/structured output, grounding.
5. **Vector databases** — FAISS, Chroma, Pinecone, pgvector; what they store and how you query.

### Tier 2 — Strongly expected
6. **Agents / Agentic AI** — agent vs LLM call; tool/function calling loop; ReAct pattern; when to use an agent vs a workflow.
7. **MCP (Model Context Protocol)** — what it is, why it standardizes tool access (you built MCP servers!).
8. **LangChain / LangGraph** — chains vs agents vs graphs; what problem they solve.
9. **Hallucination & guardrails** — grounding, citations, output validation, human-in-the-loop, RBAC.
10. **Evaluation & observability** — how to test LLM apps; LangFuse/tracing; eval sets, LLM-as-judge.

### Tier 3 — Nice to have (bonus points)
11. **Fine-tuning vs RAG vs prompt engineering** — when each.
12. **Cost/latency control** — caching, smaller models, token budgeting, streaming.
13. **Chunking strategies** — size, overlap, semantic chunking.
14. **Reranking** — improving retrieval quality (e.g. cross-encoder rerank).
15. **Security** — prompt injection, data leakage, PII handling.

---

## 3. Interview questions + model answers

### A. LLM & GenAI basics

**Q: What is a Large Language Model, in simple terms?**
A neural network (transformer architecture) trained on massive text to predict the next token given previous tokens. "Generative" = it produces new text rather than just classifying. It doesn't "know" facts — it predicts statistically likely continuations, which is why it can hallucinate.

**Q: What is a token? What is the context window?**
A token is a chunk of text (~4 characters / ¾ of a word). The **context window** is the max number of tokens the model can consider at once (input + output). If your prompt + retrieved docs exceed it, you must chunk/summarize. This is *why* RAG chunks documents.

**Q: What does temperature do?**
Controls randomness. `temperature=0` → deterministic, focused (good for RCA, code, extraction). Higher (0.7–1.0) → more creative/varied. **For ops automation you want low temperature** for reliable, repeatable output. (Great to tie to your AIOps agent.)

**Q: What is hallucination and how do you reduce it?**
When the model confidently outputs false info. Reduce with: **grounding** (RAG — give it real context), low temperature, asking it to cite sources / say "I don't know", output validation/guardrails, and human review for high-stakes actions.

---

### B. Embeddings & vector search

**Q: What is an embedding?**
A numeric vector (e.g. 1536 floats) representing the *meaning* of text. Similar meanings → vectors that are close together in space. This lets you do **semantic search** ("find text about DB timeouts") instead of keyword search.

**Q: How do you measure similarity between embeddings?**
**Cosine similarity** (angle between vectors) most commonly; also dot product or Euclidean distance. Higher cosine = more similar.

**Q: What is a vector database and why not a normal DB?**
A DB optimized for storing embeddings and doing fast **nearest-neighbor** search (ANN algorithms like HNSW). Normal DBs can't efficiently find "the 5 most semantically similar vectors" at scale. Examples: FAISS (local), Chroma, Pinecone, Weaviate, pgvector (Postgres extension).

---

### C. RAG (the big one)

**Q: Explain RAG end to end.**
Retrieval-Augmented Generation grounds an LLM in your own data:
1. **Ingest** documents (runbooks, docs, tickets).
2. **Chunk** them into pieces (e.g. 500 tokens, 50 overlap).
3. **Embed** each chunk → vectors.
4. **Store** vectors in a vector DB.
5. **Retrieve**: embed the user's question, find the top-k most similar chunks.
6. **Augment**: stuff those chunks into the prompt as context.
7. **Generate**: LLM answers using that grounded context (+ cite sources).

**Q: Why RAG instead of fine-tuning?**
- RAG: knowledge stays **external and fresh** — update docs, no retraining; cheaper; gives citations; good for facts that change.
- Fine-tuning: changes the model's **behavior/style/format**, not its knowledge; expensive; data goes stale.
- Rule: **RAG for knowledge, fine-tuning for behavior.** Most real problems = RAG (or both).

**Q: What is chunking and why does chunk size matter?**
Splitting docs into retrievable pieces. Too big → irrelevant text dilutes the context and wastes tokens. Too small → loses context/meaning. Overlap keeps continuity across boundaries. Typical: 300–800 tokens with 10–20% overlap.

**Q: How do you improve RAG quality if answers are bad?**
Better chunking; more/better metadata; **reranking** retrieved chunks with a cross-encoder; hybrid search (keyword + semantic); increase/tune top-k; better embeddings model; add a "answer only from context" instruction; evaluate with an eval set.

---

### D. Prompt engineering

**Q: System vs user prompt?**
**System** sets role/rules/constraints ("You are an SRE assistant. Only use the provided context. Output JSON."). **User** is the actual query. **Assistant** is the model's reply. System prompt shapes all behavior.

**Q: What techniques do you use?**
- **Few-shot**: give examples of desired input→output.
- **Chain-of-thought**: "think step by step" for reasoning tasks.
- **Structured output**: request strict JSON (or use JSON mode / function calling) so it's machine-parseable.
- **Grounding**: "answer only from the context below; if not present, say you don't know."
- **Constraints**: limit length, tone, format.

**Q: How do you get reliable JSON out of an LLM?**
Use the provider's JSON/structured-output mode or function calling; give a schema/example; low temperature; validate the output (e.g. with Pydantic) and retry on parse failure.

---

### E. Agents & Agentic AI

**Q: What's the difference between an LLM call and an agent?**
A plain LLM call = one prompt in, one answer out. An **agent** can **use tools** and **loop**: it decides an action (call an API, run a query), observes the result, and repeats until the task is done. It has autonomy over *which* steps to take.

**Q: What is the ReAct pattern?**
**Reason + Act**: the model alternates between reasoning ("I need pod logs") and acting (calls the logs tool), feeding observations back in, until it can answer. It's the core loop of most agents.

**Q: What is tool/function calling?**
You describe tools (name, params, schema) to the LLM. It responds with a structured request to call a tool; your code executes it and returns the result; the LLM continues. This is how agents interact with real systems safely.

**Q: When would you NOT use an agent?**
When the steps are known/fixed — use a deterministic **workflow/chain** instead (more reliable, cheaper, testable). Agents add value only when the path is dynamic. (Your AIOps collector is deterministic on purpose; the LLM only reasons over gathered context.)

**Q: What is MCP (Model Context Protocol)?**
An open standard (from Anthropic) that standardizes how LLM apps connect to tools/data sources — like a "USB-C for AI tools." Instead of custom integrations per tool, an MCP server exposes tools/resources in a common format any MCP-compatible client can use. **You built MCP servers in DevPack** — own this; very few candidates have.

---

### F. Guardrails, evaluation, observability

**Q: How do you stop an AI agent from doing something destructive?**
Layered guardrails: (1) **least-privilege / read-only RBAC** so it *can't* do harm; (2) **human-in-the-loop approval** for write actions; (3) **output validation** against a schema; (4) **allowlists** of permitted actions; (5) **audit logging**. (This is your AIOps story — read-only K8s RBAC, LLM never executes.)

**Q: How do you evaluate an LLM application?**
Build an **eval set** (question → expected answer). Metrics: for RAG, retrieval quality (are right chunks retrieved?) + answer quality (faithfulness, relevance). Use **LLM-as-judge** to score at scale, plus human spot-checks. Track over time as you change prompts/models.

**Q: What is LangFuse / LLM observability?**
Tooling to **trace** LLM app runs — every prompt, retrieval, tool call, token count, latency, cost. Essential for debugging agents and catching regressions in production (parallels normal observability — which LM sells).

---

## 4. Your project talking points (rehearse these)

### AIOps K8s Incident Response Agent (your headline)
- **Pitch:** "Event-driven agent that auto-investigates Kubernetes alerts. Deterministic, read-only collector gathers pod status, events, logs, and Prometheus metrics; an LLM produces a structured RCA with remediation; it emails the on-call. The LLM never touches the cluster — RBAC is read-only."
- **Design decisions to defend:** why read-only (safety/trust), why deterministic collection (testable, LLM only reasons), why fetch *previous* container logs (crash-loop root cause), low temperature (reliable RCA), guardrails.
- **"How would you make it act?"** → separate writer with explicit narrow scope + human approval gate (e.g. propose `rollout restart`, approve in Slack, then execute).

### DevPack (internal AI tooling)
- **Pitch:** "Internal AI tooling suite — MCP servers, Cursor rules, guardrails, and AI-driven deployment validation that cut deployment defects ~20%."
- Be ready to explain: what the MCP servers exposed, what "guardrails" enforced, how you measured the 20%.

---

## 5. Likely design / scenario question

**"Design an AI assistant that answers questions from our internal runbooks."** → Describe a **RAG system**:
- Ingest runbooks → chunk → embed → store in vector DB.
- On a question: embed it, retrieve top-k chunks, build a grounded prompt ("answer only from context, cite the runbook"), generate.
- Add: reranking, citations, eval set, guardrails, caching, LangFuse tracing.
- Mention trade-offs: chunk size, cost, freshness (re-index on doc change).

**"Design an agent that auto-remediates alerts."** → This is your AIOps project. Walk it, then add the human-approval writer for actions.

---

## 6. One-line glossary (rapid recall)

- **LLM**: next-token predictor (transformer).
- **Token**: ~¾ word; unit models process.
- **Context window**: max tokens in/out at once.
- **Temperature**: randomness knob (low = deterministic).
- **Embedding**: text → meaning vector.
- **Cosine similarity**: closeness of two vectors.
- **Vector DB**: stores embeddings, does nearest-neighbor search.
- **RAG**: retrieve relevant docs → put in prompt → generate grounded answer.
- **Chunking**: splitting docs for retrieval.
- **Reranking**: reordering retrieved chunks for relevance.
- **Fine-tuning**: retrain model to change behavior (not knowledge).
- **Agent**: LLM that uses tools and loops to complete tasks.
- **ReAct**: reason ↔ act loop.
- **Tool/function calling**: LLM requests a structured tool invocation.
- **MCP**: open standard for connecting LLMs to tools/data.
- **Hallucination**: confident false output.
- **Guardrails**: constraints keeping AI safe/correct.
- **LLM-as-judge**: using an LLM to grade outputs.
- **LangFuse**: LLM tracing/observability.

---

## 7. Study plan (3–4 short sessions)
1. Read Tiers 1–2 topics; be able to define each glossary term out loud.
2. Rehearse the RAG end-to-end explanation until it's smooth (most-asked).
3. Rehearse your AIOps + DevPack pitches + the "how to make it act" follow-up.
4. Practice the two design questions out loud.
```
Remember: you've BUILT this. Speak from experience, use the exact vocabulary above, and always bridge concept -> your project.
```
