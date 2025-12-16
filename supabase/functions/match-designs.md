---
title: "match-designs"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["ai", "matching", "embeddings", "design", "similarity"]
version: "3"
---

# match-designs

AI-powered design matching using vector embeddings to find visually and contextually similar designs.

## Overview
**Function Slug:** match-designs  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 3

## Purpose
Uses machine learning embeddings to match designs based on visual similarity (image embeddings) and contextual similarity (text embeddings). Employs cosine similarity to find designs that match the style and content of liked designs.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/match-designs
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `liked_design_keys` | array | Yes | Array of design keys user has liked |
| `exclude_ids` | array | No | Design keys to exclude from results |
| `category` | string | No | Filter by category (currently unused) |
| `threshold` | number | No | Minimum similarity score (default 0.1) |
| `match_count` | number | No | Number of matches to return (default 10) |
| `match_mode` | string | No | 'visual', 'content', or 'both' (default 'visual') |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/match-designs \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "liked_design_keys": ["design_123", "design_456", "design_789"],
    "exclude_ids": ["design_999"],
    "threshold": 0.2,
    "match_count": 15,
    "match_mode": "both"
  }'
```

### JavaScript Example

```javascript
const likedDesigns = ['design_123', 'design_456', 'design_789']

const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/match-designs', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    liked_design_keys: likedDesigns,
    match_count: 10,
    match_mode: 'both',
    threshold: 0.15
  })
})

const data = await response.json()
console.log(`Found ${data.data.length} similar designs`)
data.data.forEach(match => {
  console.log(`${match.key}: ${(match.similarity * 100).toFixed(1)}% match`)
})
```

## Response

### Success Response (200)

```json
{
  "data": [
    {
      "key": "design_555",
      "similarity": 0.847,
      "individual_similarities": [
        {
          "key": "design_123",
          "similarity": 0.847
        },
        {
          "key": "design_456",
          "similarity": 0.723
        },
        {
          "key": "design_789",
          "similarity": 0.612
        }
      ]
    },
    {
      "key": "design_666",
      "similarity": 0.792,
      "individual_similarities": [...]
    }
  ],
  "error": null
}
```

### Error Responses

**400 Bad Request** - Invalid input
```json
{
  "error": "liked_design_keys is required and must be a non-empty array",
  "data": null
}
```

**500 Internal Server Error**
```json
{
  "error": "Error message details",
  "data": null
}
```

## Match Modes

### Visual Mode (default)
- Uses text embeddings from design descriptions
- Matches based on semantic content and context
- Good for finding designs with similar themes/messaging
- Processes ~10,000 text embeddings

### Content Mode
- Uses image embeddings from design visuals
- Matches based on visual similarity
- Good for finding designs with similar aesthetics
- Processes ~2,000 image embeddings

### Both Mode
- Combines visual and content matching
- Returns best matches from either method
- Comprehensive similarity analysis
- Longer processing time but better results

## Similarity Scoring

### Cosine Similarity
- Range: 0.0 to 1.0
- 1.0 = identical embeddings
- 0.0 = completely different
- Typical matches: 0.3-0.8

### Threshold
- Default: 0.1 (permissive)
- Recommended: 0.2-0.3
- Higher threshold = fewer, better matches
- Lower threshold = more, varied matches

### Individual Similarities
Each match includes similarity scores to each liked design:
- Helps understand why a design matched
- Identifies strongest connections
- Sorted by similarity (highest first)

## Database Tables

### style_image_embeddings
```sql
Columns:
- key (text, primary key)
- embedding (vector/array)
- metadata (jsonb)
- file_path (text)
- entity_id (text)
- source_type (text)
```

### style_text_embeddings
```sql
Columns:
- id (uuid, primary key)
- embedding (vector/array)
- metadata (jsonb)
```

## Performance

### Processing Time
- Visual mode: 1-3 seconds
- Content mode: 2-4 seconds
- Both mode: 3-6 seconds

### Optimization
- Processes max 2,000 image embeddings
- Processes max 10,000 text embeddings
- Uses efficient cosine similarity calculation
- In-memory computation for speed

### Logging
Function logs:
- Number of liked designs
- Number of candidates evaluated
- Processing duration
- Top match count

## Use Cases

1. **Design Recommendations:** "More like this" feature
2. **Style Matching:** Find designs with similar aesthetics
3. **Theme Discovery:** Find designs with related content
4. **Client Preferences:** Match based on liked designs
5. **Asset Curation:** Build collections of similar designs

## Algorithm

1. **Load Liked Embeddings** - Fetch embeddings for user's liked designs
2. **Load Candidate Pool** - Get all available design embeddings
3. **Calculate Similarities** - Compute cosine similarity for each candidate
4. **Track Best Matches** - Store max similarity to any liked design
5. **Filter by Threshold** - Keep only candidates above threshold
6. **Sort and Limit** - Return top N matches by similarity

## Examples

### Find Similar Visual Designs
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/match-designs \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "liked_design_keys": ["modern_sermon_01"],
    "match_mode": "content",
    "match_count": 5
  }'
```

### Comprehensive Matching
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/match-designs \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "liked_design_keys": ["christmas_01", "christmas_02"],
    "exclude_ids": ["christmas_03"],
    "match_mode": "both",
    "threshold": 0.25,
    "match_count": 20
  }'
```

## Notes

- Requires pre-computed embeddings in database
- Embeddings generated offline via ML model
- Higher match_count = longer processing time
- Exclude_ids prevents showing already seen designs
- Individual similarities help explain matches
- Source_type filter limits to 'remix' designs
- Entity_id links images to text embeddings
- Cosine similarity is computed in-memory (not database)
- Function is idempotent (same input = same output)

