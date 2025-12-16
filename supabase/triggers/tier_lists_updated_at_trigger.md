# Trigger: tier_lists_updated_at_trigger

This trigger fires `BEFORE UPDATE` on the `public.tier_lists` table and executes the `update_tier_lists_updated_at()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER tier_lists_updated_at_trigger BEFORE UPDATE ON public.tier_lists FOR EACH ROW EXECUTE FUNCTION update_tier_lists_updated_at()
```

## Function: update_tier_lists_updated_at()

This function sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`now()`) before the update operation is applied to the `public.tier_lists` table.

```sql
CREATE OR REPLACE FUNCTION public.update_tier_lists_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$function$
```

