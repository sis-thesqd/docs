# Trigger: update_strategy_contractor_passcodes_updated_at

This trigger fires `BEFORE UPDATE` on the `public.strategy_contractor_passcodes` table and executes the `update_strategy_contractor_passcodes_updated_at()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_strategy_contractor_passcodes_updated_at BEFORE UPDATE ON public.strategy_contractor_passcodes FOR EACH ROW EXECUTE FUNCTION update_strategy_contractor_passcodes_updated_at()
```

## Function: update_strategy_contractor_passcodes_updated_at()

This function sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`now()`) before the update operation is applied to the `public.strategy_contractor_passcodes` table.

```sql
CREATE OR REPLACE FUNCTION public.update_strategy_contractor_passcodes_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$function$
```



