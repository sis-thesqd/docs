# Trigger: set_account_id_trigger

This trigger fires `BEFORE INSERT` on the `public.prf_project_groups` table and executes the `set_account_id_from_rls()` function. Its purpose is to set the `account_id` based on the current RLS (Row Level Security) settings.

## Trigger Definition

```sql
CREATE TRIGGER set_account_id_trigger BEFORE INSERT ON public.prf_project_groups FOR EACH ROW EXECUTE FUNCTION set_account_id_from_rls()
```

## Function: set_account_id_from_rls()

This function retrieves the `accid` from the current request query settings and sets it as the `account_id` for the new row being inserted. If `accid` is not present in the request query, `account_id` will be set to `NULL`.

```sql
CREATE OR REPLACE FUNCTION public.set_account_id_from_rls()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  NEW.account_id := NULLIF(current_setting('request.query.accid', true), '');
  RETURN NEW;
END;
$function$

