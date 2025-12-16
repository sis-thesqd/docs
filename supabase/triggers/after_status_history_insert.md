# Trigger: after_status_history_insert

This trigger fires `AFTER INSERT` on the `public.status_history` table and executes the `notify_status_activation_webhook()` function. Its purpose is to detect status changes from inactive to active and send a notification to a webhook if certain conditions are met.

## Trigger Definition

```sql
CREATE TRIGGER after_status_history_insert AFTER INSERT ON public.status_history FOR EACH ROW EXECUTE FUNCTION notify_status_activation_webhook()
```

## Function: notify_status_activation_webhook()

This function is triggered after an insert into `public.status_history`. It checks if a task's status is changing from an inactive to an active state. If this occurs within specific ClickUp spaces (1301552 or 1306092) and the associated account has a `room_value` less than or equal to 0 in `active_projects_mv`, a JSON payload with task and user details is sent via HTTP POST to an n8n webhook (`https://sis2.thesqd.com/webhook/status-activation`). Changes made by a specific user ID (`4463869`) are skipped.

```sql
CREATE OR REPLACE FUNCTION public.notify_status_activation_webhook()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
AS $function$
DECLARE
  status_before_active boolean;
  status_after_active boolean;
  task_list_id bigint;
  account_id bigint;
  space_id bigint;
  room_value int;
  payload jsonb;
  header_value text;
  user_name text;
  user_id text;
BEGIN
  -- Skip if the change was made by the specific user ID
  IF NEW.changed_by = '4463869' THEN
    RETURN NEW;
  END IF;
  
  -- Modify the queries to ensure only one row is returned
  -- Use LIMIT 1 to ensure only one row is returned
  SELECT active INTO status_before_active
  FROM clickup_statuses 
  WHERE LOWER(status) = LOWER(NEW.status_before)
  LIMIT 1;
  
  SELECT active INTO status_after_active
  FROM clickup_statuses 
  WHERE LOWER(status) = LOWER(NEW.status_after)
  LIMIT 1;

  -- Continue only if we're changing from inactive to active status
  IF status_before_active = false AND status_after_active = true THEN
    -- Get the list_id from the tasks table
    SELECT list_id INTO task_list_id
    FROM tasks
    WHERE task_id = NEW.task_id;
    
    -- Get the space and account from the clickup_lists table
    SELECT space, account INTO space_id, account_id
    FROM clickup_lists
    WHERE id = task_list_id;
    
    -- Only proceed if the space matches one of the specified spaces
    IF space_id = 1301552 OR space_id = 1306092 THEN
      -- Check if account exists in active_projects_mv and get room value
      SELECT room INTO room_value
      FROM active_projects_mv
      WHERE account = account_id;
      
      -- Get the username and clickup_id of the user who made the change
      -- Add explicit type cast to changed_by
      SELECT username, clickup_id INTO user_name, user_id
      FROM clickup_users
      WHERE clickup_id::text = NEW.changed_by;
      
      -- Only proceed if the room value is less than or equal to 0
      IF room_value IS NOT NULL AND room_value <= 0 THEN
        -- Build the payload
        payload := jsonb_build_object(
          'task_id', NEW.task_id,
          'status_before', NEW.status_before,
          'status_after', NEW.status_after,
          'account_id', account_id,
          'room_value', room_value,
          'changed_at', NEW.changed_at,
          'user_name', user_name,
          'user_id', user_id
        );
        
        -- Content-Type header
        header_value := 'application/json';
        
        -- Send HTTP POST request to the webhook endpoint
        PERFORM net.http_post(
          url := 'https://sis2.thesqd.com/webhook/status-activation',
          body := payload,
          headers := jsonb_build_object('Content-Type', header_value)
        );
      END IF;
    END IF;
  END IF;
  
  RETURN NEW;
END;
$function$
```

