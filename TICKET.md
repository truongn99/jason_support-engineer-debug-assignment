# Follow-up Ticket (Fill in)

**Title: Intermittent 500 Errors on Task Creation Due to Invalid Timestamp Parsing  
**Priority: P2
**Owner:**

## Description
What should be improved after the immediate incident is resolved?
- Production logs show intermittent 500 errors when creating tasks.
The error occurs when the API attempts to parse the X-Client-Timestamp request header using DateTime.Parse. If the header is missing or empty, the application throws a System.FormatException, resulting in an unhandled exception and HTTP 500 response.
Example log evidence:

Connection id "0HNK0PGV633MF", Request id "0HNK0PGV633MF:00000001": An unhandled exception was thrown by the application.
      System.FormatException: String '' was not recognized as a valid DateTime.
         at System.DateTimeParse.Parse(ReadOnlySpan`1 s, DateTimeFormatInfo dtfi, DateTimeStyles styles)
         at System.DateTime.Parse(String s)

This indicates the system assumes the header is always present, but some clients send requests without it.

We should improve more detailed logging and monitoring should be implemented to detect problems

## Acceptance criteria
* Task creation endpoint no longer throws HTTP 500 when X-Client-Timestamp is missing or invalid.
* Replace DateTime.Parse with safe parsing (DateTime.TryParse).
* Add structured logging for invalid or missing timestamp headers.


## Notes / context
- Links to relevant code/areas

File: src/SupportEngineerChallenge.Api/Endpoints/TaskEndpoints.cs

Before
var createdAt = DateTime.Parse(clientTimestamp);

After
var createdAt = DateTime.TryParse(clientTimestamp, out var parsed)
? parsed
: DateTime.UtcNow;

