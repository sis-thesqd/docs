# Trigger: tag_history_time_estimate_processor_v2

This trigger fires `AFTER INSERT` on the `public.tag_history` table and executes the `time_estimate_processor_v2()` function.

## Trigger Definition

```sql
CREATE TRIGGER tag_history_time_estimate_processor_v2 AFTER INSERT ON public.tag_history FOR EACH ROW EXECUTE FUNCTION time_estimate_processor_v2()
```

## Associated Function

The `time_estimate_processor_v2()` function is also used by the `task_time_estimate_processor_v2` trigger and is documented [here](supabase/triggers/task_time_estimate_processor_v2.md).



