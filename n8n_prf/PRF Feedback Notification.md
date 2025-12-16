# PRF Feedback Notification

## Overview
**Workflow ID:** LJE9zB7TiGctJjiA  
**Status:** Active  
**Created:** May 27, 2025  
**Last Updated:** December 16, 2025

## Purpose
Processes user feedback submissions, generates AI-powered titles, and creates Linear issues for the team to review and address.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/process-feedback`  
**URL:** https://prf.thesqd.com/webhook/process-feedback

## Workflow Process

### 1. Receive Feedback
- Webhook receives feedback payload with:
  - `user_id`: ClickUp User ID of submitter
  - `member_id`: Account member number
  - `category`: Feedback category
  - `message`: Feedback content

### 2. AI Title Generation
- Uses Anthropic Claude Sonnet 4 via LangChain
- Generates concise title (under 5 words) based on:
  - Category
  - Message content
- Fallback: "PRF Feedback" if unable to derive meaningful title

### 3. Guest Lookup
- Searches Guests table for user by ClickUp User ID
- Retrieves:
  - Name
  - Email
  - Church Name (from Account Link)

### 4. Create Linear Issue
Creates issue in Linear with:
- **Team ID:** bbc0f622-f700-498d-89f1-c81934fe7c42
- **Title:** AI-generated title
- **Assignee:** 8c7c37b2-b01d-464a-866c-e3dcd12bbc8e
- **State:** 3f182bc7-7a00-4e1a-9546-2a70ed887b78
- **Priority:** 2 (High)
- **Description:**
  ```
  **Member ID:** {member_id} - {Church Name}
  
  **Submitted by:** {Guest Name}
  **User Email:** {Guest Email}
  
  **Category:** {category}
  **Feedback:** {message}
  ```

## AI Configuration

### Model
- **Provider:** Anthropic
- **Model:** Claude Sonnet 4 (claude-sonnet-4-20250514)

### Prompt
```
Considering the category, create a title or subject for the feedback message. 
Keep the title under only 5 words. If the title can't be derived from the 
message, return "PRF Feedback"

Category: {category}
Message: {message}
```

## Key Features
- **AI-Powered:** Intelligent title generation
- **Linear Integration:** Automatic issue creation
- **Context-Rich:** Includes user and church information
- **High Priority:** All feedback flagged as important

## Database Tables

### Airtable
- Guests (tbl8BhTfaBl4F410W) - Read

### External Systems
- Linear - Issue creation

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Timeout:** 300 seconds
- **Save Progress:** Enabled

## Linear Issue Structure
- Assigned to specific team member
- High priority (2)
- Specific state for feedback issues
- Rich description with context

## Use Cases
- User feedback collection
- Feature requests
- Bug reports
- General suggestions
- User experience improvements

## Notes
- Uses LangChain for AI integration
- Feedback automatically escalated to Linear
- Maintains connection between church, user, and feedback
- AI ensures consistent, concise titles
- All feedback visible to assigned team member

