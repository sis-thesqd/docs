# Trigger: validate_location_mapping_trigger

This trigger fires `BEFORE INSERT OR UPDATE` on the `public.location_mappings` table and executes the `validate_location_mapping()` function. Its purpose is to ensure that the `object_id` provided in a `location_mappings` entry corresponds to a valid ID for the specified `location_type` (folder, list, or space).

## Trigger Definition

```sql
CREATE TRIGGER validate_location_mapping_trigger BEFORE INSERT OR UPDATE ON public.location_mappings FOR EACH ROW EXECUTE FUNCTION validate_location_mapping()
```

## Function: validate_location_mapping()

This function validates the `object_id` in the `public.location_mappings` table based on its `location_type`. It checks if the `object_id` exists in the corresponding `clickup_folders`, `clickup_lists`, or `clickup_spaces` table. If the `object_id` is not found for the given `location_type`, it raises an exception.

```sql
CREATE OR REPLACE FUNCTION public.validate_location_mapping()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    CASE NEW.location_type
        WHEN 'folder' THEN
            IF NOT EXISTS (SELECT 1 FROM clickup_folders WHERE id = NEW.object_id) THEN
                RAISE EXCEPTION 'Invalid folder ID: %', NEW.object_id;
            END IF;
        WHEN 'list' THEN
            IF NOT EXISTS (SELECT 1 FROM clickup_lists WHERE id = NEW.object_id) THEN
                RAISE EXCEPTION 'Invalid list ID: %', NEW.object_id;
            END IF;
        WHEN 'space' THEN
            IF NOT EXISTS (SELECT 1 FROM clickup_spaces WHERE id = NEW.object_id) THEN
                RAISE EXCEPTION 'Invalid space ID: %', NEW.object_id;
            END IF;
    END CASE;
    RETURN NEW;
END;
$function$
```

