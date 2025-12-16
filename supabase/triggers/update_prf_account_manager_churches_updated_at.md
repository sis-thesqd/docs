# Trigger: update_prf_account_manager_churches_updated_at

This trigger fires `BEFORE UPDATE` on the `public.prf_account_manager_churches` table and executes the `update_updated_at_column()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_prf_account_manager_churches_updated_at BEFORE UPDATE ON public.prf_account_manager_churches FOR EACH ROW EXECUTE FUNCTION update_updated_at_column()
```

## Associated Function

The `update_updated_at_column()` function is a generic utility function that sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`now()`). This is commonly used in `BEFORE UPDATE` triggers to automatically track the last modification time of a record and is documented [here](supabase/triggers/update_prf_ministry_departments_updated_at.md).

