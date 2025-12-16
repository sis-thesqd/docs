# Trigger: aa_log_queue_trigger_light

This trigger fires `AFTER INSERT OR UPDATE` on the `public.aa_log` table, specifically when the `status` of a new or updated row is 'queued'. It executes the `refresh_queue_numbers_light()` function to manage task queue numbers.

## Trigger Definition

```sql
CREATE TRIGGER aa_log_queue_trigger_light AFTER INSERT OR UPDATE ON public.aa_log FOR EACH ROW EXECUTE FUNCTION refresh_queue_numbers_light()
```

## Function: refresh_queue_numbers_light()

This function is triggered by changes to the `public.aa_log` table, specifically when a task's status is 'queued'. It updates the `queue_num` for tasks in a specific account's queue. It includes safeguards to skip processing if the queue is too large (more than 50 items) or if a timeout occurs, to prevent performance issues. The function aims to assign sequential queue numbers, prioritizing existing `queue_num` values and then `row_created` order.

```sql
CREATE OR REPLACE FUNCTION public.refresh_queue_numbers_light()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
DECLARE
    target_account BIGINT;
    queue_count INTEGER;
BEGIN
    -- Only proceed if the status is 'queued' for INSERT/UPDATE
    IF NOT ((TG_OP = 'INSERT' AND NEW.status = 'queued') OR 
            (TG_OP = 'UPDATE' AND (NEW.status = 'queued' OR OLD.status = 'queued'))) THEN
        RETURN COALESCE(NEW, OLD);
    END IF;
    
    -- Get target account
    target_account := CASE WHEN TG_OP = 'INSERT' THEN NEW.account ELSE NEW.account END;
    
    -- Check if there are too many queued items for this account (skip if > 50)
    SELECT COUNT(*) INTO queue_count
    FROM aa_log 
    WHERE status = 'queued' AND account = target_account;
    
    IF queue_count > 50 THEN
        -- Skip auto-refresh for large queues to prevent timeouts
        RAISE NOTICE 'Skipping queue refresh for account % - too many queued items (%)', target_account, queue_count;
        RETURN COALESCE(NEW, OLD);
    END IF;
    
    -- Only update if queue_num is NULL or needs reordering
    IF (TG_OP = 'INSERT' AND NEW.queue_num IS NULL) OR
       (TG_OP = 'UPDATE' AND (OLD.status != 'queued' OR NEW.status != 'queued')) THEN
        
        -- Set a very short timeout
        SET LOCAL statement_timeout = '2s';
        
        BEGIN
            -- Simple queue number assignment without complex joins
            UPDATE aa_log 
            SET queue_num = subq.new_queue_num
            FROM (
                SELECT 
                    task_id,
                    ROW_NUMBER() OVER (
                        ORDER BY 
                            CASE WHEN queue_num IS NOT NULL THEN 0 ELSE 1 END,
                            queue_num ASC,
                            row_created ASC
                    ) as new_queue_num
                FROM aa_log
                WHERE status = 'queued' AND account = target_account
            ) subq
            WHERE aa_log.task_id = subq.task_id
            AND aa_log.status = 'queued' 
            AND aa_log.account = target_account;
            
        EXCEPTION
            WHEN query_canceled OR others THEN
                -- If timeout or other error, just log and continue
                RAISE NOTICE 'Queue refresh skipped due to timeout for account %'`, target_account;
        END;
    END IF;
    
    RETURN COALESCE(NEW, OLD);
END;
$function$
```

