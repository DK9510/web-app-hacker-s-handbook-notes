# Automating Customized Attacks
- some advanced customized automating testing is more effective, less time & effort Consuming and give results faster.

## Uses for Customized Automation
1. Enumerating Identifiers
2. Harvesting data
3. Web application Fuzzing

# Enumerating Valid Identifiers
- application's login functionality, differ for invalid username and invalid password, lead to username enumeration
- application implement resource with documentIDs, log entries etc
## Basic Approach
- use burp intruder to set parameter and fuzz or automate
## Detecting Hits
### HTTP Status Code
- 200 - ok
- 301- 302 redirection to diff URL
- 401 or 403 - not authorized or not allowed
- 404 - content not found
- 500 - server side error while processing request

### Response Length
- larger response length may return our content
### Response Body
- specific string that used to detect when returned valid resource or not 
- like string contain in response for invalid id `Invalid document ID` we separate response based on the string contain or not
### Location header
- some time application redirect always like , we fuzz for report if valid id is provided then redirect to `download.jsp` other wise `error.jsp`, we differentiate using redirection header value
### Time Delays
- server response take time when requeste is valid, and invalid

# Harvesting Useful Data
# Fuzzing for Common Vulnerability
# Burp Intruder - puuting all together 
- use session handling Rules, Cookie jar and Macros to automate complex request in Burp
- also have functionality called tracer which use for tracing and debugging of the session handling rules

# CAPTCHA Controls
	