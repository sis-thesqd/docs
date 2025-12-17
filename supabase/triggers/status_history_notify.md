# Trigger: status_history_notify

This trigger fires `AFTER INSERT OR UPDATE` on the `public.status_history` table and executes the `notify_webhook()` function.

## Trigger Definition

```sql
CREATE TRIGGER status_history_notify AFTER INSERT OR UPDATE ON public.status_history FOR EACH ROW EXECUTE FUNCTION notify_webhook()
```

## Associated Function

The `notify_webhook()` function is also used by the `status_history_insert` trigger and is documented [here](supabase/triggers/status_history_insert.md).



