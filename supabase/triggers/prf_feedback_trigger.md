# Trigger: prf_feedback_trigger

This trigger fires `AFTER INSERT` on the `public.prf_feedback` table and executes the `n8n_trigger_function_bf2fc0f0_fd47_49af_a46e_496ee0b87a69()` function. Its purpose is to notify an n8n webhook about new feedback submissions.

## Trigger Definition

```sql
CREATE TRIGGER prf_feedback_trigger AFTER INSERT ON public.prf_feedback FOR EACH ROW EXECUTE FUNCTION n8n_trigger_function_bf2fc0f0_fd47_49af_a46e_496ee0b87a69()
```

## Function: n8n_trigger_function_bf2fc0f0_fd47_49af_a46e_496ee0b87a69()

This function is a generic n8n trigger function. It sends a notification to a PostgreSQL channel named `n8n_channel_bf2fc0f0_fd47_49af_a46e_496ee0b87a69`, with the JSON representation of the new row (`NEW`) as the payload. This mechanism is typically used to trigger external workflows (e.g., in n8n) when data changes in the database.

```sql
CREATE OR REPLACE FUNCTION public.n8n_trigger_function_bf2fc0f0_fd47_49af_a46e_496ee0b87a69()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$ begin perform pg_notify('n8n_channel_bf2fc0f0_fd47_49af_a46e_496ee0b87a69', row_to_json(new)::text); return null; end; $function$
```



