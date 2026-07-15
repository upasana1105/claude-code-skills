# Multi-Tier Models Skill

**Cost-optimized hierarchical model usage: Fable 5 advises, Opus 4.8 orchestrates, Haiku/Gemini execute specialized tasks.**

## When to Use

Invoke this skill when:
- The user wants to "use multiple models", "hierarchical agents", "optimize model costs"
- Complex tasks that benefit from strategic planning + efficient execution
- Budget-conscious workflows requiring Fable 5's capabilities without excessive cost

## Pattern Overview

```
Fable 5 (Strategic Advisor)
    ↓ provides high-level strategy
Opus 4.8 (Orchestrator/Planner)
    ↓ creates detailed execution plan
Workers (parallel execution):
    ├─ Haiku 4.5 → text/code tasks
    ├─ Gemini 3.5 Flash → multimodal analysis
    ├─ Gemini Omni Flash Preview → video generation/editing
    └─ Gemini 3.1 Flash Image → image generation/editing
```

## Implementation

### Step 1: Fable 5 Strategy (if needed)
Use Fable 5 for:
- Ambiguous/open-ended problems requiring reasoning
- Strategic decisions, trade-offs, architectural choices
- Novel problem decomposition

Skip if: Task is well-defined and doesn't need strategic insight.

### Step 2: Opus 4.8 Planning
Always use Opus 4.8 to:
- Break work into discrete, parallelizable steps
- Create detailed execution plan with context for each step
- Determine dependencies and ordering

### Step 3: Worker Execution
Spawn specialized workers based on task type:

**Haiku 4.5** for:
- Text analysis, summarization, extraction
- Code generation, review, refactoring
- General-purpose concrete tasks

**Gemini 3.5 Flash** for:
- Multimodal analysis (images + text)
- Document understanding with visuals
- Cross-modal reasoning

**Gemini Omni Flash Preview** for:
- Video generation from text
- Video editing and transformation
- Video analysis and summarization

**Gemini 3.1 Flash Image** for:
- Image generation from text
- Image editing and manipulation
- Visual content creation

All workers execute in parallel where possible.

### Step 4: Synthesis
Use Opus 4.8 (or current model) to:
- Collect worker results
- Synthesize final output
- Handle any conflicts/inconsistencies

## Usage Example

```
User: "Research competitive landscape for our new product"

1. [Optional] Fable 5 strategy:
   - What dimensions matter? (pricing, features, market position, etc.)
   - How to structure the research?
   
2. Opus 4.8 plan:
   - Step 1: Research competitor A's pricing
   - Step 2: Research competitor B's features
   - Step 3: Analyze market positioning
   - Step 4: Summarize customer reviews
   (Each step includes context, search queries, output format)

3. Haiku workers (parallel):
   - 4 agents, each handling one step
   
4. Synthesis:
   - Combine into competitive analysis report
```

## Cost Optimization Tips

### Model Selection
- **Use Fable 5 sparingly**: Only for strategy, not execution
- **Choose the right worker**: Gemini Flash for multimodal/video/image, Haiku for text/code
- **Leverage global endpoints**: Gemini models use optimized global infrastructure

### Context Optimization (Critical!)
- **Compact contexts**: Extract minimal context in planning; workers get only what they need
- **Structured outputs**: Use JSON schemas to enforce compact, structured results
- **Avoid duplication**: Use reference IDs instead of embedding large content
- **Summarize between stages**: Compress intermediate results in multi-stage pipelines
- **Enable prompt caching**: Reuse expensive context across calls (Claude models)

### Execution Efficiency
- **Batch worker execution**: Spawn all workers in parallel, not sequentially
- **Reuse plans**: For similar tasks, skip Fable 5, reuse Opus plans

**See `references/context-optimization.md` for detailed strategies (60-80% token savings)**

## Model Selection Guide

| Model | Use For | Cost | Typical Token Usage |
|-------|---------|------|---------------------|
| **Claude Models** | | | |
| Fable 5 | Strategy, complex reasoning | Highest | 1-2K tokens |
| Opus 4.8 | Planning, orchestration, synthesis | Medium | 2-5K tokens |
| Haiku 4.5 | Text/code execution tasks | Low | 500-2K tokens each |
| **Gemini Models (Global Endpoints)** | | | |
| Gemini 3.5 Flash | Multimodal analysis, doc+image | Very Low | 1-3K tokens |
| Gemini Omni Flash Preview | Video generation/editing | Very Low | Varies by video |
| Gemini 3.1 Flash Image | Image generation/editing | Very Low | Varies by image |

**Cost optimization**: Use Gemini Flash models for multimodal/video/image tasks - optimized for speed and cost on global infrastructure.

## Implementation

When this skill is invoked:

1. **Assess if Fable 5 needed**:
   - Is the problem ambiguous or novel?
   - Are there strategic trade-offs to consider?
   - Would reasoning significantly improve the approach?

2. **Use Agent tool to spawn Fable 5** (if needed):
   ```
   Agent({
     description: "Strategic guidance for X",
     model: "fable",
     prompt: "Provide strategic framework for: [task]. Focus on: approach, key dimensions, potential pitfalls."
   })
   ```

3. **Create detailed plan with Opus 4.8** (with context extraction):
   ```
   Agent({
     description: "Create execution plan for X",
     model: "opus",
     prompt: `Based on strategy: [fable_output], create detailed step-by-step plan.
     
     For each step:
     - Task description (what to do)
     - Minimal context (ONLY what this step needs - max 200 words)
     - Expected output format (structured schema)
     
     Extract essentials; avoid duplicating full context to each step.`
   })
   ```

4. **Spawn specialized workers in parallel**:
   ```
   // In single message, spawn all workers (choose model by task type)
   Agent({ model: "haiku", prompt: "Text analysis step: [details]" })
   Agent({ model: "gemini-3.5-flash", prompt: "Multimodal step (doc+images): [details]" })
   Agent({ model: "gemini-3.1-flash-image", prompt: "Generate image: [details]" })
   Agent({ model: "haiku", prompt: "Code generation step: [details]" })
   ```
   
   **Worker selection**:
   - `haiku`: Text, code, general tasks
   - `gemini-3.5-flash`: Multimodal (images + text)
   - `gemini-omni-flash-preview`: Video generation/editing
   - `gemini-3.1-flash-image`: Image generation/editing

5. **Synthesize results**:
   - Collect all worker outputs
   - Combine, resolve conflicts
   - Present final result to user

## Workflow Script Alternative

For complex multi-step workflows, consider using the Workflow tool instead of Agent:

```javascript
export const meta = {
  name: 'multi-tier-research',
  description: 'Research using Fable 5 strategy, Opus planning, Haiku execution',
  phases: [
    {title: 'Strategy', model: 'fable'},
    {title: 'Planning', model: 'opus'},
    {title: 'Execute', model: 'haiku'},
    {title: 'Synthesize'}
  ]
}

phase('Strategy')
const strategy = await agent('Provide strategic framework for: ' + args.task, {
  model: 'fable',
  schema: STRATEGY_SCHEMA
})

phase('Planning')
const plan = await agent('Create execution plan based on: ' + JSON.stringify(strategy), {
  model: 'opus',
  schema: PLAN_SCHEMA
})

phase('Execute')
const results = await parallel(plan.steps.map(step => () =>
  agent(step.prompt, {model: 'haiku', schema: step.outputSchema})
))

phase('Synthesize')
return synthesize(results, strategy)
```

## Reference Files

- **`references/complete-example.md`** - **START HERE** - Full optimized example (83% cost savings)
- **`references/quick-reference.md`** - Cheat sheet and decision flow
- **`references/context-optimization.md`** - Context compaction strategies (60-80% token savings)
- **`references/gemini-workers.md`** - Gemini integration patterns
- **`references/workflow-example.md`** - Workflow script patterns
- **`references/agent-pattern.md`** - Agent tool usage patterns

## Notes

- This pattern mirrors the "plan big, execute small" from Claude Cookbooks
- **Context optimization is critical**: Can save 60-80% tokens through compaction
- Works best for research, analysis, multi-step generation, multimodal tasks
- Not suitable for: single-step tasks, interactive workflows, iterative refinement
- Monitor costs: Check token usage to ensure savings vs. single-model approach
- Gemini models use global endpoints for optimal performance worldwide
