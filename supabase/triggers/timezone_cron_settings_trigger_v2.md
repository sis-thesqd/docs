# Trigger: timezone_cron_settings_trigger_v2

This trigger fires `AFTER INSERT OR UPDATE` on the `public.timezone_snapshot_automation_settings` table and executes the `update_timezone_cron_jobs_on_settings_change_v2()` function. Its purpose is to dynamically update or reschedule timezone-aware cron jobs for automated workload snapshots whenever the settings in `timezone_snapshot_automation_settings` are changed.

## Trigger Definition

```sql
CREATE TRIGGER timezone_cron_settings_trigger_v2 AFTER INSERT OR UPDATE ON public.timezone_snapshot_automation_settings FOR EACH ROW EXECUTE FUNCTION update_timezone_cron_jobs_on_settings_change_v2()
```

## Function: update_timezone_cron_jobs_on_settings_change_v2()

This function is executed by the `timezone_cron_settings_trigger_v2`. It calls the `manage_timezone_snapshot_cron_jobs_v2()` function, which is responsible for dynamically creating, updating, or deleting cron jobs that trigger timezone-aware automated workload snapshots based on the latest settings in `timezone_snapshot_automation_settings`.

```sql
CREATE OR REPLACE FUNCTION public.update_timezone_cron_jobs_on_settings_change_v2()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    -- Call the enhanced function to manage timezone-aware cron jobs
    PERFORM manage_timezone_snapshot_cron_jobs_v2();
    RETURN NEW;
END;
$function$
```

## Function: manage_timezone_snapshot_cron_jobs_v2()

This function manages `pg_cron` jobs for automated timezone-aware workload snapshots. It retrieves settings from `timezone_snapshot_automation_settings`. It removes existing jobs and then creates new ones for each active timezone and specified snapshot time. Each local time is converted to UTC for cron scheduling, and an HTTP POST request is sent to the `automated-snapshots` Edge Function with timezone and time period details.

```sql
CREATE OR REPLACE FUNCTION public.manage_timezone_snapshot_cron_jobs_v2()
 RETURNS text
 LANGUAGE plpgsql
AS $function$
DECLARE
    settings_record RECORD;
    timezone_name TEXT;
    timezone_config JSONB;
    snapshot_time TEXT;
    job_name TEXT;
    cron_expression TEXT;
    utc_hour INTEGER;
    utc_minute INTEGER;
    hour_part INTEGER;
    minute_part INTEGER;
    sql_command TEXT;
    result_text TEXT := '';
    jobs_created INTEGER := 0;
    timezone_offsets JSONB := '{
        \"America/Los_Angeles\": -8,
        \"America/Denver\": -7, 
        \"America/Chicago\": -6,
        \"America/New_York\": -5
    }';
    timezone_offset INTEGER;
    is_enabled BOOLEAN;
    snapshot_times TEXT[];
BEGIN
    -- Get current timezone-aware settings
    SELECT * INTO settings_record 
    FROM timezone_snapshot_automation_settings 
    ORDER BY updated_at DESC 
    LIMIT 1;
    
    -- If no settings found, remove all timezone snapshot jobs
    IF NOT FOUND THEN
        PERFORM cron.unschedule(jobname) 
        FROM cron.job 
        WHERE jobname LIKE 'timezone-snapshot-%';
        
        result_text := 'No settings found - all cron jobs removed';
        RETURN result_text;
    END IF;
    
    -- Remove existing timezone snapshot jobs
    PERFORM cron.unschedule(jobname) 
    FROM cron.job 
    WHERE jobname LIKE 'timezone-snapshot-%';
    
    result_text := 'Removed existing jobs. Creating new jobs: ';
    
    -- Process each timezone configuration
    FOR timezone_name IN SELECT jsonb_object_keys(settings_record.timezone_configs)
    LOOP
        timezone_config := settings_record.timezone_configs -> timezone_name;
        is_enabled := (timezone_config ->> 'enabled')::BOOLEAN;
        snapshot_times := ARRAY(SELECT jsonb_array_elements_text(timezone_config -> 'times'));
        
        -- Skip if timezone is disabled
        IF NOT is_enabled THEN
            result_text := result_text || timezone_name || ' (disabled) ';
            CONTINUE;
        END IF;
        
        -- Get timezone offset
        timezone_offset := (timezone_offsets ->> timezone_name)::INTEGER;
        
        -- Create cron jobs for each time in this timezone
        FOREACH snapshot_time IN ARRAY snapshot_times
        LOOP
            -- Parse the local time (format: HH:MM)
            hour_part := CAST(split_part(snapshot_time, ':', 1) AS INTEGER);
            minute_part := CAST(split_part(snapshot_time, ':', 2) AS INTEGER);
            
            -- Convert local time to UTC for cron scheduling
            utc_hour := hour_part - timezone_offset;
            
            -- Handle day overflow/underflow
            IF utc_hour < 0 THEN
                utc_hour := utc_hour + 24;
            ELSIF utc_hour >= 24 THEN
                utc_hour := utc_hour - 24;
            END IF;
            
            -- Create cron expression (minute hour * * *)
            cron_expression := minute_part || ' ' || utc_hour || ' * * *';
            
            -- Create unique job name
            job_name := 'timezone-snapshot-' || 
                       replace(timezone_name, '/', '-') || '-' || 
                       replace(snapshot_time, ':', '');
            
            -- Determine time period for the snapshot
            DECLARE
                time_period TEXT;
            BEGIN
                IF hour_part = 9 THEN
                    time_period := 'morning';
                ELSIF hour_part = 12 THEN
                    time_period := 'midday';
                ELSIF hour_part = 17 THEN
                    time_period := 'evening';
                ELSE
                    time_period := 'other';
                END IF;
                
                -- Build the SQL command to call the edge function
                sql_command := 'SELECT net.http_post(
                    url:=''https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/automated-snapshots'',
                    headers:=''{\"Content-Type\": \"application/json\", \"Authorization\": \"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Ind0dGd3b3hsZXpxb3l6bWVzZWt0Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTcyMzc2MTg2NCwiZXhwIjoyMDM5MzM3ODY0fQ.KOPqP6qvn2xrLnYlh_ey5wkxJk-SnAgT6iFTU_lPWBg\"}''::jsonb,
                    body:=''{\"source\": \"timezone_cron_v2\", \"target_timezone\": \"' || timezone_name || '\", \"scheduled_time\": \"' || snapshot_time || '\", \"time_period\": \"' || time_period || '\"}''::jsonb
                ) as request_id;';
                
                -- Schedule the cron job
                PERFORM cron.schedule(job_name, cron_expression, sql_command);
                
                jobs_created := jobs_created + 1;
                result_text := result_text || timezone_name || '@' || snapshot_time || ' ';
            END;
        END LOOP;
    END LOOP;
    
    result_text := result_text || '- Total: ' || jobs_created || ' jobs';
    RETURN result_text;
END;
$function$
```



