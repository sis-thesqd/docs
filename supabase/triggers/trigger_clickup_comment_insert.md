# Trigger: trigger_clickup_comment_insert

This trigger fires `AFTER INSERT` on the `public.clickup_comments` table and executes the `notify_new_clickup_comment()` function. Its purpose is to send an HTTP GET request to an n8n webhook whenever a new comment is inserted.

## Trigger Definition

```sql
CREATE TRIGGER trigger_clickup_comment_insert AFTER INSERT ON public.clickup_comments FOR EACH ROW EXECUTE FUNCTION notify_new_clickup_comment()
```

## Function: notify_new_clickup_comment()

This function is triggered after a new comment is inserted into the `public.clickup_comments` table. It performs an HTTP GET request to a specified n8n webhook (`https://sis1.thesqd.com/webhook-test/`) and appends the `task_id` of the new comment as a query parameter. This is typically used to trigger an external workflow in n8n based on new ClickUp comments.

```sql
CREATE OR REPLACE FUNCTION public.notify_new_clickup_comment()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
DECLARE
BEGIN
  -- Call the n8n webhook
  PERFORM
    http_get(
      'https://sis1.thesqd.com/webhook-test/?task_id=' || NEW.task_id
    );
  RETURN NEW;
END;
$function$
```



