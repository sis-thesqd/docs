# Trigger: submission_updates_tier_list

This trigger fires `AFTER INSERT` on the `public.submissions` table and executes the `update_tier_list_on_submission()` function. Its purpose is to update the `updated_at` timestamp of the associated `tier_lists` entry whenever a new submission is made.

## Trigger Definition

```sql
CREATE TRIGGER submission_updates_tier_list AFTER INSERT ON public.submissions FOR EACH ROW EXECUTE FUNCTION update_tier_list_on_submission()
```

## Function: update_tier_list_on_submission()

This function updates the `updated_at` column of the `tier_lists` table, setting it to the current timestamp (`now()`). The update is performed on the `tier_list` record corresponding to the `tier_list_id` of the newly inserted submission (`NEW.tier_list_id`).

```sql
CREATE OR REPLACE FUNCTION public.update_tier_list_on_submission()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  UPDATE tier_lists SET updated_at = now() WHERE id = NEW.tier_list_id;
  RETURN NEW;
END;
$function$
```

