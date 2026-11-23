# Implementation Plan: Dynamic Agent Skills & Telemetry Logging

## Phase 1: Skill Discovery & Scoring (`SkillManager`)
1. Implement `SkillManager` in `mini_coding_agent.py` initialized with `repo_root` and `threshold`.
2. Add `discover()` using `pathlib.Path` to scan `skills/*/SKILL.md` and `_parse_skill_md()` to extract frontmatter metadata and body text.
3. Implement `_tokenize()`, `score_skill()`, and `match()` to rank candidate skills against user prompts.
4. Add `log_run()` to print formatted `[SKILL TELEMETRY]` output to console.

## Phase 2: Agent & Tool Integration (`MiniAgent`)
1. Instantiate `self.skill_manager` and `self.active_skill = None` in `MiniAgent.__init__()`; clear `self.active_skill` in `reset()`.
2. Register `use_skill` in `build_tools()`, `validate_tool()`, `tool_example()`, and `parse_xml_tool()`, backed by `tool_use_skill()`.
3. Update `build_prefix()` to advertise compact skill summaries and usage rules, and update `prompt()` to inject `self.active_skill["body"]` when active.
4. Update `ask()` to refresh discovery, run similarity matching, log initial telemetry, and re-log if `use_skill` activates a skill during execution.

## Phase 3: CLI Configuration & Verification
1. Expose `--skill-threshold` in `build_arg_parser()` and wire it through `build_agent()`.
2. Create sample skill playbooks (e.g., `skills/pr-review/SKILL.md` and `skills/q-learning-debug/SKILL.md`).
3. Verify via automated tests covering cross-platform path resolution, threshold triggering, `use_skill` fallback activation, and console telemetry formatting.
