# Errors

The Slant API uses the following error codes:

Error code | Meaning
---------- | -------
400 | Bad Request -- There's something wrong with your request.
401 | Unauthorized -- Your API key is invalid. The user should be asked to log in again.
403 | Forbidden -- You don't have permission to access the specified resource.
404 | Not Found -- The specified resource could not be found.
406 | Not Acceptable -- You requested a format that isn't JSON.
429 | Too Many Requests -- You're sending too many requests. Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
