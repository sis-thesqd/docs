# Sync Guests to Supabase

## Overview
**Workflow ID:** TZDioLUF9z7P3Mgb  
**Status:** Active  
**Created:** July 1, 2025  
**Last Updated:** December 16, 2025

## Purpose
Synchronizes guest user data from Airtable to Supabase, maintaining a reference table for quick guest lookups.

## Triggers
**Type:** Manual + Schedule

### Manual Trigger
Execute via n8n interface ("Test workflow" button)

### Schedule Trigger
**Frequency:** Every 15 minutes

## Workflow Process

### 1. Fetch Airtable Guests
- Searches all guest records from Guests table in Airtable
- Retrieves all fields

### 2. Filter Valid Records
- Filters for records with:
  - Non-empty `id` field
  - Non-empty `Clickup User ID` field
- Ensures data quality before sync

### 3. Upsert to Supabase
Inserts or updates records in `prf_airtable_guests`:
- **id:** Airtable record ID (primary key)
- **clickup_user_id:** ClickUp User ID
- **email:** Guest email address
- **name:** Guest name
- **last_synced:** Current timestamp

## Key Features
- **High Frequency:** Runs every 15 minutes for near real-time sync
- **Upsert Logic:** Updates existing, creates new records
- **Timestamp Tracking:** Records last sync time
- **Data Validation:** Filters incomplete records
- **Retry Logic:** Enabled on Airtable fetch

## Database Tables

### Source (Airtable)
- Guests (tblSpD6Sw8VVnr0Wh)

### Destination (Supabase)
- `prf_airtable_guests`

## Table Schema: prf_airtable_guests
```sql
id TEXT PRIMARY KEY,
clickup_user_id TEXT NOT NULL,
email TEXT,
name TEXT,
last_synced TIMESTAMP
```

## Performance
- **Frequency:** Every 15 minutes
- **Batch Processing:** All guests in single run
- **Upsert Operation:** Efficient update or insert

## Use Cases
- Quick guest lookups without Airtable API
- ClickUp User ID to email mapping
- Guest name retrieval for notifications
- Data consistency verification
- Reduced API calls to Airtable

## Integration Points
- Other PRF workflows that need guest information
- User identification in webhooks
- Email notifications
- Task assignment lookups

## Notes
- Lighter weight than calling Airtable API repeatedly
- 15-minute sync keeps data fresh
- Uses Airtable record ID as primary key
- ClickUp User ID is required field
- Maintains guest data consistency across systems
- Enables faster guest lookups in other workflows

