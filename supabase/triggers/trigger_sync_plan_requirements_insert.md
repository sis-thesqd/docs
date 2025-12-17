# Trigger: trigger_sync_plan_requirements_insert

This trigger fires `AFTER INSERT` on the `public.prf_plans` table for each statement (not each row) and executes the `sync_plan_requirements()` function. Its purpose is to synchronize plan requirements across `prf_selection_types` based on the features defined in `prf_plans`.

## Trigger Definition

```sql
CREATE TRIGGER trigger_sync_plan_requirements_insert AFTER INSERT ON public.prf_plans FOR EACH STATEMENT EXECUTE FUNCTION sync_plan_requirements()
```

## Function: sync_plan_requirements()

This function is triggered after an insert operation on the `public.prf_plans` table. It extracts unique feature keys (starting with 'has') from the `plan_query` column of all active `prf_plans`. It then constructs a JSONB object with these features and updates the `plan_requirements` column in `prf_selection_types` for any rows where `plan_requirements` is currently `NULL` or an empty JSONB object. This ensures all `prf_selection_types` have a consistent set of plan requirement fields.

```sql
CREATE OR REPLACE FUNCTION public.sync_plan_requirements()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
DECLARE
    all_features JSONB;
BEGIN
    -- Extract unique features from all plan queries
    -- This assumes plan_query contains JSON with feature flags
    SELECT jsonb_object_agg(
        'requires' || key,
        false
    )
    INTO all_features
    FROM (
        SELECT DISTINCT jsonb_object_keys(plan_query::jsonb) as key
        FROM prf_plans
        WHERE active = true
        AND plan_query IS NOT NULL
        AND plan_query != ''
        AND plan_query::jsonb ? 'has'  -- Only get fields that start with 'has'
    ) features;

    -- Update all selection types with the new structure if needed
    UPDATE prf_selection_types
    SET plan_requirements = COALESCE(all_features, '{}'::jsonb)
    WHERE plan_requirements IS NULL OR plan_requirements = '{}'::jsonb;

    RETURN NULL;
END;
$function$
```



