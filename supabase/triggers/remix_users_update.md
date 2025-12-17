# Trigger: remix_users_update

This trigger fires `AFTER DELETE` on the `public.remix_users` table. It directly makes an HTTP POST request to a webhook. The webhook URL is `https://hkdk.events/z0o8owd5gaamw1` and it includes a custom header `{\"Content-type\":\"application/json\",\"type\":\"remix_update_delete\"}`.

## Trigger Definition

```sql
CREATE TRIGGER remix_users_update AFTER DELETE ON public.remix_users FOR EACH ROW EXECUTE FUNCTION supabase_functions.http_request('https://hkdk.events/z0o8owd5gaamw1', 'POST', '{\"Content-type\":\"application/json\",\"type\":\"remix_update_delete\"}', '{}', '5000')
```



