# Trigger: log_sis_upvote_activity_trigger

This trigger fires `AFTER INSERT OR DELETE` on the `public.sis_problem_upvotes` table and executes the `log_sis_upvote_activity()` function. Its purpose is to log activity related to upvotes on SIS problems, including when an upvote is added or removed.

## Trigger Definition

```sql
CREATE TRIGGER log_sis_upvote_activity_trigger AFTER INSERT OR DELETE ON public.sis_problem_upvotes FOR EACH ROW EXECUTE FUNCTION log_sis_upvote_activity()
```

## Function: log_sis_upvote_activity()

This function logs upvote activity for SIS problems in the `sis_problem_activity` table. On `INSERT`, it logs an 'upvoted' event with details like `context_added` and `churches_represented`. On `DELETE`, it logs a 'removed_upvote' event. The `actor_clickup_id` is the `upvoter_clickup_id`.

```sql
CREATE OR REPLACE FUNCTION public.log_sis_upvote_activity()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO sis_problem_activity (problem_id, actor_clickup_id, action, details)
    VALUES (NEW.problem_id, NEW.upvoter_clickup_id, 'upvoted', 
            jsonb_build_object('context', NEW.context_added, 'churches', NEW.churches_represented));
  ELSIF TG_OP = 'DELETE' THEN
    INSERT INTO sis_problem_activity (problem_id, actor_clickup_id, action, details)
    VALUES (OLD.problem_id, OLD.upvoter_clickup_id, 'removed_upvote', '{}');
  END IF;
  RETURN COALESCE(NEW, OLD);
END;
$function$
```



