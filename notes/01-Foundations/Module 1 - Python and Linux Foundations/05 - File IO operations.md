# File I/O Operations

## What & Why

Ingesting enterprise documents (PDFs, logs, config files) always starts with reading bytes off disk correctly. Getting encoding and streaming wrong here causes garbled text or memory blowups long before any AI logic runs.

## How It Works

```python
# Always use a context manager — guarantees the file is closed even on error
with open("report.txt", "r", encoding="utf-8") as f:
    text = f.read()

# Binary mode for non-text files (PDFs, images)
with open("scan.pdf", "rb") as f:
    data = f.read()

# Streaming a large file instead of loading it all into memory
with open("huge_log.txt", "r", encoding="utf-8") as f:
    for line in f:
        process(line)
```

- **Text mode (`"r"`/`"w"`)**: decodes/encodes bytes using a specified encoding (default depends on OS — always pass `encoding="utf-8"` explicitly).
- **Binary mode (`"rb"`/`"wb"`)**: raw bytes, no decoding — required for PDFs, images, and other non-text formats.
- **Streaming**: iterating line-by-line (or in fixed-size chunks for binary) keeps memory flat regardless of file size.
- **File locking**: prevents two processes from writing the same file concurrently and corrupting it (relevant for shared logs/config on a server).

## Pitfalls

- Opening a file without `with` — on an exception between `open()` and `close()`, the file handle leaks.
- Assuming UTF-8 when a legacy enterprise export is actually Latin-1/Windows-1252 — causes `UnicodeDecodeError` or silently mangled characters.
- Loading an entire multi-GB file into memory with `.read()` when line-by-line streaming would do.

## Connections

Directly reused when ingesting legacy PDFs and documents in Module 8 (*Multimodal RAG and Vision AI* → *Ingesting unstructured legacy enterprise PDFs*).
