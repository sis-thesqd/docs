# Trigger: ensure_plan_requirements_trigger

This trigger fires `BEFORE INSERT OR UPDATE` on the `public.prf_selection_types` table and executes the `ensure_all_plan_requirements()` function. Its purpose is to ensure that the `plan_requirements` JSONB column always has a default structure if it's null.

## Trigger Definition

```sql
CREATE TRIGGER ensure_plan_requirements_trigger BEFORE INSERT OR UPDATE ON public.prf_selection_types FOR EACH ROW EXECUTE FUNCTION ensure_all_plan_requirements()
```

## Function: ensure_all_plan_requirements()

This function checks if the `plan_requirements` column of the new or updated row in `prf_selection_types` is `NULL`. If it is, the function initializes `plan_requirements` with a default JSONB object containing various boolean flags for different service requirements, all set to `false`.

```sql
CREATE OR REPLACE FUNCTION public.ensure_all_plan_requirements()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    IF NEW.plan_requirements IS NULL THEN
        NEW.plan_requirements = jsonb_build_object(
            'requiresDesign', false,
            'requiresSocial', false,
            'requiresVideo', false,
            'requiresCreativeDirector', false,
            'requiresSms1', false,
            'requiresSms2', false,
            'requiresSms3', false,
            'requiresVideoOnly', false,
            'requiresWeb', false
        );
    END IF;
    RETURN NEW;
END;
$function$
```



