# Sync Comms Rep for Socials

## Overview
**Workflow ID:** vwseuUgxgHL8LO7b  
**Status:** Active  
**Created:** July 26, 2025  
**Last Updated:** December 16, 2025

## Purpose
Updates the communications representative (comms_rep) field in the accounts table when changes occur, maintaining accurate contact information for social media projects.

## Trigger
**Type:** Webhook  
**Method:** POST  
**Path:** `/webhook/sync-comms-rep-for-socials`  
**URL:** https://prf.thesqd.com/webhook/sync-comms-rep-for-socials

## Workflow Process

### 1. Receive Update Request
- Webhook receives payload with:
  - `member`: Account number
  - `comms_rep`: New communications representative value

### 2. Update Database
- Updates `accounts` table in Supabase
- Sets `comms_rep` field for specified account
- Matches on `account` field

## Key Features
- **Simple & Fast:** Single-purpose, minimal overhead
- **Direct Update:** No complex logic or branching
- **Real-time Sync:** Immediate update on trigger

## Database Tables

### Supabase
- `accounts` (write)
  - Matches on: `account` (member number)
  - Updates: `comms_rep`

## Use Cases
- CSS Rep assignment changes
- Communications contact updates
- Social media project coordination
- Account management synchronization

## Integration Points
- Likely triggered by Airtable automations
- Updates used by social media project workflows
- Maintains communication chain for client interactions

## Notes
- Extremely lightweight workflow
- No error handling specified (uses defaults)
- Fast execution time
- Critical for social media project assignments
- Ensures designers know who to coordinate with




