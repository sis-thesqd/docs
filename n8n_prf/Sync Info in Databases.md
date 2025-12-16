# Sync Info in Databases

## Overview
**Workflow ID:** rTbgVzOg8MHJ53Pf  
**Status:** Active  
**Created:** May 29, 2025  
**Last Updated:** December 16, 2025

## Purpose
Synchronizes PRF general submission data across multiple databases (Supabase and Airtable), ensuring data consistency and creating necessary relational records.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/prf-general-submission`  
**URL:** https://prf.thesqd.com/webhook/prf-general-submission

## Workflow Process

### 1. Initial Data Storage (Parallel)
The workflow simultaneously:
- Stores general form data in `prf_general_submissions` (Supabase)
- Stores selection data in `prf_selection_submissions` (Supabase)
- Saves execution data for tracking

### 2. Project Type Processing
- Splits selected project types array
- Fetches project type details from `prf_selection_types`
- Formats project fields with ministry and account information
- Aggregates all project selections

### 3. User Information
- Retrieves or creates user record from Airtable
- Uses ClickUp User ID for lookup

### 4. Ministry Department Sync
- Checks for existing ministry department in Supabase
- Creates or updates department record in Airtable with:
  - Department Name
  - Account link
  - Member ID
  - Category
  - Supabase ID

### 5. User Selection Record
- Creates/updates user selection in Airtable
- Links to:
  - Submission ID
  - Account
  - Guest record

### 6. Project Selection Record
- Creates/updates project selection in Airtable
- Links all selected project types

### 7. General Information Record
- Final record creation in Airtable with:
  - Submission details
  - Account link
  - Audience/Department
  - Description
  - Creative direction with inspiration images
  - Design style
  - Links to User Selection and Project Selection records

## Key Features
- **Parallel Processing:** Multiple database operations happen simultaneously
- **Data Consistency:** Ensures all databases have matching records
- **Relational Integrity:** Maintains proper links between records
- **Error Handling:** Error workflow configured (38tttU6Tcl3Y8tZy)
- **Retry Logic:** Automatic retries on failures
- **Typecast Support:** Enables flexible data type handling

## Database Tables Updated

### Supabase
- `prf_general_submissions`
- `prf_selection_submissions`
- `prf_ministry_departments`

### Airtable
- General Information (tblbHuKhDjUCwebAK)
- Account - Department (tbl93Rrk6L4jZCkhe)
- User Selection (tbliUuVlf8xQsPBul)
- Project Selection (tblik6QFXNagHRclV)

## Data Mapping

### Design Style Mapping
```
'experimental' → 'Experimental'
'grunge' → 'Grunge'
'minimal' → 'Minimal'
'photo-driven' → 'Photo Driven'
'professional' → 'Professional'
'type-driven' → 'Type Driven'
```

### Imagery Format
Inspiration images formatted as:
```
Inspo image 1 link
[URL with spaces encoded]
What We Like: [description]
```

## Execution Settings
- **Execution Order:** v1
- **Save Progress:** Enabled
- **Timeout:** 300 seconds
- **Caller Policy:** Workflows from same owner

## Notes
- Critical for maintaining data synchronization across systems
- Handles complex relational data structures
- Supports multiple project type selections
- Properly encodes image URLs with spaces

