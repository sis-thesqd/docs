# Trigger: time_estimate_changes_trigger_jacob_n8n

This trigger fires `AFTER INSERT` on the `public.time_estimate_history` table and executes the `notify_task_changes_jacob_n8n()` function.

## Trigger Definition

```sql
CREATE TRIGGER time_estimate_changes_trigger_jacob_n8n AFTER INSERT ON public.time_estimate_history FOR EACH ROW EXECUTE FUNCTION notify_task_changes_jacob_n8n()
```

## Associated Function

The `notify_task_changes_jacob_n8n()` function is also used by the `assignee_changes_trigger_jacob_n8n` trigger and `task_changes_trigger_jacob_n8n` trigger, and is documented [here](supabase/triggers/assignee_changes_trigger_jacob_n8n.md).



