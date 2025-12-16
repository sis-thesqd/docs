# Trigger: new_comment

This trigger fires `AFTER INSERT` on the `public.clickup_comments` table. It directly makes an HTTP POST request to a webhook on `sis1.thesqd.com`.

## Trigger Definition

```sql
CREATE TRIGGER new_comment AFTER INSERT ON public.clickup_comments FOR EACH ROW EXECUTE FUNCTION supabase_functions.http_request('https://sis1.thesqd.com/webhook/6047251c-1bf5-4813-8083-6f59d98e717d', 'POST', '{\"Content-type\":\"application/json\"}', '{}', '1000')
```

