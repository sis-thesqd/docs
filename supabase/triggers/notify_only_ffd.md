# Trigger: notify_only_ffd

This trigger fires `AFTER INSERT` on the `public.status_history` table and executes the `notify_only_ffd()` function. Its purpose is to send a PostgreSQL notification specifically when a task's status changes to 'final files delivered'.

## Trigger Definition

```sql
CREATE TRIGGER notify_only_ffd AFTER INSERT ON public.status_history FOR EACH ROW EXECUTE FUNCTION notify_only_ffd()
```

## Function: notify_only_ffd()

This function checks if the `status_after` of a new `public.status_history` record is 'final files delivered'. If it is, the function sends a notification to a PostgreSQL channel named `notify_only_ffd` with the JSON representation of the new row (`NEW`) as the payload. This function ensures that notifications are sent only for this specific status change.

```sql
CREATE OR REPLACE FUNCTION public.notify_only_ffd()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  -- Only notify for the specific status we want
  IF NEW.status_after = 'final files delivered' THEN
    -- Use a completely unique channel name
    PERFORM pg_notify('notify_only_ffd', row_to_json(NEW)::text);
  END IF;
  RETURN NEW;
END;
$function$
```



