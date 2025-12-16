# Trigger: aa_trigger_n8n_cloud_v2

This trigger fires `AFTER INSERT OR UPDATE` on the `public.aa_log` table, specifically when the `status` of the new row is 'triggered'. It directly makes an HTTP POST request to an n8n webhook.

## Trigger Definition

```sql
CREATE TRIGGER aa_trigger_n8n_cloud_v2 AFTER INSERT OR UPDATE ON public.aa_log FOR EACH ROW WHEN ((new.status = 'triggered'::text)) EXECUTE FUNCTION supabase_functions.http_request('https://hkdk.events/pm3hmhtrwgr8q6', 'POST', '{\"Content-Type\":\"application/json\",\"source\":\"supabase-webhook\"}', '{}', '1000')
```

