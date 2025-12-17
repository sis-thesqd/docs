# Trigger: update_sis_trigger_timestamp

This trigger fires `BEFORE UPDATE` on the `public.sis_triggers` table and executes the `update_sis_trigger_timestamp()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_sis_trigger_timestamp BEFORE UPDATE ON public.sis_triggers FOR EACH ROW EXECUTE FUNCTION update_sis_trigger_timestamp()
```

## Function: update_sis_trigger_timestamp()

This function is a utility function that sets the `updated_at` column of the new or updated row (`NEW`) to the `CURRENT_TIMESTAMP`. This is commonly used in `BEFORE UPDATE` triggers to automatically track the last modification time of a record.

```sql
CREATE OR REPLACE FUNCTION public.update_sis_trigger_timestamp()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$function$
```



