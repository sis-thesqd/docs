# Trigger: snapshot_settings_change_trigger

This trigger fires `AFTER INSERT OR UPDATE` on the `public.snapshot_automation_settings` table and executes the `update_cron_jobs_on_settings_change()` function. Its purpose is to dynamically update or reschedule cron jobs that perform automated workload snapshots whenever the settings in `snapshot_automation_settings` are changed.

## Trigger Definition

```sql
CREATE TRIGGER snapshot_settings_change_trigger AFTER INSERT OR UPDATE ON public.snapshot_automation_settings FOR EACH ROW EXECUTE FUNCTION update_cron_jobs_on_settings_change()
```

## Function: update_cron_jobs_on_settings_change()

This function is executed by the `snapshot_settings_change_trigger`. It calls the `manage_snapshot_cron_jobs()` function, which is responsible for dynamically creating, updating, or deleting cron jobs that trigger automated workload snapshots based on the latest settings in `snapshot_automation_settings`.

```sql
CREATE OR REPLACE FUNCTION public.update_cron_jobs_on_settings_change()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    -- Call the function to manage cron jobs
    PERFORM manage_snapshot_cron_jobs();
    RETURN NEW;
END;
$function$
```

## Function: manage_snapshot_cron_jobs()

This function manages the `pg_cron` jobs for automated workload snapshots. It retrieves the latest settings from `snapshot_automation_settings`. If automation is inactive or no settings are found, all existing snapshot cron jobs are unscheduled. Otherwise, it unschedules existing jobs and creates new ones for each specified interval, converting the times to UTC for cron scheduling. Each cron job makes an HTTP POST request to the `automated-snapshots` Edge Function.

```sql
CREATE OR REPLACE FUNCTION public.manage_snapshot_cron_jobs()
 RETURNS void
 LANGUAGE plpgsql
AS $function$
DECLARE
    settings_record RECORD;
    interval_time TEXT;
    job_name TEXT;
    cron_expression TEXT;
    hour_part INTEGER;
    minute_part INTEGER;
    sql_command TEXT;
BEGIN
    -- Get current settings
    SELECT * INTO settings_record 
    FROM snapshot_automation_settings 
    ORDER BY updated_at DESC 
    LIMIT 1;
    
    -- If no settings found or automation is not active, remove all snapshot jobs
    IF NOT FOUND OR NOT settings_record.is_active THEN
        -- Remove any existing snapshot cron jobs
        PERFORM cron.unschedule(jobname) 
        FROM cron.job 
        WHERE jobname LIKE 'dynamic-workload-snapshot-%';
        RETURN;
    END IF;
    
    -- Remove existing dynamic snapshot jobs
    PERFORM cron.unschedule(jobname) 
    FROM cron.job 
    WHERE jobname LIKE 'dynamic-workload-snapshot-%';
    
    -- Create new cron jobs for each interval
    FOREACH interval_time IN ARRAY settings_record.intervals
    LOOP
        -- Parse the time (format: HH:MM)
        hour_part := CAST(split_part(interval_time, ':', 1) AS INTEGER);
        minute_part := CAST(split_part(interval_time, ':', 2) AS INTEGER);
        
        -- Convert Eastern time to UTC for cron scheduling
        -- Eastern Standard Time (EST) is UTC-5, Eastern Daylight Time (EDT) is UTC-4
        -- For simplicity, we'll add 5 hours (assuming EST) - this should be adjusted for DST
        hour_part := (hour_part + 5) % 24;
        
        -- Create cron expression (minute hour * * *)
        cron_expression := minute_part || ' ' || hour_part || ' * * *';
        
        -- Create unique job name
        job_name := 'dynamic-workload-snapshot-' || replace(interval_time, ':', '');
        
        -- Build the SQL command
        sql_command := 'SELECT net.http_post(
            url:=''https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/automated-snapshots'',
            headers:=''{\"Content-Type\": \"application/json\", \"Authorization\": \"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Ind0dGd3b3hsZXpxb3l6bWVzZWt0Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTcyMzc2MTg2NCwiZXhwIjoyMDM5MzM3ODY0fQ.KOPqP6qvn2xrLnYlh_ey5wkxJk-SnAgT6iFTU_lPWBg\"}''::jsonb,
            body:=''{\"source\": \"cron\", \"scheduled_time\": \"' || interval_time || '\"}''::jsonb
        ) as request_id;';
        
        -- Schedule the cron job
        PERFORM cron.schedule(job_name, cron_expression, sql_command);
        
    END LOOP;
END;
$function$
```



