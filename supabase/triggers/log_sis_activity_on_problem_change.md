# Trigger: log_sis_activity_on_problem_change

This trigger fires `AFTER INSERT OR UPDATE` on the `public.sis_problems` table and executes the `log_sis_problem_activity()` function. Its purpose is to log various activities related to SIS problems, such as creation, status changes, and assignment changes.

## Trigger Definition

```sql
CREATE TRIGGER log_sis_activity_on_problem_change AFTER INSERT OR UPDATE ON public.sis_problems FOR EACH ROW EXECUTE FUNCTION log_sis_problem_activity()
```

## Function: log_sis_problem_activity()

This function logs activity for SIS problems in the `sis_problem_activity` table. On `INSERT`, it logs a 'created' event with the problem description. On `UPDATE`, it logs 'status_changed' events if the status differs from the old record, and 'assigned' events if the `assigned_to_clickup_id` changes. The `actor_clickup_id` is derived from the new record's primary submitter or assigned user.

```sql
CREATE OR REPLACE FUNCTION public.log_sis_problem_activity()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO sis_problem_activity (problem_id, actor_clickup_id, action, details)
    VALUES (NEW.id, NEW.primary_submitter_clickup_id, 'created', jsonb_build_object('description', NEW.description));
    RETURN NEW;
  ELSIF TG_OP = 'UPDATE' THEN
    -- Log status changes
    IF OLD.status != NEW.status THEN
      INSERT INTO sis_problem_activity (problem_id, actor_clickup_id, action, details)
      VALUES (NEW.id, NEW.primary_submitter_clickup_id, 'status_changed', 
              jsonb_build_object('from', OLD.status, 'to', NEW.status));
    END IF;
    
    -- Log assignment changes
    IF OLD.assigned_to_clickup_id IS DISTINCT FROM NEW.assigned_to_clickup_id THEN
      INSERT INTO sis_problem_activity (problem_id, actor_clickup_id, action, details)
      VALUES (NEW.id, COALESCE(NEW.assigned_to_clickup_id, NEW.primary_submitter_clickup_id), 'assigned', 
              jsonb_build_object('from', OLD.assigned_to_clickup_id, 'to', NEW.assigned_to_clickup_id));
    END IF;
    
    RETURN NEW;
  END IF;
  RETURN NULL;
END;
$function$
```

