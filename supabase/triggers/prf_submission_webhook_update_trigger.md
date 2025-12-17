# Trigger: prf_submission_webhook_update_trigger

This trigger fires `AFTER UPDATE` on the `public.prf_general_submissions` table and executes the `send_prf_submission_webhook()` function. Its purpose is to send a webhook notification when a general submission is updated, provided certain conditions related to folder numbers and existing project folders are met.

## Trigger Definition

```sql
CREATE TRIGGER prf_submission_webhook_update_trigger AFTER UPDATE ON public.prf_general_submissions FOR EACH ROW EXECUTE FUNCTION send_prf_submission_webhook()
```

## Associated Function

The `send_prf_submission_webhook()` function is also used by the `prf_submission_webhook_insert_trigger` trigger and is documented [here](supabase/triggers/prf_submission_webhook_insert_trigger.md).



