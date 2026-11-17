---
name: grill-with-docs
description: A relentless interview to sharpen a plan or design, which also decides what the docs (ADRs and glossary) should say, ending in tickets ready to implement.
disable-model-invocation: true
---

1. Call the Skill tool twice, for "grilling" and "domain-modeling". Use domain-modeling to decide what the glossary and ADRs should say, and hold each result as a **direct change** in the shared understanding, unwritten, as grilling says. This replaces domain-modeling's inline `GLOSSARY.md` updates and lazy file creation: the direct-change ticket writes the files.
2. Once the user confirms the shared understanding, call the Skill tool with "to-tickets" to turn it into tickets.
3. After the tickets are published, list them and end with the next command, then do nothing else:
   - Two or more tickets: "Run `/clear`, then `/implement-batch <first ticket>`."
   - One ticket: "Run `/clear`, then `/implement <ticket>`."
