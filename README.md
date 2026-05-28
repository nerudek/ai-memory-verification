<div align="center">

# AI Memory Verification 🧪

**Does Your AI Memory ACTUALLY Work? Test Both Layers — File System AND Semantic Search (ChromaDB)**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/nerudek/ai-memory-verification?style=flat-square)](https://github.com/nerudek/ai-memory-verification/stargazers)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/nerudek/ai-memory-verification/pulls)

Find silent failures before you trust your agent's memory. ChromaDB broken? File layer survives.

</div>

---

## Table of Contents

- [Problem Statement](#1-problem-statement)
- [Solution Overview](#2-solution-overview)
- [Architecture](#3-architecture)
- [Quick Start](#4-quick-start)
- [Layer 1: File System Test](#5-layer-1-file-system-test)
- [Layer 2: ChromaDB Semantic Test](#6-layer-2-chromadb-semantic-test)
- [State Matrix](#7-state-matrix)
- [Known Issues](#8-known-issues)
- [FAQ](#9-faq)
- [Contributing & Support](#10-contributing--support)

---

## 1. Problem Statement

AI agents claim their memory works. They say "I'll remember that." But their memory is broken silently — ChromaDB crashes on Python 3.14, embeddings never get stored, and semantic search returns nothing. You don't find out until you ask "what did we discuss last week?" and get a blank stare.

| Problem | Impact |
|---------|--------|
| **ChromaDB silently fails** | Semantic search returns zero results — no errors shown |
| **Cross-version Python breakage** | `chromadb` depends on Pydantic v1, removed in Python 3.14+ |
| **False sense of security** | Agent says "memory stored" but no embedding was ever persisted |
| **No automated verification** | Teams trust broken memory for weeks before discovering it |

Without a verification toolkit, you trust broken memory until it's too late.

---

## 2. Solution Overview

A two-layer verification toolkit that tells you, in under 30 seconds, exactly which parts of your AI memory system are working:

| Layer | What It Tests | Expected Status | How to Verify |
|-------|--------------|----------------|--------------|
| **File System** | Write + read markdown files | Always works | `echo "test" > file && cat file` |
| **ChromaDB Semantic** | Vector collections, embeddings | Often broken | `python3 -c "import chromadb; ..."` |

If Layer 2 fails, you know immediately that RAG and embeddings are dead — and you don't waste time debugging agent behavior caused by broken memory.

---

## 3. Architecture

```
┌─────────────────────────────────────────────────────┐
│              AI MEMORY VERIFICATION                   │
│                                                      │
│  ┌──────────────────────┐   ┌────────────────────┐  │
│  │  Layer 1: File Sys   │   │  Layer 2: ChromaDB  │  │
│  │                      │   │                    │  │
│  │  write → persist     │   │  collections →     │  │
│  │  read  → retrieve    │   │  embeddings →      │  │
│  │  status: ✅ ALWAYS   │   │  status: ❌ PTB    │  │
│  └──────────────────────┘   └────────────────────┘  │
│           ▲                          ▲               │
│           │     verification         │               │
│           └──────────┬───────────────┘               │
│                      │                               │
└──────────────────────┼───────────────────────────────┘
                       │
            ┌──────────┴──────────┐
            │                     │
            ▼                     ▼
     ┌──────────────┐    ┌──────────────┐
     │  Any Agent   │    │   Human      │
     │  (Claude,    │    │   Operator   │
     │  OpenClaw,   │    │              │
     │  Hermes)     │    │              │
     └──────────────┘    └──────────────┘
```

### Test Flow

```
Agent claims "memory works"
        │
        ▼
Run Layer 1: File system test
        │
        ├── PASS → File layer is healthy
        │
        ▼
Run Layer 2: ChromaDB test
        │
        ├── PASS → Semantic search is working (full RAG capability)
        │
        └── FAIL → Semantic search is dead, file-only mode (graceful degradation)
```

---

## 4. Quick Start

```bash
# Clone the repository
git clone https://github.com/nerudek/ai-memory-verification.git
cd ai-memory-verification

# Run the combined verification (or use SKILL.md for detailed instructions)
```

---

## 5. Layer 1: File System Test

The file system layer is the most reliable memory path. Markdown files persist across sessions, agent switches, and reboots.

### How to Verify

```bash
# Write a test file
echo "test-memory-palace-works" > ~/.mempalace/palace/test.md

# Read it back
cat ~/.mempalace/palace/test.md
```

**Expected result:** `test-memory-palace-works` printed to stdout.

### Why It Always Works

| Property | Detail |
|----------|--------|
| Storage medium | Plain text files on disk |
| Dependencies | None (OS-level read/write only) |
| Python version | Independent |
| Network required | No |
| Agent required | No — humans can read too |

---

## 6. Layer 2: ChromaDB Semantic Test

The semantic search layer (ChromaDB) is where things break. Python version changes silently kill it.

### How to Verify

```bash
python3 -c "import chromadb; c=chromadb.PersistentClient(path='$HOME/.mempalace/palace'); print(c.list_collections())"
```

### Expected Results

| Python Version | ChromaDB Expected Status | Notes |
|---------------|------------------------|-------|
| 3.10 – 3.12 | ✅ Working | Pydantic v1 compatible |
| 3.13 | ⚠️ May work | Depends on chromadb release |
| 3.14+ | ❌ Broken | Pydantic v1 removed from stdlib |

### Failure Symptom

```
pydantic.v1.errors.ConfigError: unable to infer type for attribute
```

If you see this error: **semantic search is dead.** Downgrade to Python 3.12 or wait for upstream chromadb to support Pydantic v2.

---

## 7. State Matrix

Current verified state (Mac Mini M4, tested 2026-05-13):

| Layer | Status | Notes |
|-------|--------|-------|
| File write | ✅ OK | Markdown files persist |
| File read | ✅ OK | All agents can read |
| ChromaDB collections | ❌ BROKEN | Python 3.14 incompatibility |
| ChromaDB embeddings | ❌ BROKEN | Same root cause |
| Semantic search | ❌ BROKEN | Downstream of ChromaDB |
| RAG (retrieval-augmented gen) | ❌ BROKEN | No embeddings available |
| iii-engine | ⬜ NOT INSTALLED | Never deployed |
| agentmemory | ⬜ NOT INSTALLED | Never deployed |
| Auto-session-log | ⬜ NOT IMPLEMENTED | No automatic capture |

---

## 8. Known Issues

### Python 3.14 + ChromaDB

**Root cause:** `chromadb` depends on Pydantic v1 (`pydantic.v1`), which was removed in Python 3.14. The `on-error` import fallback doesn't work because `pydantic.v1` is a namespace package that can't be imported conditionally.

**Symptom:**
```
pydantic.v1.errors.ConfigError: unable to infer type for attribute
```

**Fix options (pick one):**

| Option | Command | Pros | Cons |
|--------|---------|------|------|
| Downgrade Python | `pyenv install 3.12` | Reliable fix | Requires pyenv |
| Docker ChromaDB | `docker run -p 8000:8000 chromadb/chroma` | Bypasses local Python | Needs Docker |
| Use ChromaDB HTTP client | Point at remote ChromaDB | No local deps | Network latency |
| Wait for upstream fix | — | No work needed | No timeline |

**Track upstream:** [github.com/chroma-core/chroma](https://github.com/chroma-core/chroma)

---

## 9. FAQ

**Q1: What is AI Memory Verification?**
A: A two-layer test toolkit that verifies whether your AI agent's memory system is actually working — file system layer (always works) and ChromaDB semantic layer (often broken).

**Q2: How do I know if my memory is actually working?**
A: Run both tests from Quick Start. File layer test writes and reads a file. ChromaDB test tries to list collections. Both must pass.

**Q3: Why does ChromaDB break on Python 3.14?**
A: ChromaDB requires Pydantic v1 (`pydantic.v1`), which was removed from Python 3.14's standard library. See Known Issues.

**Q4: What's the fix for the ChromaDB issue?**
A: Downgrade to Python 3.12, use a venv with Python 3.12, or run ChromaDB in Docker.

**Q5: Can I still use AI memory without ChromaDB?**
A: Yes — the file layer works fine. Agents can read and write Markdown files. You lose semantic search and embeddings, but basic memory works.

**Q6: Does this affect all AI agents the same way?**
A: Only agents that use ChromaDB for semantic search. Plain file-based memory (Obsidian vault, markdown notes) works fine regardless of Python version.

**Q7: What are the symptoms of broken ChromaDB?**
A: Agent says "I saved that" but can't recall it later. Semantic search returns nothing. No errors visible — it fails silently.

**Q8: Can I run ChromaDB in Docker?**
A: Yes. `docker run -p 8000:8000 chromadb/chroma` bypasses the local Python version issue entirely. Recommended for production.

**Q9: How do I check which Python version I'm running?**
A: `python3 --version`. If it shows 3.14+, ChromaDB is likely broken.

**Q10: What is the recommended setup for new installations?**
A: Python 3.12 venv + chromadb, or Docker container. Avoid Python 3.13+ until chromadb confirms Pydantic v2 support.

**Q11: What is the difference between file layer and semantic layer?**
A: File layer = Markdown files on disk (bulletproof, keyword-searchable). Semantic layer = ChromaDB vector embeddings (meaning-based retrieval, fragile).

**Q12: How do I set up Memory Palace from scratch?**
A: `pip install chromadb` (on Python <=3.12), create `~/.mempalace/palace/`, and configure your agent to use it. Verify with both tests.

**Q13: Does this affect Hermes Agent skills that use Memory Palace?**
A: Yes — any skill that imports `chromadb` will fail on Python 3.14+. Check the skill's dependencies before installing.

**Q14: Will ChromaDB be fixed upstream?**
A: Eventually. ChromaDB needs to migrate from Pydantic v1 to v2. No timeline available.

**Q15: Is auto-session-log implemented?**
A: Not yet. It was planned but never built. Currently, session logging requires manual triggers.

---

## 10. Contributing & Support

Contributions are welcome! Here's how to help:

1. **Fork** the repository
2. **Create a feature branch:** `git checkout -b feat/my-verification-addon`
3. **Commit your changes:** `git commit -am 'feat: add memory layer test for Qdrant'`
4. **Push:** `git push origin feat/my-verification-addon`
5. **Open a Pull Request**

Please ensure your changes maintain backward compatibility and clearly document expected test results.

---

**License:** MIT — see [LICENSE](LICENSE) for details.

**Issues:** [GitHub Issues](https://github.com/nerudek/ai-memory-verification/issues)

**Author:** [@nerudek](https://github.com/nerudek) on GitHub

---

<div align="center">

If this saved you time: [PayPal.me/nerudek](https://www.paypal.me/nerudek)

⭐ Star the repo if you find this useful!

</div>

---

See [SKILL.md](./SKILL.md) for the skill reference card (Hermes Agent / Claude Code skill manifest).
