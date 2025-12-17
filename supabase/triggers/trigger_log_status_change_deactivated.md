# Trigger: trigger_log_status_change_deactivated

This trigger fires `AFTER INSERT` on the `public.status_history` table and executes the `log_status_change_deactivated()` function. Its purpose is to log changes when a task's status transitions from an active state to an inactive state.

## Trigger Definition

```sql
CREATE TRIGGER trigger_log_status_change_deactivated AFTER INSERT ON public.status_history FOR EACH ROW EXECUTE FUNCTION log_status_change_deactivated()
```

## Function: log_status_change_deactivated()

This function checks for status changes in `public.status_history` from an active state to an inactive state. If such a change occurs for a task within specific ClickUp spaces (1306092 or 1301552), it retrieves associated account information and inserts a log entry into `aa_decision_log` with the type `'deactivated'`.

```sql
CREATE OR REPLACE FUNCTION public.log_status_change_deactivated()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
DECLARE
    is_active_before BOOLEAN;
    is_active_after BOOLEAN;
    account_info JSONB;
BEGIN
    -- Check if the status_before is active
    SELECT active INTO is_active_before
    FROM clickup_statuses
    WHERE status = NEW.status_before;

    -- Check if the status_after is active
    SELECT active INTO is_active_after
    FROM clickup_statuses
    WHERE status = NEW.status_after;

    -- If status changed from active to not active, log the decision
    IF is_active_before = TRUE AND is_active_after = FALSE THEN
        -- Get the account information for the task only if it's in the specified spaces
        SELECT jsonb_build_object('account', cl.account, 'church_name', acc.church_name) INTO account_info
        FROM tasks t
        JOIN clickup_lists cl ON t.list_id = cl.id
        JOIN accounts acc ON cl.account = acc.account
        WHERE t.task_id = NEW.task_id
          AND cl.space IN (1306092, 1301552);  -- Check for the specified spaces

        -- Only insert if account_info is not null (task is in the specified spaces)
        IF account_info IS NOT NULL THEN
            -- Insert into aa_decision_log with updated execution_data
            INSERT INTO aa_decision_log (task_id, execution_data, type)
            VALUES (NEW.task_id, jsonb_set(to_jsonb(NEW), '{account_info}', account_info), 'deactivated');
        END IF;
    END IF;

    RETURN NEW;
END;
$function$
```



