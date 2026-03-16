# Incident Summary (Fill in)

**Title: Intermittent 500 Errors on Task Creation Due to Invalid Timestamp Parsing  
**Date: 3/13/26

**Severity: Sev-2

## Impact
- Who/what was impacted? 

Some users attempting to create tasks through the API.

- Symptoms observed by customers/internal users

There are some tasks creation requests returned HTTP 500 Internal Server Error

Issue occurred intermittently and tasks could not be created successfully for affected users

## Detection
- How did we learn about this? (customer reports, monitoring, etc.)

Customer states that: “Sometimes creating a task fails with a 500.” The issue appears intermittently when submitting a request to the POST /api/tasks endpoint. 

Investigation and found from the production logs keep repeating with the exception below:
Example log evidence:

> System.FormatException: String '' was not recognized as a valid DateTime.

## Timeline (UTC)

10:05 — Customer report received about intermittent task creation failures.

10:15 — Reviewed application logs and identified FormatException during task creation.

11:15 — Confirmed failure occurs when X-Client-Timestamp header is missing.

11:45 — Reproduced issue locally by sending a request without the timestamp header.

12:30 — Implemented defensive parsing using DateTime.TryParse.

13:30 — Tested fix with missing, invalid, and valid timestamps.

-14:30 — Verified that task creation succeeds without returning HTTP 500.


## Root cause
- What happened and why?

The CreateTask endpoint attempted to parse the X-Client-Timestamp request header using DateTime.Parse.
When the header was missing or empty, the code executed:

DateTime.Parse("")

This resulted in:

System.FormatException

Because the exception was not handled, the API returned an HTTP 500 Internal Server Error.

Some clients included the header correctly, while others did not, causing the issue to appear intermittently in production.

## Mitigation / resolution
- What did we change to stop the bleeding?

Added defensive input validation to ensure the timestamp header is parsed safely.
Replaced unsafe parsing with DateTime.TryParse.

- What was the final fix?

File need to change: src/SupportEngineerChallenge.Api/Endpoints/TaskEndpoints.cs

Before

>var createdAt = DateTime.Parse(clientTimestamp);

After

>var createdAt = DateTime.TryParse(clientTimestamp, out var parsed)
>? parsed
>: DateTime.UtcNow;

## Verification
- How did we verify the fix worked?

Performed the test below:
1. Start API
2. Create sample request:

Using:  Post /api/tasks

Body:

{

  "userId": "user-001",

  "title": "Seeded task 3987 for user-001"

}

3. Confirm task created successfully with response code http 200 or 201.



## Follow-ups / action items
* Add request validation to ensure required headers are properly handled.
* Improve structured logging for incoming requests to simplify debugging.
* Add monitoring for increased error rates on the tasks endpoint.
* Add unit tests covering invalid and missing request headers.

