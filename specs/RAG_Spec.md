# RAG Specification — Medical Query Assistant

## 1. Approved Data Sources
- Internal hospital knowledge base (curated clinical guidelines, drug formularies, treatment protocols).
- Approved textbooks and journals indexed by the medical library.
- Patient records (accessed only through tool calls, not directly indexed in RAG).

## 2. Chunking Strategy
- **Chunk size**: 512 tokens (empirically determined for paragraph-level retrieval).
- **Overlap**: 64 tokens (12.5%) to preserve context at boundaries.
- **Metadata per chunk**:
    {
      "source_id": "string",
      "document_title": "string",
      "section": "string",
      "published_date": "YYYY-MM-DD",
      "access_level": "public | restricted",
      "owner_department": "string"
    }

## 3. Retrieval Method
- **Primary**: Hybrid retrieval (dense vector + BM25) with weight 0.7/0.3.
- **Reranking**: Cross-encoder (bi-encoder) reranks top-20 retrieved chunks to top-5.
- **Filters applied pre-retrieval**:
  - `access_level` must match user`s role.
  - `published_date` must be within last 5 years (unless no recent result).

## 4. Grounding Rules
- **No source, no claim**: Every factual statement in the answer must be backed by at least one retrieved chunk. If no relevant chunk is found, the assistant must say “I don’t have evidence on this topic.”
- **Cite sources**: Each chunk used is listed in the `sources` field of the output.
- **Distinguish evidence from inference**: Direct quotes are marked with quotation marks; paraphrases are attributed to the source.
- **Grounding failure metric**: Ratio of ungrounded claims to total claims. Threshold: < 0.05. If exceeded, trigger alert and log the conversation.

## 5. Update & Versioning
- The RAG index is rebuilt weekly or when any approved source is updated.
- Each index version is tagged with a date and the hash of the source manifest.