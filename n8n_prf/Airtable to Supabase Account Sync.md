# Airtable to Supabase Account Sync to prf_airtable_account_records

## Overview
**Workflow ID:** S59o9PgWUrkUL2sn  
**Status:** Active  
**Created:** July 23, 2025  
**Last Updated:** December 16, 2025

## Purpose
Synchronizes account data from Airtable to Supabase, maintaining a reference table of account records with member numbers and church names.

## Triggers
**Type:** Manual + Schedule

### Manual Trigger
Execute via n8n interface ("Execute workflow" button)

### Schedule Trigger
**Frequency:** Every minute

## Workflow Process

### 1. Fetch Airtable Accounts
- Searches all accounts from Accounts table
- Retrieves only essential fields:
  - Member #
  - Church Name

### 2. Filter Valid Records
- Filters for accounts with non-empty Member # field
- Ensures data quality before sync

### 3. Upsert to Supabase
Inserts or updates records in `prf_airtable_account_records`:
- **id:** Airtable record ID
- **member_number:** Member # field
- **church_name:** Church Name field
- **last_synced:** Current timestamp

## Key Features
- **High Frequency:** Runs every minute for near real-time sync
- **Upsert Logic:** Updates existing records, creates new ones
- **Timestamp Tracking:** Records last sync time for monitoring
- **Data Validation:** Filters out incomplete records

## Database Tables

### Source (Airtable)
- Accounts (tblJEtYMR7ZPSOkc6)

### Destination (Supabase)
- `prf_airtable_account_records`

## Table Schema: prf_airtable_account_records
```sql
id TEXT PRIMARY KEY,
member_number TEXT NOT NULL,
church_name TEXT,
last_synced TIMESTAMP
```

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Save Progress:** Enabled
- **Caller Policy:** Workflows from same owner

## Performance
- **Frequency:** Every minute
- **Batch Processing:** Processes all accounts in single run
- **Upsert Operation:** Efficient update or insert

## Use Cases
- Reference table for member number lookups
- Quick church name retrieval without Airtable API calls
- Cache for account data in Supabase queries
- Data consistency verification

## Notes
- Very lightweight workflow
- Minimal fields synced for performance
- High frequency suitable for reference data
- Uses Airtable record ID as primary key
- Filters ensure only valid accounts synced




