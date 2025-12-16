---
title: "analyze-transactions"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["transactions", "analysis", "ai", "financial", "anomaly-detection"]
version: "8"
---

# analyze-transactions

Performs comprehensive analysis of financial transactions using mathematical analysis and optional AI-powered insights.

## Overview
**Function Slug:** analyze-transactions  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 8

## Purpose
Analyzes recent financial transactions for anomalies, patterns, and insights. Combines statistical analysis with optional OpenAI-powered analysis to provide actionable recommendations.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/analyze-transactions
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/analyze-transactions \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "analysisId": "123e4567-e89b-12d3-a456-426614174000",
  "summary": {
    "transactionsAnalyzed": 100,
    "anomaliesFound": 5,
    "riskScore": 35,
    "hasAIAnalysis": true
  }
}
```

## Analysis Components

### Mathematical Analysis
- **Mean Calculation:** Average transaction amount
- **Standard Deviation:** Variance analysis
- **Anomaly Detection:** Transactions > 2σ from mean
- **Merchant Analysis:** Top spending merchants
- **Category Breakdown:** Spending by category
- **Risk Score:** Calculated risk (0-100)

### AI Analysis (Optional)
- **Pattern Detection:** Unusual spending patterns
- **Cost Savings:** Optimization opportunities
- **Risk Assessment:** Areas requiring attention
- **Recommendations:** Actionable insights
- Uses GPT-4o-mini model
- Requires OpenAI API key

## Database Tables

### Table: transactions
```sql
SELECT * FROM transactions
ORDER BY created_at DESC
LIMIT 100
```

### Table: baseline_data
```sql
SELECT * FROM baseline_data
WHERE is_active = true
```

### Table: analysis_results
```sql
INSERT INTO analysis_results (
  analysis_type,
  mathematical_analysis,
  ai_explanation,
  findings,
  confidence_score,
  risk_score,
  baseline_comparison,
  recommendations
)
```

## Anomaly Detection

### Criteria
- Transactions > 2 standard deviations from mean
- Calculated as: `|amount - mean| > 2 * stdDev`

### Anomaly Data
```json
{
  "id": "transaction_id",
  "amount": 1500.00,
  "merchant": "Vendor Name",
  "description": "Transaction description",
  "deviation": 2.5
}
```

## Risk Score Calculation

### Formula
```typescript
riskScore = min(100, 
  (anomalies.length / totalTransactions) * 100 + 
  (stdDev > mean * 0.5 ? 25 : 0)
)
```

### Risk Levels
- **0-25:** Low risk
- **26-50:** Moderate risk
- **51-75:** High risk
- **76-100:** Critical risk

## Baseline Comparison

### Comparison Metrics
- Current average vs baseline average
- Variance percentage
- Exceeds baseline threshold (20%)

### Baseline Data
```json
{
  "current_average": 250.00,
  "baseline_average": 200.00,
  "variance_percentage": 25.0,
  "exceeds_baseline": true
}
```

## AI Analysis

### OpenAI Integration
- Model: `gpt-4o-mini`
- Max tokens: 1000
- Temperature: 0.3
- System prompt: Financial analyst expert

### AI Output
- Explanation text
- Confidence score (0.85)
- Insights array
- Recommendations array

## Recommendations

### Mathematical Recommendations
- Review high-value transactions if >5 anomalies
- Implement spending controls if risk >50
- Consolidate vendors if >20 merchants

### AI Recommendations
- Implement AI-suggested controls
- Monitor flagged transactions
- Review spending policies quarterly

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |
| `OPENAI_API_KEY` | No | OpenAI API key (optional) |

## Use Cases

1. **Fraud Detection:** Identify unusual transactions
2. **Spending Analysis:** Understand spending patterns
3. **Budget Monitoring:** Compare against baseline
4. **Risk Assessment:** Calculate risk scores
5. **Optimization:** Find cost savings opportunities

## Notes

- Analyzes last 100 transactions
- Requires baseline_data table for comparison
- AI analysis optional (requires OpenAI key)
- Stores results in analysis_results table
- Risk score calculated automatically
- Anomalies detected via statistical analysis
- Merchant and category breakdown included
- Recommendations generated automatically
- Confidence scores provided for both analyses
- Baseline comparison optional (if baseline exists)

