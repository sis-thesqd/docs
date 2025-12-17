# Trigger: n8n_trigger_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b

This trigger fires `AFTER INSERT` on the `public.prf_feedback` table and executes the `n8n_trigger_function_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b()` function. Its purpose is to notify an n8n webhook about new feedback submissions.

## Trigger Definition

```sql
CREATE TRIGGER n8n_trigger_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b AFTER INSERT ON public.prf_feedback FOR EACH ROW EXECUTE FUNCTION n8n_trigger_function_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b()
```

## Function: n8n_trigger_function_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b()

This function is a generic n8n trigger function. It sends a notification to a PostgreSQL channel named `n8n_channel_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b`, with the JSON representation of the new row (`NEW`) as the payload. This mechanism is typically used to trigger external workflows (e.g., in n8n) when data changes in the database.

```sql
CREATE OR REPLACE FUNCTION public.n8n_trigger_function_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$ begin perform pg_notify('n8n_channel_7d72a734_3dff_46a0_8e7a_5e84e02c7d2b', row_to_json(new)::text); return null; end; $function$
```



