# Gemini Workers Reference

**Using Google's Gemini Flash models as specialized workers for multimodal, video, and image tasks.**

## Model Endpoints

All Gemini models use **global endpoints** for optimal latency and availability:

- **Gemini 3.5 Flash**: `gemini-3.5-flash` (multimodal)
- **Gemini Omni Flash Preview**: `gemini-omni-flash-preview` (video)
- **Gemini 3.1 Flash Image**: `gemini-3.1-flash-image` (image generation)

## When to Use Each Model

### Gemini 3.5 Flash (Multimodal Worker)

**Use for:**
- Document analysis with images/charts/diagrams
- Screenshot understanding and UI analysis
- Mixed media content processing
- Visual Q&A and image reasoning
- PDF/document extraction with visual elements

**Example tasks:**
- "Extract table data from this scanned document"
- "Analyze these UI mockups and compare designs"
- "Summarize this presentation with slide images"
- "What's wrong with this error screenshot?"

**Cost benefit**: ~10x cheaper than Claude for image+text tasks

### Gemini Omni Flash Preview (Video Worker)

**Use for:**
- Video generation from text descriptions
- Video editing and transformation
- Video-to-video style transfer
- Video summarization and analysis
- Clip extraction and scene detection

**Example tasks:**
- "Generate a 30-second product demo video"
- "Edit this video to add transitions"
- "Summarize key moments in this meeting recording"
- "Extract all scenes with people speaking"

**Cost benefit**: Specialized for video, no Claude equivalent

### Gemini 3.1 Flash Image (Image Generation Worker)

**Use for:**
- Image generation from text prompts
- Image editing and manipulation
- Style transfer and transformations
- Visual content creation
- Image-to-image translation

**Example tasks:**
- "Generate a hero image for this blog post"
- "Edit this product photo to change background"
- "Create icons for these UI components"
- "Transform this sketch into a realistic rendering"

**Cost benefit**: Flash-optimized for fast, cheap image generation

## Integration Patterns

### Pattern 1: Multimodal Analysis with Gemini 3.5 Flash

```javascript
// Opus creates plan for analyzing documents with images
phase('Plan')
const plan = await agent('Analyze these 5 product comparison PDFs...', {
  model: 'opus',
  schema: PLAN_SCHEMA
})

// Gemini 3.5 Flash processes each document (multimodal)
phase('Analyze')
const results = await parallel(plan.documents.map(doc => () =>
  agent(`Analyze this document (images + text): ${doc.url}
  
  Extract: pricing tables, feature comparisons, visual diagrams.
  Output structured data.`, {
    model: 'gemini-3.5-flash',
    label: `analyze-${doc.name}`
  })
))

// Opus synthesizes
phase('Synthesize')
const report = await agent('Combine analysis results...', {model: 'opus'})
```

### Pattern 2: Video Generation Pipeline

```javascript
// Fable provides creative strategy
phase('Strategy')
const strategy = await agent('Create video strategy for product launch...', {
  model: 'fable',
  effort: 'high'
})

// Opus creates detailed video scripts
phase('Plan')
const scripts = await agent(`Based on strategy, create 3 video scripts:
${JSON.stringify(strategy)}

Each script: scene descriptions, timing, key messages.`, {
  model: 'opus'
})

// Gemini Omni Flash generates videos in parallel
phase('Generate')
const videos = await parallel(scripts.videos.map(script => () =>
  agent(`Generate video from script:
${script.content}

Duration: ${script.duration}s
Style: ${script.style}`, {
    model: 'gemini-omni-flash-preview',
    label: `video-${script.id}`
  })
))
```

### Pattern 3: Image Generation for Content

```javascript
// Opus creates content + image requirements
phase('Plan')
const content = await agent('Create blog post with image requirements...', {
  model: 'opus'
})

// Gemini 3.1 Flash Image generates all images in parallel
phase('Generate Images')
const images = await parallel(content.images.map(img => () =>
  agent(`Generate image:
Prompt: ${img.prompt}
Style: ${img.style}
Dimensions: ${img.dimensions}`, {
    model: 'gemini-3.1-flash-image',
    label: `image-${img.id}`
  })
))

// Current model assembles final blog post with images
const finalPost = assemblePost(content, images)
```

### Pattern 4: Mixed Claude + Gemini Workers

```javascript
// Opus plans hybrid workflow
phase('Plan')
const plan = await agent('Research competitors: analyze websites + generate comparison chart', {
  model: 'opus',
  schema: PLAN_SCHEMA
})

// Parallel execution: Haiku for text, Gemini for images
phase('Execute')
const [textResults, imageResults] = await Promise.all([
  // Haiku workers for text extraction
  parallel(plan.textTasks.map(task => () =>
    agent(task.prompt, {model: 'haiku'})
  )),
  
  // Gemini 3.5 Flash for screenshot analysis
  parallel(plan.imageTasks.map(task => () =>
    agent(task.prompt, {model: 'gemini-3.5-flash'})
  ))
])

// Opus synthesizes both
phase('Synthesize')
const report = await agent('Combine text + visual analysis...', {model: 'opus'})
```

## API Access Methods

### Method 1: Via Agent Tool (Recommended)

```javascript
Agent({
  model: 'gemini-3.5-flash',  // or 'gemini-omni-flash-preview', 'gemini-3.1-flash-image'
  prompt: '...'
})
```

### Method 2: Direct API Calls (if needed)

If Agent tool doesn't support Gemini models, use direct API:

```python
from google import genai

client = genai.Client(api_key='...')

# Multimodal analysis
response = client.models.generate_content(
    model='gemini-3.5-flash',
    contents='...'
)

# Image generation
response = client.models.generate_content(
    model='gemini-3.1-flash-image',
    contents='Generate: ...'
)

# Video generation
response = client.models.generate_content(
    model='gemini-omni-flash-preview',
    contents='Create video: ...'
)
```

### Method 3: Via MCP Server (if configured)

If you have a Gemini MCP server configured, use it for authentication/routing.

## Cost Optimization

### Example: Document Analysis (10 PDFs with images)

**Approach A: Claude Sonnet**
- 10 calls @ 15K tokens each = $0.45

**Approach B: Multi-tier with Gemini**
- Opus plan (2K tokens) = $0.030
- 10x Gemini 3.5 Flash (3K tokens each) = $0.015
- Opus synthesis (4K tokens) = $0.060
- **Total = $0.105** (77% savings)

### Example: Video Generation (3 videos)

**Approach A: External service** (e.g., Runway)
- 3 videos @ $0.50 each = $1.50

**Approach B: Multi-tier with Gemini**
- Fable strategy (1.5K tokens) = $0.045
- Opus scripts (3K tokens) = $0.045
- 3x Gemini Omni Flash = $0.30 (estimated)
- **Total = $0.39** (74% savings)

## Best Practices

1. **Use Gemini for specialized tasks only**
   - Don't use Gemini 3.5 Flash for pure text (Haiku is better)
   - Reserve video/image models for actual generation, not planning

2. **Batch multimodal work**
   - Process all images in parallel
   - Generate all videos together
   - Minimize API round-trips

3. **Quality check with Claude**
   - Use Opus to validate Gemini outputs
   - Have Opus refine or fix issues
   - Claude synthesis ensures coherence

4. **Monitor costs**
   - Track Gemini API usage separately
   - Compare vs Claude-only approach
   - Optimize worker allocation based on actual costs

## Troubleshooting

**Q: Agent tool doesn't recognize Gemini models?**
→ Use direct API calls or configure Gemini MCP server

**Q: Gemini Flash quality lower than expected?**
→ Use Opus to refine prompts, then retry with Gemini
→ For critical tasks, use Claude Sonnet instead

**Q: Video generation too slow?**
→ Reduce video duration/complexity in planning phase
→ Generate videos in parallel, not sequentially

**Q: Need to pass images to Gemini?**
→ Use base64 encoding or public URLs in prompts
→ Ensure global endpoint access to image sources
