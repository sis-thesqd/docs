# Trigger: task_changes_trigger_jacob_n8n

This trigger fires `AFTER INSERT OR UPDATE` on the `public.tasks` table and executes the `notify_task_changes_jacob_n8n()` function.

## Trigger Definition

```sql
CREATE TRIGGER task_changes_trigger_jacob_n8n AFTER INSERT OR UPDATE ON public.tasks FOR EACH ROW EXECUTE FUNCTION notify_task_changes_jacob_n8n()
```

## Associated Function

The `notify_task_changes_jacob_n8n()` function is also used by the `assignee_changes_trigger_jacob_n8n` trigger and is documented [here](supabase/triggers/assignee_changes_trigger_jacob_n8n.md).

