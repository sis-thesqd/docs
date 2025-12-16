# Trigger: prf_client_completion_email_trigger

This trigger fires `AFTER UPDATE` on the `public.prf_project_submissions` table, specifically when the `end_time` column is updated. It executes the `prf_send_client_confirmation_email()` function to send a client completion email.

## Trigger Definition

```sql
CREATE TRIGGER prf_client_completion_email_trigger AFTER UPDATE OF end_time ON public.prf_project_submissions FOR EACH ROW EXECUTE FUNCTION prf_send_client_confirmation_email()
```

## Function: prf_send_client_confirmation_email()

This function is triggered when the `end_time` column of a `public.prf_project_submissions` record is updated (from NULL to a value, or to a different value). It sends an HTTP POST request to a specific n8n webhook (`https://sis2.thesqd.com/webhook/d7f9c87a-23c1-461f-8be1-905b079ba9ac`). The payload includes the `project_submission_id`, the new `end_time`, and the `updated_at` timestamp.

```sql
CREATE OR REPLACE FUNCTION public.prf_send_client_confirmation_email()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
begin
  if (old.end_time is null and new.end_time is not null) or 
     (old.end_time is not null and new.end_time <> old.end_time) then
    perform net.http_post(
      url := 'https://sis2.thesqd.com/webhook/d7f9c87a-23c1-461f-8be1-905b079ba9ac',
      headers := '{\"Content-Type\": \"application/json\"}'::jsonb,
      body := jsonb_build_object(
        'project_submission_id', new.project_submission_id,
        'end_time', new.end_time,
        'updated_at', current_timestamp
      )
    );
  end if;
  return new;
end;
$function$
```

