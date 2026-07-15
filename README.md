# Claude Code Skills

Collection of production-ready Claude Code skills for AI workflow optimization.

## Skills

### 🎯 [Multi-Tier Models](./multi-tier-models/)

**Cost-optimized hierarchical model usage with 40-83% cost reduction**

Strategic framework combining:
- **Fable 5** (strategic advisor) for high-level reasoning
- **Opus 4.8** (orchestrator) for planning and synthesis
- **Haiku 4.5** (workers) for text/code execution
- **Gemini 3.5 Flash** (workers) for multimodal analysis

**Real Results:**
- 83% cost savings ($1.89 → $0.32)
- 75% time reduction (8min → 2min)
- 64% token reduction (63K → 22.7K)
- Equivalent quality to single-model approach

[View Documentation](./multi-tier-models/SKILL.md)

## Installation

### Option 1: Install to Claude Code

```bash
# Clone this repository
git clone https://github.com/upasana1105/claude-code-skills.git

# Install a skill (copy to your skills directory)
cp -r claude-code-skills/multi-tier-models ~/.claude/skills/

# Use in Claude Code
/multi-tier-models
```

### Option 2: Manual Setup

1. Navigate to `~/.claude/skills/`
2. Create a directory for the skill (e.g., `multi-tier-models`)
3. Copy the skill files from this repository
4. Invoke via `/skill-name` in Claude Code

## Skills Overview

| Skill | Purpose | Savings | Status |
|-------|---------|---------|--------|
| [multi-tier-models](./multi-tier-models/) | Hierarchical model usage with context optimization | 40-83% cost, 75% time | ✅ Active |

## About

Built by [Upasana Pati](https://github.com/upasana1105) for production AI workflows.

## License

MIT License - See individual skill directories for details.
