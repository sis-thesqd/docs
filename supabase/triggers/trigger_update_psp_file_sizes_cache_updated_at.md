# Trigger: trigger_update_psp_file_sizes_cache_updated_at

This trigger fires `BEFORE UPDATE` on the `public.psp_file_sizes_cache` table and executes the `update_psp_file_sizes_cache_updated_at()` function. Its purpose is to automatically update the `updated_at` timestamp and reset the `cache_expires` timestamp whenever a row in the file sizes cache is updated.

## Trigger Definition

```sql
CREATE TRIGGER trigger_update_psp_file_sizes_cache_updated_at BEFORE UPDATE ON public.psp_file_sizes_cache FOR EACH ROW EXECUTE FUNCTION update_psp_file_sizes_cache_updated_at()
```

## Function: update_psp_file_sizes_cache_updated_at()

This function is triggered before an update operation on the `public.psp_file_sizes_cache` table. It sets the `updated_at` column of the new or updated row (`NEW`) to the current timestamp (`NOW()`) and also sets the `cache_expires` column to `NOW() + INTERVAL '36 hours'`, effectively extending the cache validity for 36 hours from the update time.

```sql
CREATE OR REPLACE FUNCTION public.update_psp_file_sizes_cache_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at = NOW();
    NEW.cache_expires = NOW() + INTERVAL '36 hours';
    RETURN NEW;
END;
$function$
```

