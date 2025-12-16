# Trigger: after_task_insert

This trigger fires `AFTER INSERT` on the `public.tasks` table and executes the `notify_new_task_webhook()` function. Its purpose is to send a webhook notification for newly inserted tasks, provided they are within specific ClickUp spaces and do not have any dependencies.

## Trigger Definition

```sql
CREATE TRIGGER after_task_insert AFTER INSERT ON public.tasks FOR EACH ROW EXECUTE FUNCTION notify_new_task_webhook()
```

## Function: notify_new_task_webhook()

This function is triggered after a new task is inserted into the `public.tasks` table. It retrieves the `space_id` associated with the task's `list_id` and checks if the task has any `depends_on` dependencies in the `task_dependencies` table. If the task is in one of the specified ClickUp spaces (1306092 or 1301552) and has no dependencies, it constructs a JSON payload with `task_id`, `task_name`, `space_id`, and `created_at` and sends it as an HTTP POST request to an n8n webhook (`https://sis2.thesqd.com/webhook/new-task`).

```sql
CREATE OR REPLACE FUNCTION public.notify_new_task_webhook()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
AS $function$
DECLARE
  space_id bigint;
  payload jsonb;
  header_value text;
  dependency_exists boolean;
BEGIN
  -- Get the space_id for the new task
  SELECT cs.id INTO space_id
  FROM clickup_lists cl
  JOIN clickup_spaces cs ON cl.space = cs.id
  WHERE cl.id = NEW.list_id;
  
  -- Check if the task has dependencies
  SELECT EXISTS (
    SELECT 1 
    FROM task_dependencies 
    WHERE task_id = NEW.task_id AND type = 'depends_on'
  ) INTO dependency_exists;
  
  -- Check if the task belongs to one of the specified space IDs and has no dependencies
  IF space_id IN (1306092, 1301552) AND NOT dependency_exists THEN
    -- Build the payload with the correct field names from your schema
    payload := jsonb_build_object(
      'task_id', NEW.task_id,
      'task_name', NEW.name,
      'space_id', space_id,
      'created_at', NEW.created_at
    );
    
    -- Content-Type header
    header_value := 'application/json';
    
    -- Send HTTP POST request to the webhook endpoint
    PERFORM net.http_post(
      url := 'https://sis2.thesqd.com/webhook/new-task',
      body := payload,
      headers := jsonb_build_object('Content-Type', header_value)
    );
  END IF;
  
  RETURN NEW;
END;
$function$
```

