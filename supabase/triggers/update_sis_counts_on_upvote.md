# Trigger: update_sis_counts_on_upvote

This trigger fires `AFTER INSERT, DELETE, OR UPDATE` on the `public.sis_problem_upvotes` table and executes the `update_sis_problem_counts()` function. Its purpose is to update the aggregated counts (upvote, unique submitters, church count) in the `sis_problems` table whenever an upvote record changes.

## Trigger Definition

```sql
CREATE TRIGGER update_sis_counts_on_upvote AFTER INSERT OR DELETE OR UPDATE ON public.sis_problem_upvotes FOR EACH ROW EXECUTE FUNCTION update_sis_problem_counts()
```

## Function: update_sis_problem_counts()

This function updates the `upvote_count`, `unique_submitter_count`, and `church_count` for a problem in the `sis_problems` table. It recalculates these counts based on related entries in `sis_problem_upvotes` and `sis_problems` itself, ensuring the aggregated data is always current. It is triggered by changes to `sis_problem_upvotes`.

```sql
CREATE OR REPLACE FUNCTION public.update_sis_problem_counts()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
  UPDATE sis_problems 
  SET 
    upvote_count = (SELECT COUNT(*) FROM sis_problem_upvotes WHERE problem_id = COALESCE(NEW.problem_id, OLD.problem_id)),
    unique_submitter_count = (
      SELECT COUNT(DISTINCT submitter) FROM (
        SELECT primary_submitter_clickup_id as submitter FROM sis_problems WHERE id = COALESCE(NEW.problem_id, OLD.problem_id)
        UNION
        SELECT jsonb_array_elements_text(co_submitters)::bigint FROM sis_problems WHERE id = COALESCE(NEW.problem_id, OLD.problem_id)
        UNION 
        SELECT upvoter_clickup_id FROM sis_problem_upvotes WHERE problem_id = COALESCE(NEW.problem_id, OLD.problem_id)
      ) submitters
    ),
    church_count = (
      SELECT COUNT(DISTINCT church) FROM (
        SELECT jsonb_array_elements_text(churches_affected) as church FROM sis_problems WHERE id = COALESCE(NEW.problem_id, OLD.problem_id)
        UNION
        SELECT jsonb_array_elements_text(churches_represented) FROM sis_problem_upvotes WHERE problem_id = COALESCE(NEW.problem_id, OLD.problem_id)
      ) churches
    ),
    updated_at = now()
  WHERE id = COALESCE(NEW.problem_id, OLD.problem_id);
  RETURN COALESCE(NEW, OLD);
END;
$function$
```

