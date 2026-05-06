# Slack Post Draft — Context Engineering Part 4: Knowledge Graph

## For: #tmt-solutions (adjust channel tag for #ai-club)

---

Hey TMT - CMRCL - SEs — Part 4 of the context engineering series. This one changed everything.

:spider_web: Teaching Claude to Think in Graphs: Context Engineering Part 4 :spider_web:

Part 1 gave Claude memory. Part 2 showed what happens without it. Part 3 connected it to live data. This one solves the problem I didn't realize I had: Claude remembers everything but finds nothing.

My system had grown to 217 memory files, 86 wiki pages, and 11 rules files — over 300 documents. Session startup would surface 3 recent files by modification time. That's like searching your email by "most recently received." You get something, but rarely what you need.

The fix was a knowledge graph. Not a cloud database or a vector store — a local SQLite file with 379 nodes and 5,120 edges. Every file becomes a node. Every concept, account, product, and project becomes an entity. Typed edges connect them. "What should Claude load for this session?" becomes a graph traversal instead of a timestamp sort.

The engineering challenge was entity quality. Without word-boundary matching, "Exa" (a search tool) matched every file containing "example." Short entity names polluted results everywhere. We added word-boundary regex and IDF filtering — entities appearing in more than 25% of files get excluded from relationship scoring. Edge count dropped 82% (16,274 to 2,877) and every remaining connection was meaningful.

The part that surprised me: validation. We tested 5 real session starts — compared the old mtime-based approach against graph traversal. Graph won all 5. The biggest gap was a Salesforce development session where mtime surfaced 3 unrelated recent files, but the graph pulled the exact architecture rules, data model wiki, and relevant feedback memories. It knew the relationships that recency couldn't.

Three phases, all shipped:
- **P1:** Entity extraction + graph assembly (379 nodes, 5,120 edges)
- **P2:** Session-init integration with automatic fallback (151ms execution)
- **P3:** Auto-indexing hook — graph stays current without manual rebuilds (61ms per file)

The graph now powers every session start. When it's present, Claude loads context by relevance. When it's absent, the old approach kicks in seamlessly. No single point of failure.

Part 4 — https://jtehrani84.github.io/context-engineering-knowledge-graph/
Part 3 — https://jtehrani84.github.io/context-engineering-mcp/
Part 2 — https://jtehrani84.github.io/context-engineering-agentscript/
Part 1 — https://jtehrani84.github.io/claude-context-architecture/
