# Trigger: fill_account_on_insert

This trigger fires `AFTER INSERT` on the `public.designer_changes` table and executes the `update_account_from_master()` function. Its purpose is to populate the `account` field in `designer_changes` using data from `master_project_view`.

## Trigger Definition

```sql
CREATE TRIGGER fill_account_on_insert AFTER INSERT ON public.designer_changes FOR EACH ROW EXECUTE FUNCTION update_account_from_master()
```

## Function: update_account_from_master()

This function updates the `account` column in the `designer_changes` table for the newly inserted row (`NEW`). It retrieves the corresponding `account` from the `master_project_view` using the `task_id` and sets it in `designer_changes`.

```sql
CREATE OR REPLACE FUNCTION public.update_account_from_master()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  UPDATE designer_changes
  SET account = mpv.account
  FROM master_project_view mpv
  WHERE designer_changes.task_id = mpv.task_id
    AND designer_changes.id = NEW.id;
  RETURN NEW;
END;
$function$
```

