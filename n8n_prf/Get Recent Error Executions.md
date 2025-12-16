# Get Recent Error Executions

## Overview
**Workflow ID:** j9OnjzIJCrRy9hQY  
**Status:** Active  
**Created:** August 7, 2025  
**Last Updated:** December 16, 2025

## Purpose
Provides an API endpoint to retrieve recent workflow execution errors for monitoring and debugging purposes.

## Trigger
**Type:** Webhook  
**Method:** GET  
**Path:** `/webhook/get-errors`  
**URL:** https://prf.thesqd.com/webhook/get-errors  
**Response Mode:** Last node (returns JSON)

## Workflow Process

### 1. Fetch Error Executions
- Queries n8n API for executions
- Filters by status: "error"
- Limit: 250 executions

### 2. Filter Recent Non-Manual Errors
Filters for executions where:
- **Time:** Started within last 1 hour
- **Mode:** NOT "manual" (excludes test runs)

### 3. Format Response
Returns JSON with:
- `executions`: Array of error execution objects
- `count`: Total number of recent errors

## Response Format
```json
{
  "executions": [
    {
      "id": "execution_id",
      "workflowId": "workflow_id",
      "mode": "trigger",
      "startedAt": "2025-12-16T...",
      "stoppedAt": "2025-12-16T...",
      "status": "error",
      ...
    }
  ],
  "count": 5
}
```

## Key Features
- **Recent Errors Only:** Last hour window
- **Excludes Manual Runs:** Only production errors
- **Execution Context:** Full error execution details
- **Quick Response:** Returns formatted JSON immediately

## Use Cases
- Monitoring dashboards
- Error alerting systems
- Debugging tools
- Health check integration
- Analytics and reporting
- Automated error notifications

## Integration Points
- Monitoring systems
- Alerting platforms
- Status dashboards
- Error tracking tools

## Data Retrieved
Each execution includes:
- Execution ID
- Workflow ID
- Execution mode
- Start/stop timestamps
- Status
- Error details
- And other execution metadata

## Time Window
- **Lookback:** 1 hour from current time
- **Calculation:** `$now.minus(1,'hour')`

## Notes
- Useful for real-time error monitoring
- Excludes test/manual executions for cleaner data
- Can be called by external monitoring systems
- No authentication required (internal endpoint)
- Returns empty array if no errors in last hour
- Helpful for identifying problematic workflows quickly

