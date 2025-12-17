# Weekly Sermon Pack

## Overview
**Workflow ID:** 3rTdiJ1jZbV9Vhv8  
**Status:** Active  
**Created:** December 5, 2025  
**Last Updated:** December 16, 2025

## Purpose
Orchestrates the processing of weekly sermon pack submissions by routing to appropriate sub-workflows based on content type (Social Media vs Web).

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/weekly-sermon-pack`  
**URL:** https://prf.thesqd.com/webhook/weekly-sermon-pack

## Workflow Process

### 1. Form Submission Receipt
- Receives webhook payload from Fillout
- Fetches detailed submission data including all form fields

### 2. Content Extraction
- Extracts two separate content blocks from calculations:
  - **SM Final Content:** Social Media task content
  - **Web Final Content:** Website task content

### 3. Content Cleanup
- Processes both content blocks through Content Cleanup API workflow
- Removes temporary variables and standardizes formatting

### 4. Routing Logic
Checks if content contains "Social Media Content":
- **If YES:** Routes to SMS Project Handler workflow
- **If NO:** Routes through rawContent processor first, then to Web Support Request workflow

### 5. Sub-Workflow Execution

#### SMS Project Handler Path
- Passes cleaned content, submission data, and form ID
- Handles social media project creation

#### Web Support Request Path
- First replaces file variables via rawContent workflow
- Then processes web support request
- Uses form ID: `jDwYpx1y8Zus`

## Key Features
- **Dual Processing:** Handles both social media and web content in one submission
- **Intelligent Routing:** Automatically determines correct workflow based on content type
- **Modular Design:** Leverages existing workflows for specialized processing

## Sub-Workflows Called
1. **Content Cleanup API** (osQiLJO0h5Rj002x)
2. **Replace File Variables** (FnwUJbM24oKBhILK)
3. **SMS Project Handler** (3oY3dGhM5SuYghYV)
4. **Website Support Requests** (x1clzmVD3YwUrxZn)

## Data Flow
```
Webhook → Detail Submission → Code Split → Content Cleanup → Routing Decision
  ↓                                                              ↓
  SMS Path                                                   Web Path
  ↓                                                              ↓
SMS Project Handler                                    rawContent → Web Support
```

## Integration Points
- Fillout API for form submissions
- Content Cleanup API for standardization
- SMS and Web processing workflows

## Notes
- Designed specifically for weekly sermon pack submissions
- Handles dual-output form (SM + Web content)
- All sub-workflows execute with wait enabled
- Form ID hardcoded for web support: `jDwYpx1y8Zus`




