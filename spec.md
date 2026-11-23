# Specification: Dynamic Agent Skills & Telemetry Logging

## 1. Directory & File Format (`skills/<skill-name>/SKILL.md`)
- **Path Resolution:** Resolve `Path(repo_root) / "skills"` using `pathlib.Path` exclusively to ensure POSIX and Windows compatibility.
- **Discovery Rule:** Iterate sorted subdirectories of `skills/`; load any directory containing a `SKILL.md` file. Ignore missing `skills/` directories gracefully.
- **Metadata Schema:** Parse optional `---` YAML frontmatter (`name`, `description`, `triggers` as comma-separated or bracketed strings) plus the Markdown body. Fallback to directory name and first non-empty body line if frontmatter is omitted.

## 2. Similarity Scoring & Thresholding
- **Tokenization:** Lowercase text, extract alphanumeric/hyphenated/underscored tokens, split compound identifiers, and filter stop words.
- **Scoring Function (`score_skill`):**
  - Return `1.0` on exact substring match of `name` (or hyphen-normalized `name`) or any explicit entry in `triggers`.
  - Otherwise compute `round(min(0.7 * prompt_coverage + 0.3 * jaccard, 1.0), 4)`, where `prompt_coverage = |P ∩ S| / |P|` and `jaccard = |P ∩ S| / |P ∪ S|`.
- **Trigger Threshold:** Default `--skill-threshold` = `0.25`. Select the highest-scoring skill where `score >= threshold`; otherwise `None`.

## 3. Prompt Injection & Explicit Tooling
- **Base Prefix (`build_prefix`):** Include an `Available Skills` block listing `- <name>: <description>` only (omitting full bodies).
- **Dynamic Injection (`prompt`):** When `self.active_skill` is set, inject `Active Skill Instructions (<name>):\n<body>` between `Memory` and `Transcript`.
- **Explicit Tool (`use_skill`):** Register a non-risky `use_skill(name: str)` tool in `build_tools()` and `validate_tool()`. Calling `use_skill` sets `self.active_skill` and returns the `SKILL.md` body so the LLM can self-activate skills mid-loop.

## 4. Console Telemetry Contract
On every top-level `MiniAgent.ask()` call (`depth == 0`), print a `[SKILL TELEMETRY]` block to `stdout` containing three exact fields:
1. `User Input Prompt`: Raw prompt string passed to `ask()`.
2. `Similarity Scores`: Comma-separated `<skill_name>=<score:.4f>` pairs for all discovered skills (or `(no skills discovered)`).
3. `Triggered Skill`: `<skill_name> (via similarity)` if matched automatically, `<skill_name> (via use_skill_tool)` if activated mid-loop via tool call, or `None`.
