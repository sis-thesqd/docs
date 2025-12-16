# Trigger: account_activity_trigger

This trigger fires `AFTER INSERT OR UPDATE` on the `public.igniter_accounts` table and executes the `noop_function()` function. It is likely a placeholder or a trigger intended to be enabled/disabled as needed, as the associated function performs no operations.

## Trigger Definition

```sql
CREATE TRIGGER account_activity_trigger AFTER INSERT OR UPDATE ON public.igniter_accounts FOR EACH ROW EXECUTE FUNCTION noop_function()
```

## Function: noop_function()

This is a no-operation (noop) function. It simply returns the `NEW` row without making any modifications. This type of function is often used as a placeholder or when a trigger is required but no specific action needs to be performed.

```sql
CREATE OR REPLACE FUNCTION public.noop_function()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  RETURN NEW;
END;
$function$
```

