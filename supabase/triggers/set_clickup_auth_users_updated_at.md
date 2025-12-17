# Trigger: set_clickup_auth_users_updated_at

This trigger fires `BEFORE UPDATE` on the `public.clickup_auth_users` table and executes the `update_clickup_auth_users_updated_at()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER set_clickup_auth_users_updated_at BEFORE UPDATE ON public.clickup_auth_users FOR EACH ROW EXECUTE FUNCTION update_clickup_auth_users_updated_at()
```

## Function: update_clickup_auth_users_updated_at()

This function sets the `updated_at` column of the new row (`NEW`) to the current timestamp (`now()`) before the update operation is applied to the `public.clickup_auth_users` table.

```sql
CREATE OR REPLACE FUNCTION public.update_clickup_auth_users_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$function$
```



