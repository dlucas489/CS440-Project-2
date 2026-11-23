# Intent: Dynamic Agent Skills & Telemetry Logging

## Objective
Extend `mini_coding_agent.py` to discover, match, and dynamically inject domain-specific playbooks (`skills/<skill-name>/SKILL.md`) into the agent's execution loop, accompanied by structured console telemetry.

## Problem Statement
Hardcoding domain workflows into the base system prompt increases token overhead, degrades local model focus, and prevents modular reuse across tasks. The agent needs a lightweight mechanism to load specialized instructions only when relevant to the user's request.

## Core Capabilities
1. **Cross-Platform Skill Discovery:** Scan `skills/<skill-name>/SKILL.md` using `pathlib.Path` to parse skill metadata (`name`, `description`, `triggers`) and instructional bodies across Windows, macOS, and Linux.
2. **Dual-Mode Triggering & Injection:**
   - **Automatic Similarity Routing:** Score incoming prompts against discovered skill metadata and inject matching `SKILL.md` instructions when a configurable threshold is met.
   - **Explicit Tool Invocation:** Expose a `use_skill` tool allowing the LLM to load a skill mid-execution when indirect prompts fall below the initial similarity threshold.
3. **Run Telemetry Logging:** Emit a deterministic console log on every agent run capturing:
   - The raw user input prompt.
   - The similarity match score evaluated against every discovered skill.
   - The triggered skill name (or `None`).

## Success Criteria
- Base system prompt stays compact, containing only brief skill summaries rather than full playbooks.
- Relevant `SKILL.md` instructions are injected cleanly into the active prompt loop when triggered automatically or via `use_skill`.
- Console logs accurately report prompt, per-skill scores, and triggered skill state on every invocation.
