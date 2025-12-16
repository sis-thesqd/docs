# Trigger: prf_feedback

This trigger fires `AFTER INSERT` on the `public.prf_feedback` table. It directly makes an HTTP POST request to a webhook on `prf.thesqd.com`.

## Trigger Definition

```sql
CREATE TRIGGER prf_feedback AFTER INSERT ON public.prf_feedback FOR EACH ROW EXECUTE FUNCTION supabase_functions.http_request('https://prf.thesqd.com/webhook/process-feedback', 'POST', '{\"Content-type\":\"application/json\"}', '{}', '5000')
```

