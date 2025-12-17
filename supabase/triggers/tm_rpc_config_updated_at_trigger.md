# Trigger: tm_rpc_config_updated_at_trigger

This trigger fires `BEFORE UPDATE` on the `public.tm_rpc_config` table and executes the `tm_rpc_config_updated_at()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER tm_rpc_config_updated_at_trigger BEFORE UPDATE ON public.tm_rpc_config FOR EACH ROW EXECUTE FUNCTION tm_rpc_config_updated_at()
```

## Function: tm_rpc_config_updated_at()

This function sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`now()`) before the update operation is applied to the `public.tm_rpc_config` table.

```sql
CREATE OR REPLACE FUNCTION public.tm_rpc_config_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$function$
```



