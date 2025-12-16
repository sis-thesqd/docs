# Trigger: status_history_insert

This trigger fires `AFTER INSERT` on the `public.status_history` table and executes the `notify_webhook()` function. Its purpose is to send a notification to a PostgreSQL channel when a task's status changes to 'final files delivered'.

## Trigger Definition

```sql
CREATE TRIGGER status_history_insert AFTER INSERT ON public.status_history FOR EACH ROW EXECUTE FUNCTION notify_webhook()
```

## Function: notify_webhook()

This function is triggered after an insert into `public.status_history`. It checks if the `status_after` of the new record is 'final files delivered' (case-insensitive). If so, it constructs a JSON payload containing the `task_id`, `status_after`, and a descriptive message. This payload is then sent as a notification to a PostgreSQL channel named `webhook_channel`.

```sql
CREATE OR REPLACE FUNCTION public.notify_webhook()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
DECLARE
    webhook_url text := 'https://webhook.site/485b6fc0-65a8-4a54-af7e-6ee85ca581f8';
    payload json;
BEGIN
    -- Check if the status_after is 'final files delivered'
    IF LOWER(NEW.status_after) = 'final files delivered' THEN
        -- Construct the payload
        payload := json_build_object(
            'task_id', NEW.task_id,
            'status_after', NEW.status_after,
            'message', format('Task %s status changed to %s', NEW.task_id, NEW.status_after)
        );

        -- Send the webhook
        PERFORM pg_notify('webhook_channel', payload::text);
    END IF;

    RETURN NEW;
END;
$function$
```

