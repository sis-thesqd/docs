# Trigger: update_prf_project_submissions_timestamp

This trigger fires `BEFORE UPDATE` on the `public.prf_project_submissions` table and executes the `update_timestamp()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_prf_project_submissions_timestamp BEFORE UPDATE ON public.prf_project_submissions FOR EACH ROW EXECUTE FUNCTION update_timestamp()
```

## Associated Function

The `update_timestamp()` function is also used by the `update_prf_general_submissions_timestamp` trigger and is documented [here](supabase/triggers/update_prf_general_submissions_timestamp.md).



