# Trigger: social_signup_webhook

This trigger fires `AFTER INSERT OR UPDATE` on the `public.product_history` table. It executes the `notify_social_signup()` function, which sends a webhook notification when a social product is signed up for or updated.

## Trigger Definition

```sql
CREATE TRIGGER social_signup_webhook AFTER INSERT OR UPDATE ON public.product_history FOR EACH ROW EXECUTE FUNCTION notify_social_signup()
```

## Function: notify_social_signup()

This function checks if a new or updated product in `product_history` is a social media product (`'social pro'`, `'social plus'`, or `'social starter'`) and is currently active. If these conditions are met, it sends a JSON payload containing details about the account, product, and timestamps to a specified n8n webhook URL (`https://sis2.thesqd.com/webhook/sms-new-signup`). This is used to notify an external system about social product sign-ups.

```sql
CREATE OR REPLACE FUNCTION public.notify_social_signup()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
AS $function$
begin
  if (
    lower(NEW.product) in ('social pro', 'social plus', 'social starter') 
    and NEW.current = true
  ) then
    perform net.http_post(
      'https://sis2.thesqd.com/webhook/sms-new-signup',
      jsonb_build_object(
        'account', NEW.account,
        'product', NEW.product,
        'created_at', NEW.created_at,
        'id', NEW.id,
        'current',NEW.current,
        'previous_product', COALESCE(OLD.product, 'none'),
        'previous_current', COALESCE(OLD.current, false)
      ),
      jsonb_build_object(
        'Content-Type', 'application/json'
      )
    );
  end if;
  return NEW;
end;
$function$
```

