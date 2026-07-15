# Agent Tool Pattern (Non-Workflow)

Quick reference for using multi-tier models with the Agent tool directly.

## Basic Pattern

```javascript
// 1. Strategy (Fable 5) - Optional, only if needed
const strategy = await Agent({
  description: "Strategic guidance for X",
  model: "fable",
  prompt: `Strategic question: [problem]
  
  Provide: approach, key considerations, potential pitfalls.
  Keep concise - this guides planning, not execution.`
})

// 2. Planning (Opus 4.8) - Always
const plan = await Agent({
  description: "Create execution plan",
  model: "opus", 
  prompt: `Task: [task]
  ${strategy ? `Strategy: ${strategy}` : ''}
  
  Create step-by-step execution plan. Each step:
  - Independent and parallelizable
  - Well-scoped (<5 min work)
  - Includes context and expected output
  
  Output as JSON array of steps.`
})

// 3. Execute (Haiku 4.5) - Parallel workers
// IMPORTANT: Send all Agent calls in SINGLE message for parallelism
const results = [
  Agent({model: "haiku", prompt: `Execute: ${plan.steps[0]}`}),
  Agent({model: "haiku", prompt: `Execute: ${plan.steps[1]}`}),
  Agent({model: "haiku", prompt: `Execute: ${plan.steps[2]}`}),
  // ... all steps
]

// 4. Synthesize - Current model or Opus
const final = synthesizeResults(results)
```

## Decision Tree: Use Fable 5?

```
Is the task ambiguous or novel?
├─ YES → Does it require strategic reasoning?
│         ├─ YES → Use Fable 5 for strategy
│         └─ NO → Skip to Opus planning
└─ NO → Skip to Opus planning

Examples where Fable 5 adds value:
- "How should we approach refactoring this module?"
- "What's the best way to migrate users to new API?"
- "Design a testing strategy for this feature"

Examples where Fable 5 is overkill:
- "Research pricing for these 5 competitors"
- "Summarize these 10 blog posts"  
- "Extract data from these files"
```

## Parallel Execution Critical

❌ **Bad** (sequential - slow & expensive):
```javascript
const r1 = await Agent({model: "haiku", prompt: "step 1"})
const r2 = await Agent({model: "haiku", prompt: "step 2"})
const r3 = await Agent({model: "haiku", prompt: "step 3"})
```

✅ **Good** (parallel - fast & efficient):
```javascript
// Send in SINGLE message:
Agent({model: "haiku", prompt: "step 1"})
Agent({model: "haiku", prompt: "step 2"})  
Agent({model: "haiku", prompt: "step 3"})
```

## Cost Comparison Example

**Task**: Summarize 8 technical blog posts

### Approach A: Single Fable 5 call
- 1 call @ ~20K tokens = $0.60

### Approach B: Multi-tier
- Opus plan (1 call, 2K tokens) = $0.030
- 8 Haiku workers (1.5K tokens each) = $0.024  
- Opus synthesis (1 call, 3K tokens) = $0.045
- **Total = $0.099** (83% savings)

### Approach C: All Haiku (cheapest but lower quality)
- Haiku plan (1 call) = $0.003
- 8 Haiku workers = $0.024
- Haiku synthesis = $0.005
- **Total = $0.032** (95% savings, but loses strategic quality)

**Choose**: B for quality+savings, C for maximum cost cutting.

## Common Patterns

### Research Pattern
```
Fable: Research framework (optional)
Opus: Create search queries for each dimension  
Haiku: Execute searches (parallel)
Opus: Synthesize findings
```

### Content Generation Pattern
```
Fable: Content strategy (optional)
Opus: Outline + assign sections
Haiku: Write each section (parallel)
Opus: Edit & unify voice
```

### Data Processing Pattern
```
(Skip Fable - well-defined task)
Opus: Create processing steps
Haiku: Process each file/chunk (parallel)
Opus: Combine & validate results
```

### Code Analysis Pattern  
```
Fable: Analysis approach (if complex)
Opus: Break into modules/aspects to analyze
Haiku: Analyze each module (parallel)
Opus: Identify patterns & recommendations
```

## Antipatterns

🚫 **Using Fable 5 for execution**: Expensive, no quality gain
🚫 **Sequential Haiku calls**: Defeats parallelism benefit
🚫 **Multi-tier for simple tasks**: Overhead > savings
🚫 **Haiku for synthesis**: Quality matters here, use Opus
🚫 **Skipping planning**: Workers need context to be effective

## Quick Start Checklist

- [ ] Task benefits from parallelism? (multiple steps/items)
- [ ] Need Fable 5 strategic insight? (ambiguous/novel problem)
- [ ] Create detailed plan with Opus (always)
- [ ] Spawn all Haiku workers in single message (parallel)
- [ ] Synthesize with Opus for quality (or current model)
- [ ] Verify cost savings > planning overhead
