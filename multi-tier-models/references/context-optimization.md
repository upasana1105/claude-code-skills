# Context Optimization Strategies

**Minimize token usage through context compaction, caching, and strategic information passing.**

## Core Principle

```
Planning Phase:  LARGE CONTEXT  (full background, requirements, constraints)
                       ↓
                 Extract essentials
                       ↓
Execution Phase: SMALL CONTEXTS (only what each worker needs)
                       ↓
                 Structured outputs
                       ↓
Synthesis Phase: MEDIUM CONTEXT  (worker results + strategy)
```

## Strategy 1: Context Compaction Between Phases

### Planning → Execution

❌ **Bad** (pass everything):
```javascript
// Opus creates plan
const plan = await agent(`Full context: ${fullRequirements}`, {model: 'opus'})

// Haiku gets EVERYTHING (wasteful)
const result = await agent(`
Full context: ${fullRequirements}
Plan: ${JSON.stringify(plan)}
Your task: ${step1}
`, {model: 'haiku'})
```

✅ **Good** (extract essentials):
```javascript
// Opus creates plan with context extraction
const plan = await agent(`Full context: ${fullRequirements}
Create plan AND extract minimal context for each step.`, {
  model: 'opus',
  schema: {
    steps: [{
      task: 'string',
      minimalContext: 'string',  // ONLY what this step needs
      expectedOutput: 'string'
    }]
  }
})

// Haiku gets ONLY what it needs
const result = await agent(`
Context: ${step.minimalContext}
Task: ${step.task}
Output format: ${step.expectedOutput}
`, {model: 'haiku'})
```

**Token savings**: 60-80% per worker

### Execution → Synthesis

❌ **Bad** (raw outputs):
```javascript
// Workers return verbose responses
const results = await parallel(steps.map(step => () =>
  agent(step.task, {model: 'haiku'})  // Returns full prose
))

// Synthesis gets all verbose outputs
const final = await agent(`Synthesize: ${results.join('\n\n')}`, {model: 'opus'})
```

✅ **Good** (structured outputs):
```javascript
// Workers return structured data
const results = await parallel(steps.map(step => () =>
  agent(step.task, {
    model: 'haiku',
    schema: step.outputSchema  // Forces structured output
  })
))

// Synthesis gets compact JSON
const final = await agent(`Synthesize: ${JSON.stringify(results)}`, {model: 'opus'})
```

**Token savings**: 40-60% on synthesis

## Strategy 2: Prompt Caching (Claude Models)

Use Claude's prompt caching to reuse expensive context:

```javascript
// First call - cache the strategy
const strategy = await agent(`
[Large context that will be reused]
${projectBackground}
${requirements}
${constraints}

Create strategy...
`, {
  model: 'fable',
  // Enable caching (if Agent tool supports it)
})

// Subsequent calls can reference cached context
// NOTE: This requires proper cache control headers
```

**When caching helps**:
- Reusing project context across multiple planning calls
- Long reference documents needed by multiple workers
- Iterative refinement with same base context

**Token savings**: Up to 90% on cache hits

## Strategy 3: Progressive Summarization

For multi-stage pipelines, summarize intermediate results:

```javascript
// Stage 1: Research (verbose)
const rawResearch = await parallel(sources.map(s => () =>
  agent(`Research: ${s}`, {model: 'haiku'})
))

// Compress before next stage
const summary = await agent(`
Summarize these research results into key findings only:
${JSON.stringify(rawResearch)}

Output: Bullet points, max 200 words per source.
`, {model: 'opus'})

// Stage 2: Analysis (uses summary, not raw)
const analysis = await agent(`
Analyze findings: ${summary}
...
`, {model: 'opus'})
```

**Use when**:
- Pipeline has >2 stages
- Intermediate results are verbose
- Later stages don't need full detail

**Token savings**: 50-70% on downstream stages

## Strategy 4: Structured Schemas

Force compact outputs with JSON schemas:

```javascript
const COMPACT_SCHEMA = {
  type: 'object',
  properties: {
    key_findings: {
      type: 'array',
      items: {type: 'string'},
      maxItems: 5  // Limit verbosity
    },
    metrics: {
      type: 'object',
      properties: {
        score: {type: 'number'},
        confidence: {type: 'number'}
      }
    },
    // NO free-text fields
  },
  required: ['key_findings', 'metrics']
}

// Worker forced to be concise
const result = await agent('Analyze...', {
  model: 'haiku',
  schema: COMPACT_SCHEMA
})
```

**Best practices**:
- Use enums instead of free text
- Set maxItems/maxLength limits
- Avoid description/notes fields (verbose)
- Use numbers/booleans over text

**Token savings**: 60-80% vs prose

## Strategy 5: Reference IDs Instead of Duplication

For large artifacts, use references:

❌ **Bad** (embed full content):
```javascript
const plan = {
  steps: [
    {task: 'Analyze doc1', doc: '<full 5KB document>'},
    {task: 'Analyze doc2', doc: '<full 5KB document>'},
    {task: 'Analyze doc3', doc: '<full 5KB document>'}
  ]
}
// 15KB passed to each worker
```

✅ **Good** (use references):
```javascript
// Store documents separately
const docs = {
  'doc1': '<full document>',
  'doc2': '<full document>',
  'doc3': '<full document>'
}

const plan = {
  steps: [
    {task: 'Analyze doc1', docId: 'doc1'},  // Just ID
    {task: 'Analyze doc2', docId: 'doc2'},
    {task: 'Analyze doc3', docId: 'doc3'}
  ]
}

// Each worker gets only its document
await agent(`
Doc: ${docs[step.docId]}
Task: ${step.task}
`, {model: 'haiku'})
```

**Token savings**: 70-90% on plan size

## Strategy 6: Lazy Context Loading

Don't pass context that might not be needed:

```javascript
// Opus decides what context each worker needs
const plan = await agent(`
Create plan for: ${task}

For each step, specify:
1. What it does
2. What context it needs (minimal)
3. Where to get that context (file, API, etc.)
`, {model: 'opus'})

// Workers fetch only what they need
const results = await parallel(plan.steps.map(step => () =>
  agent(`
${step.contextSource ? `Load context from: ${step.contextSource}` : ''}
Task: ${step.task}
`, {model: 'haiku'})
))
```

**Use when**:
- Some workers might not need certain context
- Context is expensive to include
- Workers can fetch context themselves

## Strategy 7: Incremental Results

For long-running tasks, workers return progress incrementally:

```javascript
// Instead of one large final output
const result = await agent('Process all 100 items...', {
  model: 'haiku'
  // Returns after processing all 100 (large output)
})

// Better: Process in chunks, return compact summaries
const results = await pipeline(
  chunks(items, 10),  // 10 chunks of 10 items
  chunk => agent(`Process: ${chunk}`, {
    model: 'haiku',
    schema: COMPACT_SUMMARY_SCHEMA  // Small output per chunk
  })
)
```

**Token savings**: 50-70% on large datasets

## Complete Example: Context-Optimized Pipeline

```javascript
export const meta = {
  name: 'context-optimized-research',
  description: 'Research with aggressive context optimization',
  phases: ['Strategy', 'Plan', 'Research', 'Synthesize']
}

// Phase 1: Strategy (large context, one-time)
phase('Strategy')
const strategy = await agent(`
[FULL PROJECT CONTEXT - will be cached/summarized]
${projectDocs}
${requirements}
${constraints}

Create research strategy.
`, {
  model: 'fable',
  schema: {
    type: 'object',
    properties: {
      approach: {type: 'string', maxLength: 500},  // Compact
      dimensions: {type: 'array', maxItems: 5},
      // Extract ONLY essentials for workers
      workerContext: {
        type: 'string',
        maxLength: 200,  // Minimal context
        description: 'Core background each worker needs'
      }
    }
  }
})

// Phase 2: Planning (uses summarized strategy)
phase('Plan')
const plan = await agent(`
Strategy: ${JSON.stringify(strategy)}

Create research tasks. For each:
- What to research (concise)
- Minimal context needed (max 100 words)
- Output format (structured)
`, {
  model: 'opus',
  schema: {
    steps: [{
      query: 'string',
      context: {type: 'string', maxLength: 200},
      outputSchema: 'object'
    }]
  }
})

// Phase 3: Research (minimal context per worker)
phase('Research')
const results = await parallel(plan.steps.map(step => () =>
  agent(`
Background: ${strategy.workerContext}
Context: ${step.context}
Task: ${step.query}
`, {
    model: 'haiku',
    schema: step.outputSchema  // Structured, compact
  })
))

// Phase 4: Synthesis (compact inputs)
phase('Synthesize')
return await agent(`
Strategy: ${strategy.approach}
Findings: ${JSON.stringify(results)}  // Compact JSON, not prose

Synthesize final report.
`, {model: 'opus'})
```

## Token Usage Comparison

### Without Context Optimization
- Strategy: 10K tokens
- Planning: 12K tokens (includes full strategy)
- 10 workers × 8K tokens = 80K tokens (each gets full context)
- Synthesis: 15K tokens (verbose worker outputs)
- **Total: ~117K tokens**

### With Context Optimization
- Strategy: 10K tokens
- Planning: 3K tokens (summarized strategy)
- 10 workers × 1K tokens = 10K tokens (minimal context, schemas)
- Synthesis: 4K tokens (compact JSON inputs)
- **Total: ~27K tokens (77% savings)**

## Best Practices Checklist

- [ ] Extract minimal context in planning phase
- [ ] Use JSON schemas to enforce compact outputs
- [ ] Summarize between stages if >2 stages
- [ ] Use references/IDs instead of duplicating large content
- [ ] Enable prompt caching for reused context (Claude)
- [ ] Lazy-load context only when needed
- [ ] Set maxLength/maxItems limits in schemas
- [ ] Avoid free-text description fields
- [ ] Pass structured data, not prose
- [ ] Compress intermediate results before next stage

## Anti-Patterns

🚫 **Passing full context to every worker**
🚫 **Verbose prose outputs from workers**
🚫 **Duplicating large documents in plan**
🚫 **No schemas (unstructured outputs)**
🚫 **Including "might need" context**
🚫 **Skipping summarization in multi-stage pipelines**

## Monitoring Context Usage

Track token usage per phase to identify optimization opportunities:

```javascript
const usage = {
  strategy: {input: 10000, output: 2000},
  planning: {input: 3000, output: 1500},
  workers: {input: 10000, output: 5000},
  synthesis: {input: 4000, output: 2000}
}

const total = Object.values(usage).reduce((sum, phase) => 
  sum + phase.input + phase.output, 0
)

// Target: <30K tokens for typical multi-tier workflow
if (total > 50000) {
  console.warn('Context optimization needed!')
}
```

## Further Reading

- Claude prompt caching: https://docs.anthropic.com/en/docs/prompt-caching
- Structured outputs: See `agent-pattern.md` for schema examples
- Workflow optimization: See `workflow-example.md`
