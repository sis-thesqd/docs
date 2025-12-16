# Trigger: update_prf_general_submissions_timestamp

This trigger fires `BEFORE UPDATE` on the `public.prf_general_submissions` table and executes the `update_timestamp()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_prf_general_submissions_timestamp BEFORE UPDATE ON public.prf_general_submissions FOR EACH ROW EXECUTE FUNCTION update_timestamp()
```

## Function: update_timestamp()

This function is a generic utility function that sets the `updated_at` column of the new or updated row (`NEW`) to the `CURRENT_TIMESTAMP`. This is commonly used in `BEFORE UPDATE` triggers to automatically track the last modification time of a record.

```sql
CREATE OR REPLACE FUNCTION public.update_timestamp()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$function$
```

