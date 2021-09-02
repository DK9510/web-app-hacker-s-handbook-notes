# WEb application hacker's methodology
![[attacking-methodology.png]]

## General Guidelines
- when modify data, modify it with url encoded data
- `&` used to separate parameters in query string, for insert we use `%26`
- `=` used to separate name and value pairs
- `?` used to mark beginning of the URL query string
- `space` used to mark end of the URL we use `%20,+`
- for inserting `+ we need to encode that to %2b`
- `;` used to separate individual cookies 
- `#` user to mark the fragment identifier within the URL, to enter we used to url encode
- `%` is used to prefix in URL encoding  we use `%25`
- `%00` null byte and `%0a` new line

![[map-application-content.png]]

# 1.1 Explore Visible content
- normal visit application and visit every endpoint while burp running in back ground
# 1.2 Consult Public resource
- use way back machine
- use google dorking
- see comments in application

# 1.3 Discover Hidden Content
- directory fuzzing 
- finding hidden parameter with param miner burp extension
- if they have AddDocument.jsp and ViewDocument.jsp then their must be EditDocument.jsp and RemoveDocument.jsp also there
- hidden endpoint in Client side scripts/JS/HTML files
# 1.4 Discover Default Content
- run Nikto 
- make request to application's root directory using range of user-agents headers `www.useragentstring.com/pages/useragentstring.php` or use burps list of user agents
# 1.5 Enumerate identifier-Specified Function 
- identify instance where functions are accessed by passing an identifier of the function in a request paramter
- compile a list of common function names, and fuzz with them

# 1.6 Test FOr Debug parameter
- choose pages or function where hidden debug parameters `debug=true` may be implemented, common at login, file upload, etc
- use common debug parameter names `debug, test, hide, source` and common values `true, yes, on, 1` 

# 2 Analyze the Application
# 2.1 identify Functionality
- identify core functionality for which application is created, or designed, 
- identify core security mechanism employed by the application and how they work, understand how mechanism that handle invalid inputs/methods
# 2.2 Identify Data Entry Points
- identify all different entry points that exist for introducing user input i.r `URLs, query string parameter, POST data, Cookies, Other HTTP headers processed by applicaiotn`
- examine customized data transmission or encoding mechanism used by application, 	
- identify out-of-band channel via user-controllable or other third-party is being introduced into the application's processing

# 2.3 Identify the Technology Used
- identify each technology used on the client side, form, script, cookies, java applets, activeX control etc
- use `builtwith.com` and wappalyzer
- run Httprint tool to finger print
- identify 3rd party scripts, that also used by other application, to find any hidden functionality, use `inurl: ` in google 

# 2.4 Map attack Surface 
# 3 Test Client-Side Controls
![[test-client-side-control.png]]

# 3.1 Test Transmission Of Data Via the Client
# 3.2 Test Client-side Controls over user input
# 3.3 Test Browser Extension Components

# 4. Test the Authentication Mechanism
![[test-authentication.png]]

## 4.1 Understand the mechanism
- identify technology used 
- locate all functionality
## 4.2 Test Password Quality
## 4.3 Test Resilience to password guessing
## 4.4 Test any ACcount Recovery Function
## 4.5 Test any Remember Me function
## 4.6 Test any Impersonate Function
## 4.7 Test Predictability of Auto Generated Credentials
## 4.8 Test for unsafe Transmission of Credentials
## 4.9 Test For unsafe Distribution of Credentials
## 4.10 Test For insecure storage 
## 4.11 Test for logic flows
 ### 4.11.1 test for failed open condition
 -   Submit an empty string as the value.
 -  Remove the name/value pair.
 -  Submit very long and very short values.
 -  Submit strings instead of numbers, and vice versa.
 - Submit the same named parameter multiple times, with the same and different values
 ### 4.11.2 Test any multistage mechanism
## 4.12 Exploit any vulnerability to gain unauthorized access

# 5 Test the session Management Mechanism
![[session-management.png]]
## 5.1 Understand the Mechanism
## 5.2 Test Token for Meaning
- Log in as several different users at different times, and record the tokens received from the server. If self-registration is available and you can choose your username, log in with a series of similar usernames that have small variations, such as A, AA, AAA, AAAA, AAAB, AAAC, AABA, and so on. If other user-specific data is submitted at the login or is stored in user profiles (such as an email address), perform a similar exercise to modify that data systematically and capture the resulting tokens.
## 5.3 Check for Insecure Transmission of Tokens
## 5.4 Check for Disclosure of Tokens in Logs
## 5.5 Test Token for Predictability
## 5.6 Check Mapping of TOkens to sessions
## 5.7 Test for Session Termination
- when logout or concurrent logins etc
## 5.8 Check for session Fixation
## 5.9 Check for CSRF

# 6 Test For Access Controls
![[Access-controls.png]]

## 6.1 Understand the Access control Requirements
## 6.2 Test with Multiple Accounts
## 6.3 Test with Limited Access
## 6.4 Test for Insecure Access Control Methods
- referer based access control,
- edit=false or access=read like this parameters
- using HEAD methods to doing some unprevilege action

# 7 Test for Input based Vulnerabilities
## 7.1 Fuzz All Request Parameters
### additional Script injection 
```bash
;echo 111111
echo 111111
response.write 111111
:response.write 111111
```

## 7.2 test for SQLI
## 7.3 Test for Xss and Other response injection
- test for header injection
- test for Open redirection
- test for Stored attack
## 7.4 Test for OS Command Injection
## 7.5 Test For Path Traversal
## 7.6 Test for Script Injection
## 7.7 Test for File Inclusion

# 8 Test for Function-specific input vulnerabilities
## 8.1 Test for SMTP Injection
## 8.2 Test for Native Software Vulnerabilities
- test for Buffer Overflow
- test for Integer Vulnerabilities
- test for Format String Vulnerabilities
## 8.3 Test For SOAP injection(simple object access protocol)
## 8.4 Test for LDAP injection
## 8.5 Test for XPAth injection
## 8.6 Test for Back-end Request injection
## 8.7 Test for XXE injection

# 9 Test for Logic Flows
## 9.1 Identify the Key attack Surface
- identify following features while mapping
	- multistage processes
	- Critical Security function such as login
	- Transition across trust boundaries 
	- Context-Based functionality presented to user
	- Checks and adjustments made to transaction price or quantities
## 9.2 Test Multistage process
## 9.3 Test Handling of Incomplete Input
## 9.4 Test Trust Boundaries
## 9.5 Test Transaction Logic

# 10 Test for shared hosting Vulnerabilities
## 10.1 Test segregation in shared infrastructure
## 10.2 Test Segregation between ASP-Hosted applications

# 11 Test for Application server Vulnerabilities
## 11.1 Test for Default credentials
## 11.2 Test for default content
## 11.3 Test for Dangerous HTTP Methods
## 11.4 Test for Proxy Functionality
## 11.5 Test for Virtual Hosting Mis-configuration
## 11.6 Test for Web server Software Bugs
## 11.7 Test for Web application Firewalling

# 12 Miscellaneous Checks
## 12.1 Check for DOM based attacks
## 12.2 CHeck for local privacy Vulnerability
## 12.3 Check for weak SSL cipher
## 12.4 Check for Same Origin Policy Configuration
- `/crossdomain.xml` 
- `clientaccesspolicy.xml`
# 13 follow up Any Information Leakage


