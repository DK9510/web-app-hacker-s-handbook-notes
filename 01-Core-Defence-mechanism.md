# Handling User Access
### Most application handle access using 
   - Authentication
   - Session Management
   - Access Control
 defect in single component may enable an Attacker to gain unrestricted Access to application's functionality and data
 
 # HandLing User Input
 ## Approaches to Input Handling
 ### Reject Known Bad
 - blocks the particular strings that are black-listed for input validation
 #### ByPass
 - `SELECT` blocked try `SeLeCt`
 - `or 1=1 --` blocked try `or 2=2 --`
 - `alert('xss')` blocked try `promt('xss)'`
 - filter design to block specific `keyword`, bypassed with `non-standerd characters b/t expressions` eg. `SELECT/*foo*/username,password/*foo*/FROM/*foo*/users` eg. `<img%09onerror=alert(1)src=a>`
 - Bypass with `NULL Byte` `%00` before the blocked syntax can stop processing input after `%00` `%00<script>print(9510)</script>`

### Accept Known Good
- can only allow the input that is whitelisted by the filter or back-end
- it is good solution but this approach does not represent all-purpose solution to the problem handling user input

### senitization
- senitize malicious input and remove them from passing to sink of the application

# Handling Attacker
- Handling Errors 
- Maintaining Audit Logs
- Alerting administrators
- Reacting to attacks

# Web application Technologies
- in HTTP version 1.1 `Host` Request Header is Mandatory 
## Request
### headers 
 - `Referer` used to indicate the URL from which request is originated
 - `User-Agent` provide information about browser from where request is initiated
 - `Host` header specified the host name, this is necessary when there is `virtual hosting` or multiple web hosted in single server
 
 ### HTTP Methods
 - `GET` normal request to fetch any content from the server
 - `HEAD` same as `GET` but the server should not return the message BODY
 - `TRACE` designed for diagnostic purpose, The server return in the response body the exact contents of the request message it received, this used to detect the `proxy effect` in between client and server
 - `OPTIONS` ask server to report available `HTTP Methods` 
 - `PUT` attempts to upload the specific resource to the server
 - `DELETE` delete the specific content from the server 

### Status code Response
413 - Request Entity too Large , body of the request is too large that server can't handle

414 - Request URI too LOng - same as 413 response, URL used in request it too large for the server to handle

503 - service unavailable ,