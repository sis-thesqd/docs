# Trigger: trigger_webhook_after_insert

This trigger fires `AFTER INSERT` on the `public.prf_integration_creds` table and executes the `trigger_webhook_on_prf_integration_creds_insert()` function. Its purpose is to send a webhook notification when new integration credentials are inserted.

## Trigger Definition

```sql
CREATE TRIGGER trigger_webhook_after_insert AFTER INSERT ON public.prf_integration_creds FOR EACH ROW EXECUTE FUNCTION trigger_webhook_on_prf_integration_creds_insert()
```

## Function: trigger_webhook_on_prf_integration_creds_insert()

This function is triggered after a new record is inserted into `public.prf_integration_creds`. It constructs a JSON payload containing the `user` and `app` from the new record and sends it as an HTTP POST request to an n8n webhook (`https://sis2.thesqd.com/webhook/backfill-pc-events`). This is typically used to initiate a backfill process for Planning Center events when new credentials are added.

```sql
CREATE OR REPLACE FUNCTION public.trigger_webhook_on_prf_integration_creds_insert()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
AS $function$
DECLARE
  payload jsonb;
BEGIN
  -- Create payload with only the user column
  payload := jsonb_build_object('user',NEW.user,'app',NEW.app);

  -- Send POST request to webhook
  PERFORM
    net.http_post(
      url := 'https://sis2.thesqd.com/webhook/backfill-pc-events',
      body := payload,
      headers := jsonb_build_object('Content-Type', 'application/json')
    );
  
  RETURN NEW;
END;
$function$
```



