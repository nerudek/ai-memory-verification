---
name: memory-palace
description: Memory Palace verification toolkit — does your AI memory ACTUALLY work? Tests file layer AND semantic search (ChromaDB). Find silent failures before you trust your agent's memory.
version: 1.0.0
author: nerudek
compatible-with: hermes-agent, openclaw, claude-code
---

# Memory Palace — Verification Toolkit

## Problem

AI agents claim their memory works. They say "I'll remember that." But their memory is broken silently — ChromaDB crashes on Python 3.14, embeddings never get stored, and semantic search returns nothing. You don't find out until you ask "what did we discuss last week?" and get a blank stare. Without a verification toolkit, you trust broken memory for weeks before noticing. This skill provides the exact tests to run before trusting any Memory Palace installation.

## Solution

Two-layer verification: test the file system layer (always works) AND the ChromaDB semantic layer (often broken). If ChromaDB fails, you know immediately that RAG and embeddings are dead — and you don't waste time debugging agent behavior that's caused by broken memory.

## Quick Test

```bash
# Layer 1: File system (should always pass)
echo "test" > ~/.mempalace/palace/test.md && cat ~/.mempalace/palace/test.md

# Layer 2: ChromaDB semantic search (the one that breaks)
python3 -c "import chromadb; c=chromadb.PersistentClient(path='$HOME/.mempalace/palace'); print(c.list_collections())"
```

If Layer 2 fails with a Pydantic error: your semantic search is dead. Downgrade to Python 3.12 or wait for chromadb to support Pydantic v2.

## Known Issue: Python 3.14 + ChromaDB

**Root cause:** `chromadb` depends on Pydantic v1, which is incompatible with Python 3.14+.
**Symptom:** `pydantic.v1.errors.ConfigError: unable to infer type for attribute`
**Fix:** Use Python 3.12, or wait for upstream chromadb fix.

## Full State Matrix (Mac Mini M4, 2026-05-13)

| Layer | Status | Notes |
|-------|--------|-------|
| File write | OK | Markdown files persist |
| File read | OK | All agents can read |
| ChromaDB | BROKEN | Python 3.14 incompatibility |
| iii-engine | NOT INSTALLED | Never deployed |
| agentmemory | NOT INSTALLED | Never deployed |
| Auto-session-log | NOT IMPLEMENTED | No automatic capture |

## Bottom Line

**Memory Palace works only as a file folder until ChromaDB is fixed.** Semantic search, RAG, and embeddings are all dead on Python 3.14+. Do not claim "memory works" without running both tests.

## FAQ

**Q1: What is Memory Palace?**
A: A hybrid memory system for AI agents: Markdown file layer (always works) + ChromaDB semantic search (Python-dependent). Used by Claude Code, OpenClaw, and Hermes agents.

**Q2: How do I know if my Memory Palace is actually working?**
A: Run both tests from the Quick Test section. File layer test writes and reads a file. ChromaDB test tries to list collections. Both must pass.

**Q3: Why does ChromaDB break on Python 3.14?**
A: ChromaDB's dependency chain requires Pydantic v1 (`pydantic.v1`), which was removed in Python 3.14. The `on-error` import fallback doesn't work because `pydantic.v1` is a namespace package that can't be imported conditionally.

**Q4: What's the fix for the ChromaDB issue?**
A: Downgrade to Python 3.12, use a venv with Python 3.12, or wait for chromadb to update their Pydantic dependency to v2.

**Q5: Can I still use Memory Palace without ChromaDB?**
A: Yes — the file layer works fine. Agents can read and write Markdown files. You lose semantic search and embeddings, but basic memory works.

**Q6: What is iii-engine and why isn't it installed?**
A: iii-engine is the "ULTRA layer" for agentmemory — a faster, more advanced memory backend. It was planned but never deployed on this setup.

**Q7: Does this affect all AI agents the same way?**
A: Only agents that use ChromaDB for semantic search. Plain file-based memory (Obsidian vault, markdown notes) works fine regardless of Python version.

**Q8: How do I check which Python version I'm running?**
A: `python3 --version`. If it shows 3.14+, ChromaDB is likely broken.

**Q9: What are the symptoms of broken ChromaDB?**
A: Agent says "I saved that" but can't recall it later. Semantic search returns nothing. No errors visible in normal agent operation — it fails silently.

**Q10: Can I run ChromaDB in a Docker container instead?**
A: Yes. `docker run -p 8000:8000 chromadb/chroma` — this bypasses the local Python version. Recommended for production.

**Q11: Is auto-session-log implemented?**
A: Not yet. It was planned as automatic session capture to Memory Palace but was never built. Currently, session logging requires manual triggers.

**Q12: How do I set up Memory Palace from scratch?**
A: `pip install chromadb` (on Python <=3.12), create `~/.mempalace/palace/`, and configure your agent to use it. Verify with the Quick Test.

**Q13: Does this affect ClawHub/OpenClaw skills that use Memory Palace?**
A: Yes — any skill that imports `chromadb` will fail on Python 3.14+. Check the skill's dependencies before installing.

**Q14: Will this be fixed upstream?**
A: Eventually. ChromaDB will need to migrate from Pydantic v1 to v2. No timeline available. Track: `github.com/chroma-core/chroma`.

**Q15: What's the recommended setup for new installations?**
A: Python 3.12 venv + chromadb. Or Docker container. Avoid Python 3.13+ until chromadb confirms Pydantic v2 support.

---

If this saved you time: [PayPal.me/nerudek](https://www.paypal.me/nerudek)
GitHub: [github.com/nerudek](https://github.com/nerudek)
