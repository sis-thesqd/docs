# Trigger: update_feedback_popup_shown_updated_at

This trigger fires `BEFORE UPDATE` on the `public.prf_feedback_popup_shown` table and executes the `update_prf_feedback_popup_shown_updated_at()` function. Its purpose is to automatically set the `updated_at` timestamp for the row being updated.

## Trigger Definition

```sql
CREATE TRIGGER update_feedback_popup_shown_updated_at BEFORE UPDATE ON public.prf_feedback_popup_shown FOR EACH ROW EXECUTE FUNCTION update_prf_feedback_popup_shown_updated_at()
```

## Function: update_prf_feedback_popup_shown_updated_at()

This function sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`NOW()`) before the update operation is applied to the `public.prf_feedback_popup_shown` table.

```sql
CREATE OR REPLACE FUNCTION public.update_prf_feedback_popup_shown_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$function$
```



