# Complete Workflow Example

## Scenario: Competitive Analysis Research

**Task**: Research 4 competitors across pricing, features, and market positioning.

### Workflow Script

```javascript
export const meta = {
  name: 'competitive-analysis',
  description: 'Multi-tier competitive research: Fable strategizes, Opus plans, Haiku executes',
  phases: [
    {title: 'Strategy', model: 'fable'},
    {title: 'Plan', model: 'opus'}, 
    {title: 'Research', model: 'haiku'},
    {title: 'Synthesize'}
  ]
}

// Schema definitions
const STRATEGY_SCHEMA = {
  type: 'object',
  properties: {
    dimensions: {type: 'array', items: {type: 'string'}},
    approach: {type: 'string'},
    pitfalls: {type: 'array', items: {type: 'string'}}
  },
  required: ['dimensions', 'approach']
}

const PLAN_SCHEMA = {
  type: 'object',
  properties: {
    steps: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          id: {type: 'string'},
          task: {type: 'string'},
          context: {type: 'string'},
          outputFormat: {type: 'string'}
        }
      }
    }
  },
  required: ['steps']
}

// Phase 1: Fable 5 Strategy (high-level reasoning)
phase('Strategy')
const strategy = await agent(`
You are a strategic advisor. Analyze this competitive research task:

Competitors: ${args.competitors.join(', ')}
Our product: ${args.ourProduct}

Provide:
1. Key dimensions to research (pricing, features, positioning, etc.)
2. Recommended approach for comparison
3. Potential pitfalls to avoid

Be concise - this guides the research, doesn't execute it.
`, {
  model: 'fable',
  schema: STRATEGY_SCHEMA,
  effort: 'high'
})

log(`Strategy: Focus on ${strategy.dimensions.join(', ')}`)

// Phase 2: Opus 4.8 Planning (detailed breakdown)
phase('Plan')
const plan = await agent(`
Create detailed research plan based on this strategy:
${JSON.stringify(strategy, null, 2)}

For each competitor (${args.competitors.join(', ')}), create research steps covering:
${strategy.dimensions.join(', ')}

Each step must:
- Be independently executable
- Include specific research questions
- Specify output format (bullet points, table, etc.)
- Take <5 minutes to research

Output a list of discrete research tasks.
`, {
  model: 'opus',
  schema: PLAN_SCHEMA
})

log(`Created ${plan.steps.length} research steps`)

// Phase 3: Haiku Workers (parallel execution)
phase('Research')
const results = await pipeline(
  plan.steps,
  step => agent(`
Research task: ${step.task}

Context: ${step.context}
Output format: ${step.outputFormat}

Use web search to find current information. Be factual and cite sources.
`, {
    model: 'haiku',
    label: `research-${step.id}`
  })
)

// Phase 4: Synthesis (combine results)
phase('Synthesize')
const report = await agent(`
Synthesize competitive analysis from research results:

Strategy: ${JSON.stringify(strategy)}
Research findings: ${JSON.stringify(results, null, 2)}

Create final competitive analysis report:
1. Executive summary
2. Detailed comparison across ${strategy.dimensions.join(', ')}
3. Key insights and recommendations
4. Gaps/areas needing more research

Format as markdown with tables where appropriate.
`, {
  model: 'opus' // Use Opus for quality synthesis
})

return {
  strategy,
  report,
  metadata: {
    stepsExecuted: plan.steps.length,
    dimensions: strategy.dimensions
  }
}
```

## Cost Analysis

Assuming 4 competitors, 3 dimensions each (pricing, features, positioning):

| Phase | Model | Calls | Avg Tokens | Cost (approx) |
|-------|-------|-------|------------|---------------|
| Strategy | Fable 5 | 1 | 1,500 | $0.045 |
| Planning | Opus 4.8 | 1 | 3,000 | $0.045 |
| Research | Haiku 4.5 | 12 | 1,500 ea | $0.024 |
| Synthesis | Opus 4.8 | 1 | 4,000 | $0.060 |
| **Total** | - | 15 | ~25K | **$0.174** |

Compare to single Fable 5 approach (one long context): ~$0.30-0.50

**Savings: ~40-65%** while maintaining Fable 5 strategic quality.

## Alternative: Agent Tool Pattern

If not using Workflow, spawn agents directly:

```
1. Agent({model: 'fable', prompt: '[strategy prompt]'})
   → Get strategic framework

2. Agent({model: 'opus', prompt: '[planning prompt with strategy]'})  
   → Get execution plan

3. In single message:
   Agent({model: 'haiku', prompt: '[step 1]'})
   Agent({model: 'haiku', prompt: '[step 2]'})
   Agent({model: 'haiku', prompt: '[step 3]'})
   ... (all workers)
   → Execute in parallel

4. Synthesize results with current model or Opus
```

## When This Pattern Doesn't Help

- **Simple tasks**: Overhead > savings
- **Iterative work**: Multi-tier adds latency
- **Requires Fable 5 throughout**: No cost benefit
- **Few parallel steps**: Can't amortize planning cost

Best for: Research, analysis, batch processing, multi-dimensional comparisons.
