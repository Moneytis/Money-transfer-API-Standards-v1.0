# Supported HTTP Codes

> Example of data JSON structured returned in case of error : 

```json
{
   "status" : 400,
   "message" : "the date expirationDate must be after today"
}
```

Code	|	Label	|	Description
---------- |---------- | -------
200	|	OK	|	Everything went fine
400	|	Bad Request	|	Server could not correctly process request
401	|	Unauthorised	|	Authorisation required to process request
403	|	Forbidden	|	Access forbidden
404	|	Not Found	|	Resource not found
405	|	Method Not Allowed	|	Method not allowed for resource
500	|	Internal Server Error	|	Error occurred on the server side


In case of error, the API should return a HTTP Code corresponding to the issue with additional information : 

* “message” is a human reading abstract message of the error

