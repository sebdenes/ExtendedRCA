# Incident: Login outage - 2026-03-14

## What happened
Users couldn't log in for about 30 minutes on Friday afternoon. Alice deployed a fix and it went back to normal.

## Root cause
The auth service was returning 500s. There was a bug in the new session handling code from the deploy earlier that day.

## Timeline
- 2:15pm - deploy went out
- 3:00pm - alerts started
- 3:20pm - Alice rolled forward with a patch
- 3:30pm - back to normal

## Impact
Users couldn't log in for ~30 min.

## Action items
- Fix the bug (done)
- Add more tests
