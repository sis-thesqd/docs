# Trigger: assignee_changes_trigger_jacob_n8n

This trigger fires after an `INSERT` operation on the `public.assignee_history` table and executes the `notify_task_changes_jacob_n8n()` function. Its purpose is to monitor and respond to changes in task assignments.

## Trigger Definition

```sql
CREATE TRIGGER assignee_changes_trigger_jacob_n8n AFTER INSERT ON public.assignee_history FOR EACH ROW EXECUTE FUNCTION notify_task_changes_jacob_n8n()
```

## Function: notify_task_changes_jacob_n8n()

This function is executed by the `assignee_changes_trigger_jacob_n8n` trigger. It calls the `check_and_notify_tasks_jacob_n8n()` function, passing the `task_id` from the newly inserted row in `assignee_history`.

```sql
CREATE OR REPLACE FUNCTION public.notify_task_changes_jacob_n8n()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    -- Call the check and notify function
    PERFORM check_and_notify_tasks_jacob_n8n(
        CASE 
            WHEN TG_TABLE_NAME = 'tasks' THEN NEW.task_id
            WHEN TG_TABLE_NAME = 'assignee_history' THEN NEW.task_id
            WHEN TG_TABLE_NAME = 'time_estimate_history' THEN NEW.task_id
            WHEN TG_TABLE_NAME = 'tag_history' THEN NEW.task_id
        END
    );
    RETURN NEW;
END;
$function$
```

## Function: check_and_notify_tasks_jacob_n8n(task_id_input text)

This function retrieves detailed task information based on a provided `task_id_input`. It filters tasks by various criteria including name, creation date, tag status, account details, and assignment status. If unassigned tasks matching the criteria are found, it aggregates the data into a JSON object and sends it to a specified webhook URL.

```sql
CREATE OR REPLACE FUNCTION public.check_and_notify_tasks_jacob_n8n(task_id_input text)
 RETURNS void
 LANGUAGE plpgsql
AS $function$DECLARE
    query_results json;
    webhook_url text := 'https://hkdk.events/pm3hmhtrwgr8q6';
BEGIN
    WITH task_base AS (
        SELECT t.task_id, t.name, t.list_id, t.row_created
        FROM public.tasks t
        WHERE t.task_id = task_id_input
            AND t.task_id !~ '_'
            AND LOWER(t.name) NOT IN ('u/d', 'u&d', 'social media plan', 'communications plan')
            AND t.created_at >= (CURRENT_TIMESTAMP - INTERVAL '2 days')
    ),
    latest_estimates AS (
        SELECT DISTINCT ON (task_id) 
            task_id, 
            estimate_mins_after
        FROM public.time_estimate_history
        WHERE estimate_mins_after IS NOT NULL
        ORDER BY task_id, created_at DESC
    ),
    latest_go_live AS (
        SELECT DISTINCT ON (task_id)
            task_id,
            go_live_date
        FROM public.go_live_dates
        ORDER BY task_id, created_at DESC
    ),
    task_tags AS (
        SELECT 
            task_id,
            array_agg(DISTINCT tag_after) AS tags
        FROM public.tag_history
        WHERE tag_after IS NOT NULL
        GROUP BY task_id
    ),
    task_department AS (
        SELECT 
            th.task_id,
            (
                SELECT ct2.department 
                FROM public.tag_history th2
                JOIN public.clickup_tags ct2 ON th2.tag_after = ct2.name
                WHERE th2.task_id = th.task_id 
                AND ct2.department IS NOT NULL
                LIMIT 1
            ) AS resp_dept
        FROM public.tag_history th
        GROUP BY th.task_id
    ),
    task_results AS (
        SELECT DISTINCT
            tb.task_id,
            tb.name,
            cl.account,
            a.high_usage,
            le.estimate_mins_after AS time_estimate,
            td.resp_dept,
            STRING_AGG(DISTINCT tg.grouping, ', ') AS tag_grouping,
            ct.min_days_future,
            ct.max_days_future,
            CASE 
                WHEN (lgl.go_live_date - CURRENT_DATE) > 29 
                     AND (lgl.go_live_date - CURRENT_DATE) <= 59 THEN TRUE
                WHEN (lgl.go_live_date - CURRENT_DATE) > 59 THEN TRUE
                ELSE FALSE
            END AS adjusted_go_live,
            NOT EXISTS (
                SELECT 1
                FROM public.assignee_history ah
                JOIN public.clickup_users cu ON ah.assignee = cu.clickup_id
                JOIN public.employees e ON cu.email = e.email
                WHERE ah.task_id = tb.task_id
            ) AS unassigned,
            tb.row_created
        FROM task_base tb
        JOIN public.clickup_lists cl ON tb.list_id = cl.id
        JOIN public.accounts a ON a.account = cl.account
        JOIN latest_estimates le ON le.task_id = tb.task_id
        LEFT JOIN latest_go_live lgl ON lgl.task_id = tb.task_id
        LEFT JOIN task_tags tt ON tt.task_id = tb.task_id
        LEFT JOIN task_department td ON td.task_id = tb.task_id
        LEFT JOIN public.tag_grouping tg ON tg.tag = ANY (tt.tags)
        LEFT JOIN public.clickup_tags ct ON ct.name = ANY (tt.tags)
        WHERE cl.account IS NOT NULL
        AND le.estimate_mins_after > 0
        AND LOWER(CAST(a.status AS TEXT)) != 'trial'
        AND a.auto_assign_override != TRUE
        AND tt.tags IS NOT NULL
        AND NOT EXISTS (
            SELECT 1 
            FROM public.product_history ph
            JOIN public.products p ON ph.product = p.name
            WHERE ph.account = cl.account 
            AND p.aa_exclude IS TRUE
        )
        AND NOT EXISTS (
            SELECT 1
            FROM UNNEST(tt.tags) AS tag
            JOIN public.clickup_tags ct2 ON tag = ct2.name
            WHERE ct2.aa_exclude IS TRUE
        )
        AND ct.min_days_future IS NOT NULL
        AND ct.max_days_future IS NOT NULL
        GROUP BY
            tb.task_id,
            tb.name,
            cl.account,
            a.high_usage,
            le.estimate_mins_after,
            td.resp_dept,
            ct.min_days_future,
            ct.max_days_future,
            lgl.go_live_date,
            tb.row_created
    )
    SELECT json_agg(task_results.*)
    INTO query_results
    FROM task_results
    WHERE unassigned = true;

    -- Only send webhook if there are results
    IF query_results IS NOT NULL AND query_results::text != '[]' THEN
            PERFORM http_post(
                webhook_url,
                query_results::text,
                'application/json'
            );
    END IF;
END;$function$



