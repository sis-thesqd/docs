# Trigger: trigger_contentsnare_status_update

This trigger fires `BEFORE UPDATE` on the `public.strategy_account_progress` table, specifically when the `contentsnare_status` column is updated. It executes the `handle_contentsnare_status_change()` function to automatically update `web_queue_status` and `contentsnare_auto_status_applied` based on the new `contentsnare_status`.

## Trigger Definition

```sql
CREATE TRIGGER trigger_contentsnare_status_update BEFORE UPDATE OF contentsnare_status ON public.strategy_account_progress FOR EACH ROW EXECUTE FUNCTION handle_contentsnare_status_change()
```

## Function: handle_contentsnare_status_change()

This function is triggered before an update on the `public.strategy_account_progress` table when the `contentsnare_status` changes. If `contentsnare_auto_status_applied` is false or NULL, it updates the `web_queue_status` to 'CC Pending' if `contentsnare_status` is 'waiting', or to 'Ready to Start' if `contentsnare_status` is 'completed'. It then sets `contentsnare_auto_status_applied` to true to prevent further automatic updates for that status change.

```sql
CREATE OR REPLACE FUNCTION public.handle_contentsnare_status_change()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  -- Only proceed if automation hasn't been applied yet
  IF NEW.contentsnare_auto_status_applied = false OR NEW.contentsnare_auto_status_applied IS NULL THEN
    
    -- Check if contentsnare_status matches our trigger conditions
    IF NEW.contentsnare_status = 'waiting' THEN
      NEW.web_queue_status := 'CC Pending';
      NEW.contentsnare_auto_status_applied := true;
      
    ELSIF NEW.contentsnare_status = 'completed' THEN
      NEW.web_queue_status := 'Ready to Start';
      NEW.contentsnare_auto_status_applied := true;
    END IF;
    
  END IF;
  
  RETURN NEW;
END;
$function$
```



