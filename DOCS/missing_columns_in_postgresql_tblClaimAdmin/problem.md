# ClaimAdmin Database Schema Mismatch - Problem Documentation

## Problem Overview

The Django application was experiencing multiple database-related errors due to a mismatch between the Django model definition and the actual database table structure for the `tblClaimAdmin` table.

## Root Cause

**Schema Drift**: The Django model (`ClaimAdmin`) expected specific database columns that didn't exist in the PostgreSQL database table (`tblClaimAdmin`). This commonly occurs when:

1. Database migrations were not properly applied
2. Manual database changes were made without updating Django migrations
3. The application was deployed to a database that wasn't properly synchronized with the model definitions
4. Legacy database structure didn't match the current Django model requirements

## Error Sequence

The errors appeared in a cascading pattern as Django tried to query non-existent columns:

### 1. Initial Error - Table Not Found
```
relation "tblClaimAdmin" does not exist
```
**Issue**: The entire table was missing from the database.

### 2. Column Missing Errors (Sequential Discovery)
After creating the table, Django revealed missing columns one by one:

```sql
-- Error sequence as Django tried to query each missing column:
column tblClaimAdmin.LastName does not exist
column tblClaimAdmin.OtherNames does not exist
column tblClaimAdmin.DOB does not exist
column tblClaimAdmin.EmailId does not exist
column tblClaimAdmin.Phone does not exist
column tblClaimAdmin.HFId does not exist
column tblClaimAdmin.HasLogin does not exist
column tblClaimAdmin.AuditUserId does not exist
column tblClaimAdmin.ValidityFrom does not exist
column tblClaimAdmin.ValidityTo does not exist
column tblClaimAdmin.LegacyID does not exist
column tblClaimAdmin.ClaimAdminId does not exist
column tblClaimAdmin.ClaimAdminUUID does not exist
column tblClaimAdmin.ClaimAdminCode does not exist
```

### 3. Data Query Error
```
ClaimAdmin matching query does not exist
```
**Issue**: After fixing the schema, the table existed but contained no data, causing Django queries to fail.

## Django Model vs Database Schema

### Django Model Fields (Expected)
```python
class ClaimAdmin(models.Model):
    last_name = models.CharField(db_column='LastName', max_length=100, blank=True, null=True)
    other_names = models.CharField(db_column='OtherNames', max_length=100, blank=True, null=True)
    dob = models.DateField(db_column='DOB', blank=True, null=True)
    email_id = models.CharField(db_column='EmailId', max_length=200, blank=True, null=True)
    phone = models.CharField(db_column='Phone', max_length=50, blank=True, null=True)
    health_facility = models.ForeignKey("location.HealthFacility", models.DO_NOTHING, db_column='HFId', blank=True, null=True)
    has_login = models.BooleanField(db_column='HasLogin', blank=True, null=True)
    audit_user_id = models.IntegerField(db_column='AuditUserId', blank=True, null=True)
    # Additional fields discovered through errors:
    # ValidityFrom, ValidityTo, LegacyID, ClaimAdminId, ClaimAdminUUID, ClaimAdminCode
```

### Database Schema (Missing)
The PostgreSQL table `tblClaimAdmin` was either:
- Completely missing, or
- Missing most/all of the required columns
