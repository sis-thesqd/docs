# Trigger: task_name_change

This trigger fires `AFTER UPDATE` on the `public.tasks` table. It directly makes an HTTP POST request to a webhook on `sis2.thesqd.com`.

## Trigger Definition

```sql
CREATE TRIGGER task_name_change AFTER UPDATE ON public.tasks FOR EACH ROW EXECUTE FUNCTION supabase_functions.http_request('https://sis2.thesqd.com/webhook/task-name-change', 'POST', '{\"Content-type\":\"application/json\"}', '{}', '5000')
```

