# Runbook — SupportEngineerChallenge

> How to diagnose these issues in production (what to look at, what logs/queries/metrics help — use the structured log lines and sample artifacts as a guide) 

**I would check the logs for something like 
FormatException
DateTime.Parse

fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HNK0PGV633MF", Request id "0HNK0PGV633MF:00000001": An unhandled exception was thrown by the application.
      System.FormatException: String '' was not recognized as a valid DateTime.
         at System.DateTimeParse.Parse(ReadOnlySpan`1 s, DateTimeFormatInfo dtfi, DateTimeStyles styles)
         at System.DateTime.Parse(String s)
         at SupportEngineerChallenge.Api.Endpoints.TaskEndpoints.<>c.<<MapTaskEndpoints>b__0_1>d.MoveNext() in /Users/truonghan/SupportEngineerDebugAssignment/src/SupportEngineerChallenge.Api/Endpoints/TaskEndpoints.cs:line 41


> How to verify the fix 

1. Start API
2. Create task without timestamp header
3. Confirm response with code http 200 or 201.


>Mitigation / rollback plan if the fix regresses 
Before making the change, I would backup of the file then deploy then roll back file if regression occurs.

> Update this file as part of the exercise.

## Service overview
- **Service:** SupportEngineerChallenge.Api
- **Purpose:** Minimal task tracker (create + list tasks)
- **Data store:** SQLite (`app.db` in the API working directory)

## Common commands

**Run locally**
```bash
cd src/SupportEngineerChallenge.Api
dotnet run
```

**Run tests**
```bash
dotnet test
```

## Key endpoints
- `GET /api/tasks?userId={id}&limit={n}`
- `POST /api/tasks`

## Using log artifacts

- **Create-task 500:** Inspect `artifacts/sample_api_log.txt` (or production logs). Look for the `CreateTask request` line — `X-Client-Timestamp present=False` or `length=0` indicates missing/invalid header. The stack trace shows `FormatException` at `DateTime.Parse`.
- **Slow list:** Look for `ListTasks completed` lines with high `elapsedMs` (e.g. `artifacts/sample_slow_list_log.txt`). Correlate `userId` and `limit` with slow requests.

## Troubleshooting checklist (starter)

### “Create task fails with 500”
- Check API logs in console.
- Verify request payload and headers.
- Look for unhandled exceptions in `POST /api/tasks`.

### “Tasks list is slow”
- Confirm dataset size (seed can be large).
- Inspect how the list endpoint fetches and filters data.
- Review query patterns and database usage.

### “Duplicates / wrong order after refresh”
- Compare API response vs UI rendering.
- Check the UI state update logic during refresh.
- Verify how the list is merged and ordered.

## Verification steps (starter)
- Create tasks from UI and via Swagger.
- Refresh tasks repeatedly; confirm no duplicates and ordering is correct.
- Validate list endpoint returns only requested user's tasks.

## Rollback / mitigation ideas (starter)
- Roll back to last known good version.
- Temporarily disable problematic client behavior (feature flag / UI change).
- Add guardrails (e.g. input validation, error handling) to prevent unhandled exceptions.
