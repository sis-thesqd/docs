# Trigger: on_project_submission_insert

This trigger fires `AFTER INSERT` on the `public.prf_project_submissions` table and executes the `notify_project_submission()` function. While the function name suggests a notification, its current implementation is a no-operation, implying it might be a placeholder or the notification logic is handled elsewhere.

## Trigger Definition

```sql
CREATE TRIGGER on_project_submission_insert AFTER INSERT ON public.prf_project_submissions FOR EACH ROW EXECUTE FUNCTION notify_project_submission()
```

## Function: notify_project_submission()

This function currently performs no operation. It simply returns the `NEW` row without any modifications or side effects. It might be a placeholder for future notification logic.

```sql
CREATE OR REPLACE FUNCTION public.notify_project_submission()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$BEGIN
RETURN NEW;
END;$function$
```



