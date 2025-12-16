# Trigger: trigger_website_launched_update

This trigger fires `BEFORE UPDATE` on the `public.strategy_account_progress` table, specifically when the `website_launched` column is updated. It executes the `handle_website_launched_change()` function to automatically update `web_queue_status` and `website_launched_auto_status_applied` when a website is marked as launched.

## Trigger Definition

```sql
CREATE TRIGGER trigger_website_launched_update BEFORE UPDATE OF website_launched ON public.strategy_account_progress FOR EACH ROW EXECUTE FUNCTION handle_website_launched_change()
```

## Function: handle_website_launched_change()

This function is triggered before an update on the `public.strategy_account_progress` table when the `website_launched` column changes. If `website_launched_auto_status_applied` is false or NULL, and `website_launched` changes to true (from false or NULL), it updates `web_queue_status` to 'Ongoing Support' and sets `website_launched_auto_status_applied` to true to prevent further automatic updates for that status change.

```sql
CREATE OR REPLACE FUNCTION public.handle_website_launched_change()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  -- Only proceed if automation hasn't been applied yet
  IF (NEW.website_launched_auto_status_applied = false OR NEW.website_launched_auto_status_applied IS NULL) THEN
    
    -- Check if website_launched changed to true (from false or null)
    IF NEW.website_launched = true AND (OLD.website_launched = false OR OLD.website_launched IS NULL) THEN
      NEW.web_queue_status := 'Ongoing Support';
      NEW.website_launched_auto_status_applied := true;
    END IF;
    
  END IF;
  
  RETURN NEW;
END;
$function$
```

