# Trigger: trigger_sync_plan_requirements_update

This trigger fires `AFTER UPDATE` of the `plan_query` column on the `public.prf_plans` table for each statement (not each row) and executes the `sync_plan_requirements()` function. Its purpose is to synchronize plan requirements across `prf_selection_types` based on the updated features defined in `prf_plans`.

## Trigger Definition

```sql
CREATE TRIGGER trigger_sync_plan_requirements_update AFTER UPDATE OF plan_query ON public.prf_plans FOR EACH STATEMENT EXECUTE FUNCTION sync_plan_requirements()
```

## Associated Function

The `sync_plan_requirements()` function is also used by the `trigger_sync_plan_requirements_insert` trigger and is documented [here](supabase/triggers/trigger_sync_plan_requirements_insert.md).



