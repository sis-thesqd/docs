# Trigger: webhook_trigger_project_submissions

This trigger fires `AFTER INSERT` on the `public.prf_project_submissions` table and executes the `send_webhook_on_project_submission()` function. Its purpose is to send a detailed webhook notification for new project submissions, specifically when `project_type` is 92.

## Trigger Definition

```sql
CREATE TRIGGER webhook_trigger_project_submissions AFTER INSERT ON public.prf_project_submissions FOR EACH ROW EXECUTE FUNCTION send_webhook_on_project_submission()
```

## Function: send_webhook_on_project_submission()

This function is triggered after a new record is inserted into `public.prf_project_submissions`. If the `project_type` of the new submission is 92, it constructs a comprehensive JSON payload containing all columns from the new record (`NEW`) and sends it as an HTTP POST request to a specific n8n webhook (`https://sis2.thesqd.com/webhook/0fiJ4xXBiYhkoDxW`). This is used to notify an external system about specific project submissions.

```sql
CREATE OR REPLACE FUNCTION public.send_webhook_on_project_submission() RETURNS trigger LANGUAGE plpgsql SECURITY DEFINER AS $function$
BEGIN
    -- Check if project_type equals 92
    IF NEW.project_type = 92 THEN
        -- Perform the HTTP POST request
        PERFORM
            net.http_post(
                url := 'https://sis2.thesqd.com/webhook/0fiJ4xXBiYhkoDxW',
                headers := '{\"Content-Type\": \"application/json\"}'::jsonb,
                body := jsonb_build_object(
                    'project_submission_id', NEW.project_submission_id,
                    'gis_id', NEW.gis_id,
                    'account_id', NEW.account_id,
                    'user_id', NEW.user_id,
                    'start_time', NEW.start_time,
                    'end_time', NEW.end_time,
                    'raw_data', NEW.raw_data,
                    'created_at', NEW.created_at,
                    'updated_at', NEW.updated_at,
                    'project_type', NEW.project_type,
                    'project_name', NEW.project_name,
                    'editable', NEW.editable,
                    'edit_id', NEW.edit_id,
                    'hidden', NEW.hidden,
                    'completion_target', NEW.completion_target,
                    'clickup_id', NEW.clickup_id,
                    'is_primary', NEW.is_primary,
                    'files_processed', NEW.files_processed,
                    'dropbox_folder_id', NEW.dropbox_folder_id,
                    'dropbox_folder_path', NEW.dropbox_folder_path,
                    'task_content', NEW.task_content,
                    'designer_selection', NEW.designer_selection,
                    'submitted_by_am', NEW.submitted_by_am,
                    'submission_method', NEW.submission_method,
                    'follow_brand', NEW.follow_brand,
                    'files_processed_2', NEW.files_processed_2,
                    'dropbox_folder_id_2', NEW.dropbox_folder_id_2,
                    'dropbox_folder_path_2', NEW.dropbox_folder_path_2,
                    'dropbox_folder_share_link', NEW.dropbox_folder_share_link,
                    'is_task_dependent', NEW.is_task_dependent
                )
            );
    END IF;
    
    RETURN NEW;
END;
$function$
```



