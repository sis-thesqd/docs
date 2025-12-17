# Trigger: update_server_shorthand_trigger

This trigger fires `BEFORE INSERT OR UPDATE` on the `public.sis_workflows` table, specifically when the `server` column is updated. It executes the `update_server_shorthand_func()` function to populate the `server_shorthand` column.

## Trigger Definition

```sql
CREATE TRIGGER update_server_shorthand_trigger BEFORE INSERT OR UPDATE OF server ON public.sis_workflows FOR EACH ROW EXECUTE FUNCTION update_server_shorthand_func()
```

## Function: update_server_shorthand_func()

This function automatically generates a shorthand name for the `server_shorthand` column based on the value in the `server` column of the `public.sis_workflows` table. It uses a `CASE` statement to match common server patterns (e.g., `%.app.n8n.cloud`, `%sis2%`) and assigns a corresponding shorthand (e.g., `n8nCloud`, `SiS2`). If no pattern matches, the `server` name itself is used as the shorthand.

```sql
CREATE OR REPLACE FUNCTION public.update_server_shorthand_func()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.server_shorthand := 
        CASE 
            WHEN NEW.server LIKE '%.app.n8n.cloud' THEN 'n8nCloud'
            WHEN NEW.server LIKE '%sis2%' THEN 'SiS2'
            WHEN NEW.server LIKE '%sis1%' THEN 'SiS1'
            WHEN NEW.server LIKE '%sisx%' THEN 'SiSx'
            ELSE NEW.server
        END;
    RETURN NEW;
END;
$function$
```



