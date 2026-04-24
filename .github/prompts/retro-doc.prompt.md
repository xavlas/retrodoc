---
description: "Launches full retro-documentation and reverse engineering of an existing application to prepare a future rewrite"
agent: Retro Doc
argument-hint: "appPath=... [targetTech=...]"
---

# Retro Doc

## Inputs

- `${input:appPath}`: (Required) Absolute path to the application to reverse-engineer.
- `${input:targetTech}`: (Optional) Target rewrite technology stack (e.g., "FastAPI + PostgreSQL", "Next.js + tRPC", "Go + gRPC"). Improves migration assessment quality.

## Requirements

1. Analyze the application at `appPath` and produce a complete retro-documentation set in `.retro-doc/`.
2. Use the target technology `targetTech` (if provided) to produce a compatibility matrix and migration recommendations tailored to that stack.
3. Present `.retro-doc/index.md` as the final deliverable and ask whether any area needs deeper analysis.

---

Start by confirming the target application path and the desired rewrite technology (if known), then begin the analysis.
