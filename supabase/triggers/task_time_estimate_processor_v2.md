# Trigger: task_time_estimate_processor_v2

This trigger is responsible for processing task time estimates. It fires after an `INSERT` or `UPDATE` operation on the `public.tasks` table and executes the `time_estimate_processor_v2()` function.

## Trigger Definition

```sql
CREATE TRIGGER task_time_estimate_processor_v2 AFTER INSERT OR UPDATE ON public.tasks FOR EACH ROW EXECUTE FUNCTION time_estimate_processor_v2()
```

## Function: time_estimate_processor_v2() (No Arguments)

This function is executed by the trigger. It calls an overloaded version of `time_estimate_processor_v2` with the `task_id` from the newly inserted or updated row in either `tasks` or `tag_history` tables.

```sql
CREATE OR REPLACE FUNCTION public.time_estimate_processor_v2()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    -- Call the check and notify function
    PERFORM time_estimate_processor_v2(
        CASE
            WHEN TG_TABLE_NAME = 'tasks' THEN NEW.task_id
            WHEN TG_TABLE_NAME = 'tag_history' THEN NEW.task_id
        END
    );
    RETURN NEW;
END;
$function$
```

## Function: time_estimate_processor_v2(task_id_input text)

This function retrieves task data based on specific conditions, including task creation date, tag status, exclusion flags, and ClickUp space IDs. If matching tasks are found, it sends this data as a JSON payload to a specified webhook URL.

```sql
CREATE OR REPLACE FUNCTION public.time_estimate_processor_v2(task_id_input text)
 RETURNS void
 LANGUAGE plpgsql
AS $function$DECLARE
    task_data json;
    webhook_url text := 'https://hkdk.events/cum8jgusyfp155';
BEGIN
    -- Get the task data that matches our conditions
    SELECT json_agg(task_results.*)
    INTO task_data
    FROM (
        SELECT DISTINCT
            t.task_id,
            t.name,
            cf.account,
            t.created_at,
            th.tag_after,
            t.row_created
        FROM tasks t
        JOIN tag_history th ON t.task_id = th.task_id
        JOIN clickup_tags ct ON th.tag_after = ct.name
        JOIN clickup_lists cl ON t.list_id = cl.id
        JOIN clickup_folders cf ON cl.folder = cf.id
        LEFT JOIN LATERAL (
            SELECT estimate_mins_after
            FROM time_estimate_history
            WHERE task_id = t.task_id
            ORDER BY created_at DESC
            LIMIT 1
        ) teh ON true
        WHERE t.task_id = task_id_input
        AND DATE(t.row_created) IN (CURRENT_DATE - 1, CURRENT_DATE)
        AND ct.status = 'Active'
        AND ct.aa_exclude IS NOT TRUE
        AND cl.space IN (1301552, 1306092)
        AND teh.estimate_mins_after IS NULL
        AND th.tag_after IS NOT NULL
    ) task_results;

    -- Only send webhook if there are results
    IF task_data IS NOT NULL AND task_data::text != '[]' THEN
        PERFORM http_post(
            webhook_url,
            task_data::text,
            'application/json'
        );
    END IF;
END;$function$



