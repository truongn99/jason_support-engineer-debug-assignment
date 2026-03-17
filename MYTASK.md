## Issues confirmed and Fixed:

Task creation returning HTTP 500


## 1) Triage & reproduction

Reproduction steps:

Getting the app running by following the steps below and I was able to get the API up and running with the port: 5000



Here are the steps:
```bash
dotnet restore
dotnet run
```


API Swagger: http://localhost:5000/swagger/index.html

UI:   http://localhost:5000/



Run tests:
```bash
dotnet test
```

I have reproduced the issue that the task creation sometimes return error code http 500 internal server.

Below are the steps:



Start the API Swagger by running this link in the browser: http://localhost:5000/swagger/index.html


Example request:

Using:  Post /api/tasks

Body:
```bash
{

  "userId": "user-001",

  "title": "Seeded task 3987 for user-001"

}
```



From the logs, I was able to see the error exception below:


```bash
fail: Microsoft.AspNetCore.Server.Kestrel[13]

      Connection id "0HNK0PGV633MF", Request id "0HNK0PGV633MF:00000001": An unhandled exception was thrown by the application.

      System.FormatException: String '' was not recognized as a valid DateTime.

         at System.DateTimeParse.Parse(ReadOnlySpan`1 s, DateTimeFormatInfo dtfi, DateTimeStyles styles)

         at System.DateTime.Parse(String s)
```
        

For the task list is slow for some users and duplicated user, I was not able to reproduce the issue on my end. Since I don't have access to production database access or restrict, I would get on a call with customer to check whether the issue was related to the network or database issue or exactly the steps on how to replicate the issue.

.

## 2) Root cause analysis

**What is happening: 

The API throws an exception error code 500 Internal Server error during task creation 


**Why it is happening (root cause) 

This is causing the issue:
```bash
System.FormatException: String '' was not recognized as a valid DateTime.
at System.DateTime.Parse(String s)
```

This means DateTime.Parse() is trying to parse an empty string. When the header is missing, the value becomes “” which throws error.

**What you considered / ruled out (short)

The following potential causes were investigated and ruled out:
> From the logs clearly show the failure right away before database interaction
> Invalid JSON
> There is no high system load.
> Network issue
> Authentication or authorization errors 

## 3) Fixes (keep them safe & minimal)


File need to change: src/SupportEngineerChallenge.Api/Endpoints/TaskEndpoints.cs



Before:
`var createdAt = DateTime.Parse(clientTimestamp);`



After:
`var createdAt = DateTime.TryParse(clientTimestamp, out var parsed)
? parsed
: DateTime.UtcNow;`



The above fix will handle the missing or empty header and invalid timestamp




For the task 4, 5 and 6 please check the github link here:
https://github.com/truongn99/jason_support-engineer-debug-assignment

## Tradeoffs

For the timestamp parsing issue, I chose a defensive fallback (DateTime.UtcNow) rather than rejecting the request with a 400 error. This approach avoids breaking existing clients that may not provide the header while still preventing the service from returning HTTP 500 errors.

With more time, I would also review client expectations to determine whether the timestamp should become a required field.



## What I Would Do Next With More Time

If I had additional time, I would:

Add automated tests covering missing or invalid headers
Add database indexes to improve task listing performance
Add monitoring/alerting for error rates and latency.

