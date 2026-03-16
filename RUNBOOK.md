# Runbook — SupportEngineerChallenge
* Operational guide for troubleshooting and maintaining the SupportEngineerChallenge API service.

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
The API will start on:
http://localhost:5000/

**Run tests**
```bash
dotnet test
```

## Key endpoints
- `GET /api/tasks?userId={id}&limit={n}`
- `POST /api/tasks`


Diagnosing Issues in Production
1. Create Task Returns HTTP 500
Symptoms

Customers report that task creation occasionally fails with a 500 error.

What to Check

Inspect API logs for requests to:

POST /api/tasks

From sample_api_log.txt from artifacts,look for log entry similar to:

CreateTask request UserId=user-001 Title=Buy groceries X-Client-Timestamp present=False length=0

Followed by a stack trace such as:

System.FormatException: String '' was not recognized as a valid DateTime.

This indicates the X-Client-Timestamp header was missing or invalid.

From my local log found:

> fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HNK0PGV633MF", Request id "0HNK0PGV633MF:00000001": An unhandled exception was thrown by the application.
      System.FormatException: String '' was not recognized as a valid DateTime.
         at System.DateTimeParse.Parse(ReadOnlySpan`1 s, DateTimeFormatInfo dtfi, DateTimeStyles styles)

2. Root Cause
The API attempted to parse the timestamp using:

DateTime.Parse()

If the header value is empty, this throws a FormatException, causing an unhandled exception and returning HTTP 500.

Metrics to Monitor

Helpful production metrics:

API 5xx error rate

POST /api/tasks error rate

Request validation failures

> How to verify the fix 

1. Start API
2. Create task without timestamp header
3. Confirm response with code http 200 or 201.


>Mitigation / rollback plan if the fix regresses
- Before making the change, I would backup of the file for the working version and deploy new code change. If the issue is still occuring, we can roll back of the working file.
