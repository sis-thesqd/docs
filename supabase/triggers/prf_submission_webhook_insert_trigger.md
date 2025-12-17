# Trigger: prf_submission_webhook_insert_trigger

This trigger fires `AFTER INSERT` on the `public.prf_general_submissions` table and executes the `send_prf_submission_webhook()` function. Its purpose is to send a webhook notification when a new general submission is inserted, provided certain conditions related to folder numbers and existing project folders are met.

## Trigger Definition

```sql
CREATE TRIGGER prf_submission_webhook_insert_trigger AFTER INSERT ON public.prf_general_submissions FOR EACH ROW EXECUTE FUNCTION send_prf_submission_webhook()
```

## Function: send_prf_submission_webhook()

This function is triggered after a new record is inserted into `public.prf_general_submissions`. It checks if `seq_folder_number` is not null or empty and if no corresponding entry exists in `project_folders` for the `submission_id`. If these conditions are met, it constructs a JSON payload with `submission_id`, `submission_type`, and the full row data, then sends it as an HTTP POST request to a specified webhook URL (`https://hkdk.events/jpeahoo62uwnbe`).

```sql
CREATE OR REPLACE FUNCTION public.send_prf_submission_webhook()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
AS $function$
DECLARE
    webhook_url TEXT := 'https://hkdk.events/jpeahoo62uwnbe';
    payload JSONB;
    http_response http_response;
BEGIN
    -- Check if criteria is met: seq_folder_number exists and no project_folder exists
    IF NEW.seq_folder_number IS NOT NULL 
       AND NEW.seq_folder_number != '' 
       AND NOT EXISTS (
           SELECT 1 FROM project_folders pf 
           WHERE pf.submission_id = NEW.submission_id
       ) THEN
        
        -- Build the payload
        payload := jsonb_build_object(
            'submission_id', NEW.submission_id,
            'submission_type', 'general',
            'row_data', row_to_json(NEW)::jsonb
        );
        
        -- Send the HTTP POST request using the http extension
        SELECT * INTO http_response FROM http_post(
            webhook_url,
            payload::text,
            'application/json'
        );
        
        -- Log the webhook attempt (optional)
        RAISE NOTICE 'Webhook sent for submission_id: %, status: %', NEW.submission_id, http_response.status;
        
    END IF;
    
    RETURN NEW;
END;
$function$
```



