# Complete Example: Optimized Multi-Tier Workflow

**Task**: Research 8 competitor products with screenshots, generate comparison report with visual chart.

This example demonstrates:
- ✅ Multi-tier model usage (Fable → Opus → Haiku/Gemini)
- ✅ Context optimization (minimal contexts, structured outputs)
- ✅ Parallel execution
- ✅ Gemini integration for multimodal + image generation

## Workflow Implementation

```javascript
export const meta = {
  name: 'competitor-analysis-optimized',
  description: 'Multi-tier competitor analysis with context optimization',
  phases: [
    {title: 'Strategy', model: 'fable'},
    {title: 'Plan', model: 'opus'},
    {title: 'Research', model: 'gemini-3.5-flash'},
    {title: 'Synthesize', model: 'opus'},
    {title: 'Visualize', model: 'gemini-3.1-flash-image'}
  ]
}

// Input: {competitors: ['Product A', 'Product B', ...], ourProduct: 'Our Product'}

// ============================================================================
// PHASE 1: Strategy (Fable 5)
// Large context, but extract essentials for downstream
// ============================================================================
phase('Strategy')

const STRATEGY_SCHEMA = {
  type: 'object',
  properties: {
    approach: {
      type: 'string',
      maxLength: 500  // Keep concise
    },
    dimensions: {
      type: 'array',
      items: {type: 'string'},
      maxItems: 6  // Limit scope
    },
    // CRITICAL: Extract minimal context for workers
    workerContext: {
      type: 'string',
      maxLength: 300,
      description: 'Essential background each worker needs (max 300 chars)'
    }
  },
  required: ['approach', 'dimensions', 'workerContext']
}

const strategy = await agent(`
You are a strategic advisor analyzing competitive landscape.

[FULL CONTEXT - only seen by Fable]
Our product: ${args.ourProduct}
Our goals: ${args.goals}
Target market: ${args.market}
Constraints: ${args.constraints}

Competitors to analyze: ${args.competitors.join(', ')}

Create analysis strategy:
1. What dimensions to compare? (pricing, features, UX, positioning, etc.)
2. What approach for comparison?
3. Extract MINIMAL context workers need (max 300 chars - just essentials)

Be concise. This guides research but doesn't execute it.
`, {
  model: 'fable',
  effort: 'high',
  schema: STRATEGY_SCHEMA
})

log(`Strategy: ${strategy.dimensions.join(', ')}`)

// ============================================================================
// PHASE 2: Planning (Opus 4.8)
// Create compact plan with minimal context per step
// ============================================================================
phase('Plan')

const PLAN_SCHEMA = {
  type: 'object',
  properties: {
    steps: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          id: {type: 'string'},
          competitor: {type: 'string'},
          task: {type: 'string', maxLength: 200},
          // CRITICAL: Only what THIS step needs
          minimalContext: {type: 'string', maxLength: 200},
          // What to extract
          extractFields: {
            type: 'array',
            items: {type: 'string'}
          }
        },
        required: ['id', 'competitor', 'task', 'minimalContext', 'extractFields']
      }
    }
  },
  required: ['steps']
}

const plan = await agent(`
Strategy: ${JSON.stringify(strategy)}
Competitors: ${args.competitors}

Create research plan. For EACH competitor, create ONE research step.

For each step:
- Task: What to research (concise)
- Minimal context: ONLY what this step needs (max 200 chars)
  → Use: "${strategy.workerContext}" + competitor-specific details
  → DON'T repeat full strategy or all competitors
- Extract fields: Specific data to extract (${strategy.dimensions.join(', ')})

Keep contexts MINIMAL - workers will analyze screenshots + web content.
`, {
  model: 'opus',
  schema: PLAN_SCHEMA
})

log(`Created ${plan.steps.length} research steps`)

// ============================================================================
// PHASE 3: Research (Gemini 3.5 Flash)
// Multimodal workers with minimal context, structured output
// ============================================================================
phase('Research')

const RESEARCH_SCHEMA = {
  type: 'object',
  properties: {
    competitor: {type: 'string'},
    findings: {
      type: 'object',
      // Dynamic properties based on dimensions
      additionalProperties: {type: 'string', maxLength: 300}
    },
    metrics: {
      type: 'object',
      properties: {
        price_range: {type: 'string'},
        feature_count: {type: 'number'},
        ux_score: {type: 'number', minimum: 1, maximum: 10}
      }
    },
    screenshot_insights: {
      type: 'array',
      items: {type: 'string'},
      maxItems: 5  // Concise insights only
    }
  },
  required: ['competitor', 'findings', 'metrics']
}

// PARALLEL execution with COMPACT contexts
const results = await parallel(plan.steps.map(step => () =>
  agent(`
Background: ${step.minimalContext}

Task: Research ${step.competitor}

Analyze:
1. Product website + screenshots
2. ${step.extractFields.join(', ')}

Extract specific data for comparison. Be factual and concise.
`, {
    model: 'gemini-3.5-flash',  // Multimodal for screenshots
    label: `research-${step.competitor.toLowerCase().replace(/\s+/g, '-')}`,
    schema: RESEARCH_SCHEMA
  })
))

// Filter out any failures
const validResults = results.filter(Boolean)
log(`Completed ${validResults.length}/${plan.steps.length} analyses`)

// ============================================================================
// PHASE 4: Synthesis (Opus 4.8)
// Compact JSON inputs, structured output
// ============================================================================
phase('Synthesize')

const REPORT_SCHEMA = {
  type: 'object',
  properties: {
    executive_summary: {type: 'string', maxLength: 500},
    comparison_table: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          competitor: {type: 'string'},
          ...Object.fromEntries(
            strategy.dimensions.map(d => [d, {type: 'string'}])
          )
        }
      }
    },
    key_insights: {
      type: 'array',
      items: {type: 'string'},
      maxItems: 8
    },
    recommendations: {
      type: 'array',
      items: {type: 'string'},
      maxItems: 5
    },
    chart_prompt: {
      type: 'string',
      maxLength: 500,
      description: 'Prompt for image generation model to create comparison chart'
    }
  },
  required: ['executive_summary', 'comparison_table', 'key_insights', 'chart_prompt']
}

const report = await agent(`
Strategy: ${strategy.approach}

Research findings (structured data):
${JSON.stringify(validResults, null, 2)}

Synthesize competitive analysis report:
1. Executive summary
2. Comparison table across: ${strategy.dimensions.join(', ')}
3. Key insights (what stands out?)
4. Strategic recommendations
5. Chart prompt (for visual comparison generation)

Output structured data for final report.
`, {
  model: 'opus',
  schema: REPORT_SCHEMA
})

// ============================================================================
// PHASE 5: Visualization (Gemini 3.1 Flash Image)
// Generate comparison chart
// ============================================================================
phase('Visualize')

const chart = await agent(`
${report.chart_prompt}

Style: Clean, professional comparison chart
Format: PNG, high resolution
Include: All competitors, key metrics, legend
`, {
  model: 'gemini-3.1-flash-image',
  label: 'comparison-chart'
})

// ============================================================================
// RETURN
// ============================================================================

return {
  report: {
    summary: report.executive_summary,
    table: report.comparison_table,
    insights: report.key_insights,
    recommendations: report.recommendations
  },
  chart: chart,
  metadata: {
    competitors_analyzed: validResults.length,
    dimensions: strategy.dimensions,
    timestamp: new Date().toISOString()
  }
}
```

## Token Usage Breakdown

### Without Optimization (Single Model)
```
Fable 5 for everything:
- Initial context: 5K tokens
- 8 research calls × 6K tokens = 48K
- Synthesis: 10K tokens
Total: ~63K tokens
Cost: ~$1.89
```

### With Optimization (Multi-Tier + Context Compaction)
```
Phase 1 - Strategy (Fable):
  Input: 5K tokens (full context)
  Output: 500 tokens (compact strategy)
  Cost: $0.165

Phase 2 - Planning (Opus):
  Input: 1K tokens (compact strategy + task)
  Output: 1.5K tokens (compact plan)
  Cost: $0.038

Phase 3 - Research (8× Gemini 3.5 Flash):
  Input: 8 × 500 tokens = 4K (minimal context each)
  Output: 8 × 800 tokens = 6.4K (structured)
  Cost: $0.052

Phase 4 - Synthesis (Opus):
  Input: 2K tokens (compact JSON)
  Output: 1.5K tokens (structured report)
  Cost: $0.053

Phase 5 - Visualization (Gemini 3.1 Flash Image):
  Input: 300 tokens (chart prompt)
  Output: Image
  Cost: $0.015

Total: ~22.7K tokens
Cost: ~$0.323 (83% savings!)
Time: ~2 min (parallel execution)
```

## Key Optimizations Applied

### 1. Context Compaction
- ✅ Fable extracts `workerContext` (300 chars) instead of full 5K context
- ✅ Opus creates minimal context per step (200 chars)
- ✅ Each worker gets ~500 tokens vs ~6K in naive approach
- **Savings**: 80% on worker inputs

### 2. Structured Outputs
- ✅ All phases use JSON schemas
- ✅ No verbose prose in intermediate outputs
- ✅ maxLength/maxItems limits enforce brevity
- **Savings**: 60% on outputs

### 3. Model Selection
- ✅ Gemini 3.5 Flash for multimodal (10× cheaper than Claude)
- ✅ Gemini 3.1 Flash Image for visualization (specialized)
- ✅ Fable only for strategy, Opus for planning/synthesis
- **Savings**: 70% vs all-Fable approach

### 4. Parallel Execution
- ✅ 8 research workers spawn in single message
- ✅ ~2 min total vs ~16 min sequential
- **Time savings**: 87%

## Running This Example

```bash
# Via Workflow tool
Workflow({
  name: 'competitor-analysis-optimized',
  args: {
    competitors: ['Product A', 'Product B', 'Product C', 'Product D', 
                  'Product E', 'Product F', 'Product G', 'Product H'],
    ourProduct: 'Our SaaS Platform',
    goals: 'Identify pricing gaps and feature opportunities',
    market: 'B2B SaaS, mid-market',
    constraints: 'Focus on products with public pricing'
  }
})
```

## Comparison Matrix

| Aspect | Single Model | Multi-Tier (No Opt) | Multi-Tier (Optimized) |
|--------|--------------|---------------------|------------------------|
| **Tokens** | 63K | 45K | 22.7K |
| **Cost** | $1.89 | $0.85 | $0.32 |
| **Time** | 8 min | 8 min | 2 min |
| **Quality** | High | High | High |
| **Complexity** | Low | Medium | Medium |

## Lessons

1. **Context is the biggest lever**: 80% savings from compaction
2. **Schemas are critical**: Force structured, compact outputs
3. **Right model for right job**: Gemini Flash for multimodal = 10× cheaper
4. **Parallelism**: 4× speed improvement
5. **Quality maintained**: Multi-tier matches single Fable quality

## Next Steps

- Try this pattern on your own tasks
- Monitor token usage per phase
- Adjust schema limits based on actual needs
- Experiment with different model combinations
- Consider prompt caching for repeated analyses
