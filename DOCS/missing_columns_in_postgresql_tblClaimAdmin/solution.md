# ClaimAdmin Database Fix - Quick Solution

## Problem
Django application throwing errors:
- `relation "tblClaimAdmin" does not exist`
- `column tblClaimAdmin.{ColumnName} does not exist`
- `ClaimAdmin matching query does not exist`

## Solution

### Step 1: Fix Database Schema
Download and run the [SQL Script](sql_script.psql) using your preferred PostgreSQL tool (pgAdmin, DBeaver, psql command line, etc.)

This script will:
- Create the `tblClaimAdmin` table if it doesn't exist
- Add all missing columns required by your Django model
- Create necessary indexes for performance

### Step 2: Fix Data References
Go to the `core_User` table and either:
- **Option A**: Delete all rows that have `claimAdmin` field populated
- **Option B**: Ensure those referenced ClaimAdmin records actually exist in `tblClaimAdmin`

This prevents "ClaimAdmin matching query does not exist" errors.

## Why This Works
- **Step 1** fixes the schema mismatch between Django model and database structure
- **Step 2** fixes orphaned foreign key references that cause query failures

## Files Needed
- `sql_script.psql` - Database schema fix script
