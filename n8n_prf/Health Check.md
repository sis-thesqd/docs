# Health Check

## Overview
**Workflow ID:** EBBWi6flFvdkc0JR  
**Status:** Active  
**Created:** September 17, 2025  
**Last Updated:** December 16, 2025

## Purpose
Provides a simple health check endpoint to verify the n8n workflow server is running and responsive.

## Trigger
**Type:** Webhook  
**Method:** GET  
**Path:** `/webhook/health-check`  
**URL:** https://prf.thesqd.com/webhook/health-check  
**Response Mode:** Last node (returns JSON)

## Workflow Process

### 1. Receive Health Check Request
- GET request to health check endpoint

### 2. Return Success Response
- Returns JSON object:
  ```json
  {
    "success": true
  }
  ```

## Key Features
- **Simple & Fast:** Minimal processing for quick response
- **Standard Endpoint:** Common pattern for monitoring
- **JSON Response:** Machine-readable success indicator

## Response Format
```json
{
  "success": true
}
```

## Use Cases
- Monitoring system checks
- Load balancer health verification
- Uptime monitoring services
- API status dashboards
- Deployment verification

## Integration Points
- Uptime monitoring tools (e.g., Uptime Robot)
- Load balancers
- Status page services
- Deployment pipelines

## Notes
- No database operations
- No error handling needed
- Immediate response
- Always returns success (as long as workflow executes)
- Can be used to verify n8n instance is running
- Good practice for production systems


