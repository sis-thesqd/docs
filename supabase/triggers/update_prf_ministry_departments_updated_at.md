# Trigger: update_prf_ministry_departments_updated_at

This trigger fires `BEFORE UPDATE` on the `public.prf_ministry_departments` table and executes the `update_updated_at_column()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_prf_ministry_departments_updated_at BEFORE UPDATE ON public.prf_ministry_departments FOR EACH ROW EXECUTE FUNCTION update_updated_at_column()
```

## Function: update_updated_at_column()

This function is a generic utility function that sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`now()`). This is commonly used in `BEFORE UPDATE` triggers to automatically track the last modification time of a record. (Note: There is also a `storage.update_updated_at_column()` function, but this documentation refers to the public schema function).

```sql
CREATE OR REPLACE FUNCTION public.update_updated_at_column()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$function$
```

