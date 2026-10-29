# Architecture Notes

## Components

### Contract ingestion
The upload form accepts one PDF per submission. The PDF data loader extracts document text and attaches the source filename as metadata. A recursive character splitter creates chunks, which are passed to a Google Gemini embedding node and stored in an in-memory vector store.

### Question answering
The chat trigger forwards the user's message to the Contract RAG Agent. The agent uses a Gemini chat model, short-window memory, and the `contract_search` vector-store tool. Retrieved passages provide the evidence used to compose the answer.

### Retrieval diagnostics
A separate manual-trigger path queries the vector store directly. Use it to inspect whether relevant passages are being retrieved before diagnosing answer-generation behavior.

## Data path

```text
PDF upload
  -> PDF text loader + source metadata
  -> recursive text splitter
  -> Gemini embeddings
  -> in-memory vector store

Chat question
  -> RAG agent
  -> contract_search + query embeddings
  -> retrieved passages
  -> Answer / Evidence / Source
```

## Important implementation checks

- Confirm ingestion and retrieval point to the same vector-store key.
- Confirm that the same compatible embedding model is used on both paths.
- Confirm chunk size, overlap, and top-k values in the imported workflow; update this document if they change.
- Keep the source filename in metadata so reviewers can locate the original PDF.
- Validate all output against the original clause before making a decision.
- Reconnect your own credentials after importing the sanitized workflow.
- The workflow export is inactive by default; test it before activating it.

## Demo versus production

The in-memory vector store is convenient for a proof of concept but is not durable storage. A production implementation should add persistent storage, authentication, role-based access, document-level permissions, duplicate/version detection, audit logs, monitoring, and a repeatable evaluation suite.
