# Trigger: update_system_settings_timestamp

This trigger fires `BEFORE UPDATE` on the `public.prf_system_settings` table and executes the `update_modified_column()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_system_settings_timestamp BEFORE UPDATE ON public.prf_system_settings FOR EACH ROW EXECUTE FUNCTION update_modified_column()
```

## Function: update_modified_column()

This function is a generic utility function that sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`now()`). This is commonly used in `BEFORE UPDATE` triggers to automatically track the last modification time of a record.

```sql
CREATE OR REPLACE FUNCTION public.update_modified_column()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
   NEW.updated_at = now(); 
   RETURN NEW;
END;
$function$
```

