# Multi-Tier Models Quick Reference

## Model Selection Cheat Sheet

```
┌─────────────────────────────────────────────────────────┐
│ TASK TYPE              → USE MODEL                      │
├─────────────────────────────────────────────────────────┤
│ Strategy/Complex       → Fable 5                        │
│ Planning/Orchestration → Opus 4.8                       │
│ Text analysis/code     → Haiku 4.5                      │
│ Image + text analysis  → Gemini 3.5 Flash              │
│ Video generation       → Gemini Omni Flash Preview     │
│ Image generation       → Gemini 3.1 Flash Image        │
│ Synthesis/Quality      → Opus 4.8                       │
└─────────────────────────────────────────────────────────┘
```

## Decision Flow

```
1. Need strategic insight?
   YES → Spawn Fable 5 for strategy (+ extract essentials)
   NO  → Skip to step 2

2. Spawn Opus 4.8 for planning
   (Always - creates worker tasks)
   ⚠️ CRITICAL: Extract MINIMAL context for each step
   
3. For each task in plan:
   Text/code?           → Haiku
   Doc + images?        → Gemini 3.5 Flash
   Need video?          → Gemini Omni Flash Preview
   Need image?          → Gemini 3.1 Flash Image
   
4. Spawn ALL workers in parallel
   (Single message, multiple Agent calls)
   ⚠️ Use JSON schemas for structured outputs
   
5. Synthesis with Opus or current model
   (Receives compact JSON, not prose)
```

## Cost Tiers

```
$$$$  Fable 5              → Strategic reasoning only
$$$   Opus 4.8             → Planning + synthesis
$     Haiku 4.5            → Text/code workers
¢     Gemini 3.5 Flash     → Multimodal workers
¢     Gemini Omni Flash    → Video workers
¢     Gemini 3.1 Flash Img → Image workers
```

## Common Patterns

### Research Pattern
```
Fable (optional) → Opus plan → Haiku workers → Opus synthesis
                                └─ Gemini 3.5 Flash (if docs have images)
```

### Content + Visuals Pattern
```
Fable (optional) → Opus plan → Haiku (text) → Opus synthesis
                                ├─ Gemini 3.1 Flash Image (images)
                                └─ Gemini Omni Flash (videos)
```

### Code + Architecture Pattern
```
Fable → Opus plan → Haiku workers → Opus synthesis
(All Haiku - no Gemini needed)
```

### Multimodal Analysis Pattern
```
(Skip Fable) → Opus plan → Gemini 3.5 Flash workers → Opus synthesis
```

## API Usage

### Agent Tool (Preferred)
```javascript
Agent({
  model: "fable" | "opus" | "haiku" | 
         "gemini-3.5-flash" | "gemini-omni-flash-preview" | 
         "gemini-3.1-flash-image",
  prompt: "...",
  description: "..."
})
```

### Parallel Workers (Critical!)
```javascript
// ✅ GOOD - Parallel (single message)
Agent({model: "haiku", prompt: "task 1"})
Agent({model: "gemini-3.5-flash", prompt: "task 2"})
Agent({model: "haiku", prompt: "task 3"})

// ❌ BAD - Sequential (slow!)
await Agent({model: "haiku", prompt: "task 1"})
await Agent({model: "haiku", prompt: "task 2"})
await Agent({model: "haiku", prompt: "task 3"})
```

## Typical Cost Savings

| Scenario | Single Model | Multi-Tier | Savings |
|----------|--------------|------------|---------|
| Text research (10 steps) | Fable: $0.60 | Opus+Haiku: $0.10 | 83% |
| Multimodal analysis (8 docs) | Sonnet: $0.45 | Opus+Gemini: $0.11 | 76% |
| Video generation (3 videos) | External: $1.50 | Fable+Opus+Gemini: $0.39 | 74% |
| Mixed content + images | Fable: $0.80 | Full multi-tier: $0.18 | 77% |

## When NOT to Use

- ❌ Single-step tasks (overhead > benefit)
- ❌ Highly iterative work (adds latency)
- ❌ Need Fable 5 for everything (no cost benefit)
- ❌ <3 parallel tasks (planning overhead not amortized)

## Gotchas

1. **Parallelism**: Always spawn workers in single message
2. **Context bloat**: Extract minimal context in planning phase (biggest cost saver!)
3. **Worker selection**: Match model to task type (don't use Gemini for pure text)
4. **Unstructured outputs**: Use JSON schemas to force compact results
5. **Synthesis**: Use Opus for quality, not Haiku
6. **Strategy**: Only use Fable when truly ambiguous/novel
7. **Global endpoints**: Gemini models require proper endpoint configuration

## Example Invocation

```
User: "Analyze 5 competitor websites and create comparison with screenshots"

Multi-tier approach:
1. Opus: Plan (identify comparison dimensions, websites to analyze)
2. Workers (parallel):
   - 5x Gemini 3.5 Flash: Analyze each website + screenshots
3. Opus: Synthesize into comparison table

Cost: ~$0.12 vs $0.50 with single Fable call
Time: ~2min vs ~8min sequential
```

## Files to Read

- `context-optimization.md` - **START HERE** - Context compaction strategies (60-80% savings)
- `quick-reference.md` - This file - cheat sheet
- `workflow-example.md` - Complete workflow script examples
- `agent-pattern.md` - Agent tool patterns (non-workflow)
- `gemini-workers.md` - Gemini integration details
- `SKILL.md` - Full skill documentation
