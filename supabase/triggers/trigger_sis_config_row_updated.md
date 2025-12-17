# Trigger: trigger_sis_config_row_updated

This trigger fires `BEFORE UPDATE` on the `public.sis_config` table and executes the `update_sis_config_row_updated()` function. Its purpose is to automatically set the `row_updated` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER trigger_sis_config_row_updated BEFORE UPDATE ON public.sis_config FOR EACH ROW EXECUTE FUNCTION update_sis_config_row_updated()
```

## Function: update_sis_config_row_updated()

This function sets the `row_updated` column of the new or updated row (`NEW`) to the current timestamp (`now()`) before the update operation is applied to the `public.sis_config` table.

```sql
CREATE OR REPLACE FUNCTION public.update_sis_config_row_updated()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.row_updated = now();
    RETURN NEW;
END;
$function$
```



