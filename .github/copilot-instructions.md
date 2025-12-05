# Copilot / AI Agent Instructions

This repo implements a small Retrieval-Augmented Generation (RAG) demo. The goal of an AI coding agent working here is to get the RAG assistant running, complete the TODOs in `src/app.py`, and keep changes minimal and well-tested.

Key places to look
- `src/app.py`: Central RAG flow. Contains `RAGAssistant`, LLM initialization, prompt template, and TODOs for `load_documents()` and `invoke()`.
- `requirements.txt`: Shows the heavy use of `langchain`, `langchain-openai`, `langchain-groq`, `langchain-google-genai`, and `chromadb`.
- `data/` (expected): The code reads `../data` for `.txt` files—add sample `data/*.txt` documents for local testing.

Big-picture architecture and conventions
- RAGAssistant: initializes an LLM (OpenAI -> Groq -> Google fallback order) using env vars (`OPENAI_API_KEY`, `GROQ_API_KEY`, `GOOGLE_API_KEY`). Prefer adding keys to a `.env` file for local dev.
- Vector DB abstraction: `vectordb.VectorDB` is used with `add_documents()` and (expected) `search()` methods. Treat `VectorDB` as the single point for indexing/search.
- Prompt + chain pattern: `ChatPromptTemplate.from_template(...)` is piped into an LLM and `StrOutputParser()` via `self.prompt_template | self.llm | StrOutputParser()` — keep this pipeline style when adding or modifying generation code.

Project-specific behaviour to preserve
- LLM selection order and zero temperature (`temperature=0.0`) — tests and reproducibility rely on deterministic outputs.
- Prompt template must include `{context}` and `{question}` placeholders.
- Documents are expected as plain text files in `data/` by current `load_documents()` hints.

Immediate actionable tasks an agent can perform
- Implement the RAG query pipeline (`RAGAssistant.invoke`):
  - Use `self.vector_db.search(query, k=n_results)` to retrieve chunks.
  - Join retrieved chunks into a single `context` string (use separators like `"\n\n---\n\n"`).
  - Call the chain with the filled prompt, e.g. `self.chain.invoke({'context': context, 'question': input})` and return the parsed string.
- Fix the CLI loop in `main()` — it currently calls `assistant.query(question)` but the class exposes `invoke()`; either add a `query()` delegating to `invoke()` or change `main()` to call `invoke()`.
- Ensure `load_documents()` handles missing `data/` gracefully and returns an empty list if none found.

Run, debug, and dev workflows (Windows PowerShell)
- Install deps (prefer virtualenv):
  - `python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install -r requirements.txt`
- Configure env: create a `.env` in the repo root with at least one key, e.g.:
  - `OPENAI_API_KEY=sk-...`
  - optionally `OPENAI_MODEL`, `GROQ_MODEL`, `GOOGLE_MODEL`
- Run the demo:
  - `python .\src\app.py`
  - Or: `python -m src.app`

Patterns and examples
- LLM initialization example (preserve this flow):
  - Check `os.getenv('OPENAI_API_KEY')` first, then `GROQ_API_KEY`, then `GOOGLE_API_KEY`.
- Prompt template example in `src/app.py` (keep structure):
  - "You are a helpful assistant. Use the following context to answer the question.\n\nContext:\n{context}\n\nQuestion: {question}\nAnswer:"

Pointers for tests and verification
- Manual smoke test: add 2-3 small `data/*.txt` files, set `OPENAI_API_KEY`, run `python -m src.app`, ask a question that should be answered from the text.
- Unit tests: None are present. If adding tests, target:
  - `RAGAssistant._initialize_llm()` behavior by mocking `os.getenv()`.
  - `load_documents()` file reading for different file types.
  - `invoke()` pipeline using a fake `VectorDB` that returns predictable chunks and asserting chain is called with expected `context`.

What not to change without asking
- Do not change the LLM fallback order or default zero temperature unless the user requests it.
- Avoid swapping out the prompt pipeline style; extend it only by composing additional prompt fragments.

If anything is unclear or you want me to implement the `invoke()` and fix the CLI loop, tell me which to do first and whether you prefer a simple implementation or more robust error handling and tests.
