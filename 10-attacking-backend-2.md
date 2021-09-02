# Injecting Into Back-end Http Requests
### Server Side HTTP Redirection
- allow attacker to specify an arbitrary resource or URL that is then requested by Front-end Application server
### HTTP Parameter Injection 
- attacks allow attacker to inject any arbitrary parameters into back-end HTTP request made by application server, HTTP Parameter Pollution used to override the original Parameter value specified by the server

## Server Side HTTP Redirection
- it arise when user controllable input taken by application and incorporate it into URL that it retrieves using backend HTTP Request, application may perform some processing on it, such as adding a standard suffix.
-  the impact is same as `SSRF` and exploitation also same as `SSRF`
-  with the help of this vuln the attacker can use this as proxy server to visit 3rd party application
### HACK STEP
  1. identify parameter that appears hostnames, IP, or full URL PATH
  2. for each param, modify value to specify an alternative resource, and see if the resource being appears in the server's response
  3. try accessing Collaborator URL using application to see request from the application or our own IP

## HTTP Parameter Injection
 - this arise  when user-supplied parameter are used as parameters with a back-end HTTP Request,
 - like SOAP 
 - front end request sent from the user's browser, cause the application to make a further back-end HTTP request to another web server within the infrastructure
 - we inject extra valid parameter while sending request to exploit application's logic
 - eg 
 ```js
 POST /dotransfer.asp HTTP/1.1
 Host: mdsec.net
 Content-Length: 123
 
 formcc=23423&amount=123432&toacc=123433%26cleardfunds%3dtrue&submit=Submit
 ```
 - here we injected `%26clearfunnds%3d=true`parameter that is pass using `toacc` parameter, when further pass into other infrastructure it acts as separate parameter and execute in backend, we have changes application's logic
 
 **NOTE:** this is hard because we need exact knowledge of back-end parameters that are used, or the application may use any other 3rd party components whose code can be obtained and researched
 
 ## HTTP Parameter Pollution
 - different webserver behave differently for how to handle Multiple Parameters like example
	 - user 1st instance of parameter
	 - use last instance of parameter
	 - Concatenate the parameter value, maybe adding a separator b/w them 
	 - Construct an array containing all the supplied values
- the impact of vulnerability is depend on backend that how where it is, how it parse, like eg, if there is firewall that prevent from giving certain value but in parameter pollution, the two same parameter merge at back-end so we devide malicious parameter value and send to server using `parameter pollution` so it by pass firewall and security mechanism easily

## Attack Against URL Translation
- some time application takes user input and process it into other URL, so we take advantage of functionality to pass do `HPP or HPI` to exploit logic of application

### HACK STEP:
  1. Target each request parameter in turn, and try to append a new injected parameter using various syntax
	  1. ``%26foo%3dbar — URL-encoded &foo=bar``
	  2. `%3bfoo%3dbar — URL-encoded ;foo=bar`
	  3. `%2526foo%253dbar — Double URL-encoded &foo=bar`
   2. identify any instances where application behaves as if the original parameter were unmodified(if parameter cause some difference in application's response)
   3. Each instance identified in the previous step has a chance of parameter injection, attempt to inject a known parameter at various points in the request to see if it can override/modify existing parameter 
   4. if this cause the new value to override the existing one, determine whether we can bypass any fron-end validation by injecting a value that is read by back-end server
   5. Replace the injected known parameter with additional parameter name like common param that may present in application, but are hidden or not accessible by privilege level, we need to bruteforce param
   6. test application's tolerance of multiple submission of same parameter within request, submit redundent values before and after other parameter, and at different locations within the request


# Injecting into Mail Service
- many application contains a facility for users to submit messages via the application, such as to report a problem to support personal or provide feedback about the application. 
- this facility is usually implemented by interfacing with a mail(SMTP) server, typically user supplied input is inserted into SMTP conversion that application server conducts with the mail server, if an attacker can submit suitable crafted Input that is not filtered or Sanitized , he can inject arbitrary SMTP commands into this conversation

## Email Header Manipulation
 - some time adding value in header to manipulate request like adding Bcc in header with new like char
 - `attackr@gmail.com%0aBcc:ourmain@dk.con`
 - will cause php mail() command to also add Bcc value so that mail also get email from the application
 
 ## SMTP Command Injection
 - in other case application perform SMTP Conversion itself, 
 - in this situation, it may be possible to inject arbitrary SMTP Commands Directly into this Conversation, potentially taking full control of messages being generated by application
 - example: application have feed back form
```js
POST feedback.php HTTP/1.1
Host: application.com
Content-Length: 213

From=daf@wah.com&Subject=Site+feedback&Message=foo
```
this cause application to perform
```bash
MAIL FROM: daf@mail.com
RCPT TO: feedback@wah.com
DATA 
From: daf@wah.com
To: feedback@wah.com
Subject: Site Feedback
foo
```

## Finding SMTP Injection Flaws
- to probe application's mail functionality effectively, we need to target every parameter that is submitted to an email-related function, even those that may initially appear to be unrelated to the content of the generated message
- test for each kind of attack, and perform each test case using both windows and UNIX-style Newline Characters

### HACK step:
 1. Submit following each of the following test strings as each parameter in turn, inserting own e-main address at relevant position:
	 1. `<youremail>%0aCc:<youremail>`
	 2. `<youremail>%0d%0aCc:<youremail>`
	 3. `<youremail>%0aBcc:<youremail>`
	 4. `<youremail>%0d%0aBcc:<youremail>`
	 5. `%0aDATA%0afoo%0a%2e%0aMAIL+FROM:+youremail>%0aRCPT+TO:+<youremail>%0aDATA%0aFrom:+<youremail>%0aTo:+<youremail>%0aSubject:+test%0afoo%0a%2e%0a`
	 6. `%0d%0aDATA%0d%0afoo%0d%0a%2e%0d%0aMAIL+FROM:+youremail>%0d%0aRCPT+TO:+<youremail>%0d%0aDATA%0d%0aFrom:+<youremail>%0d%0aTo:+youremail>%0d%0aSubject:+test%0d%0afoo%0d%0a%2e%0d%0a`
   2. Note any error message the application returns, if these appear to relate to any problem in the e-mail function, investigate it
   3. application's response may not indicate in any way whether a vulnerability exists or was successfully exploited, we monitor email address we specified to see if any mail is received
   4. Review CLosely `HTML` form that generates the relevant request, this may contain clues about the server-side software being used, may contain `hidden/ disabled` field.
   
**TIP:** Also, because they involve interfacing to an unusual back-end component, they are often implemented via a direct call to the relevant operating system command. Hence, in addition to probing for SMTP injection, you should also closely review all e-mail-related functionality for OS command injection fl	aws.
