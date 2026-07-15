# Multi-Tier Models Skill

**Cost-optimized hierarchical model usage with 40-83% cost reduction**

## Overview

Strategic framework for complex AI tasks that combines:
- **Fable 5** (strategic advisor) - high-level reasoning  
- **Opus 4.8** (orchestrator) - planning and synthesis
- **Haiku 4.5** (workers) - text/code execution
- **Gemini 3.5 Flash** (workers) - multimodal analysis

Based on Anthropic's "plan big, execute small" cookbook pattern.

## Real Results

Competitive analysis of 8 companies with screenshots:
- **83% cost savings** ($1.89 → $0.32)
- **75% time reduction** (8min → 2min)
- **64% token reduction** (63K → 22.7K)
- Equivalent quality to single-model approach

## Quick Start

```bash
# Install to Claude Code
cp -r multi-tier-models ~/.claude/skills/

# Use the skill
/multi-tier-models
```

## Core Pattern

```
1. Strategy (optional): Fable 5 for ambiguous decisions
2. Planning: Opus extracts minimal context per step
3. Execution: Parallel Haiku/Gemini workers
4. Synthesis: Opus combines results
```

## Files

- **[SKILL.md](./SKILL.md)** - Complete skill documentation
- **references/** - Implementation patterns and examples
  - `complete-example.md` - Full workflow (83% savings)
  - `context-optimization.md` - Token reduction strategies  
  - `gemini-workers.md` - Gemini integration
  - `quick-reference.md` - Decision flow cheat sheet
  - `workflow-example.md` - Workflow script patterns
  - `agent-pattern.md` - Agent tool usage

## Key Optimizations

✅ **Context compaction**: Extract only what each step needs (60-80% token savings)  
✅ **Structured outputs**: JSON schemas prevent verbose responses (40-60% savings)  
✅ **Right model for right task**: Gemini for multimodal, Haiku for text  
✅ **Parallel execution**: All workers spawn simultaneously (75% time savings)

## When to Use

- Research with multiple sources
- Content generation with multiple sections
- Multimodal analysis (documents with images/charts)
- Code analysis across modules

## When NOT to Use

- Single-step tasks (overhead > benefit)
- Highly iterative/interactive work
- Tasks requiring top-tier reasoning throughout

## License

MIT
