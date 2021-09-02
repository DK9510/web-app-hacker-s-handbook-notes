# information gathering/ map the application
- it is the first step of attacking the application.
- it is gathering and examining some key information about application to gain better understanding of application

# Enumerating Content and Functionality
- going through main application to all the embaded links and redirection to map the attack Surface.

## Web Spidering
- web spidering is technique where a bot or tool used to request application parse the response HTML and gather links from that response and request them and parse them and do this recursive until they find new content or we stop
- `robots.txt`
**NOTE:**
```
In some applications, running even a simple web spider that
parses and requests links can be extremely dangerous. For example, an applica-
tion may contain administrative functionality that deletes users, shuts down a
database, restarts the server, and the like. If an application-aware spider is used,
great damage can be done if the spider discovers and uses sensitive functional-
ity. The authors have encountered an application that included some Content
Management System (CMS) functionality for editing the content of the main
application. This functionality could be discovered via the site map and was not
protected by any access control. If an automated spider were run against this
site, it would find the edit function and begin sending arbitrary data, resulting in
the main website’s being defaced in real time while the spider was running.
```

## User Directed Spidering
- this is more sophisticated and Controlled Technique that is preferable to automated spidering, `spider of burp suite`can be used in this way, so it can't harm or damage the application
### Benefits for using this type of spider
- the user controls all data submitted to the application and ensure data validation requirements are met.
- user can login to application in usual way and if session terminate then re login possible 
- any dangerous functionality i.e `deletuser.jsp` is enumerated but user deciding which functions to actually request or carry out
### Hack steps
1. configure the browsers to work with the `burp` or `WebScarab` as local proxy
2. browse the entire application normally, attempting visit every links/URL, submitting forms, try with `javascript enables and disables`and try with `cookie enable and disable` we may reach different code path and endpoint
3. Review the site map generated proxy and identify any application content or functions that we not browse manually, do this work recursively until new end point were not found
4. Optionally, tell tool to actively spider the site using all of the already enumerated content, but identify any dangerous URL that terminate session or other, make it `OUT of Scope`

## Discovering Hidden Content
- some hidden files and contents that are directly not linked to application, may be use different functionality for different users `admin, authenticated, anonymous` etc
- some past functionality that not properly remove by the site owners, backup files, test files for debugging purpose etc
### Contents example
- Backup copies of live files, in case of dynamic pages, file extension may changed that not maped executable eg.`php use admin.php~` that gives source code of backup file if exist that file.
- Backup Archives that contain full snapshot of files within web root
- New functionality that has been deployed for testing but not yet linked from the main application.
- Old version of files that have not been removed from the server, that may have vulnerability that can be exploited in old version
- Configuration files and files contain data base credentials
- source files from which the live application's functionalities has been compiled
- Comments in the source code that in extreme case may contain usernames and passwords, but most provide info about state of application. eg. `test this function` or something similar
- log files that may contain sensitive info i.e `usernames, session tokens and action performed` etc

### Brute-Force Technique
- we use burp intruder for brutefocing the directory, file etc
**NOTE:**
```
Do not assume that the application will respond with 200 OK if a requested resource exists and 404 Not Found if it does not. Many applica-ations handle requests for nonexistent resources in a customized way, often returning a bespoke error message and a 200 response code. Furthermore, some requests for existent resources may receive a non-200 response 
```
also note some common status code with following exampe
  - **302 found:** if the redirect is to login page, the resource is there but accessible only by authenticated users. final is `investigate the redirect`
  - **400 Bad Request:** check if our word lists contain some white space or invalid character that cause application to give bad request
  - **401 Unauthorized** or **403 Forbidden:** this means the requested resource exist but not accessed by any user regardless of authentication or privilege level
  - **500 Internal server Error:** this usually indicates that, application expects certain parameters to be submitted when requesting resource
  
  ### HACK Steps
  1. make some manual request for known valid and invalid resource, and identify how the server handles the latter
  2. Use the site map generated through user-directed spidering as basis for automated discovery of hidden content
  3. make automated requests for common filenames and directories within each directory/path known to exist within the application, use burp intruder or automated scripts that fuzz the path 
  4. Capture the response received from the server and manually review them to identify valid resources.
  5. Perform this recursively as new content is discovered

## Inference from Published Content 
- most application employ some kind of naming Schema for their content and functionality, By inferring from the resources already identified within the application, it is possible to fine tune our fuzzer
- `Eg:` application have some name scheme like having 1st latter capital or etc to make dictionary acc to name scheme for better `fuzzing`

### HACK Step 
1. find name scheme used by target application by viewing some normal files
2. if they have `AddDocument.jsp`then probabily have `RemoveDocument.jsp` etc
3. add file extension like txt,bak,src etc, also use .java .cs like extension for may be source code that may be compiled for application
4. Search for temporary files that may have been created inadvertently by developer tools and file editors. Examples include the .DS_Store file, which contains a directory index under OS X, file.php~1, which is a temporary file created when file.php is edited, and the .tmp file extension that is used by numerous software tools.
5. Where a consistent naming scheme has been identified, consider performing a more focused brute-force exercise. For example, if AddDocument.jsp and ViewDocument.jsp are known to exist, you may create a list of actions (edit, delete, create) and make requests of the form XxxDocument.jsp. Alternatively, create a list of item types (user, account,file) and make requests of the form AddXxx.jsp.

**NOTE:**
```
we use Content-Discovery feature of Burp Pro
also we use Owasp Dirbuster 
```

## Use Of Public Information
- application may contain the functionality that are not presently linked from the main content, but that may have been linked in the past
	- **Search Engines:** as google, yahoo that maintain fine-grained index of all content that their powerfull spiders have discoverd and cached copy of this content
	- **Web Archives:** WayBackMachine `www.archive.org` allow us to view historical version of the application, in addition the contetn has been linked in the past, and also contain reference to the third party sites.
### HACK Step 
   1. use search engines and web archives to discover what content they indexed or stored
   2. use google dork like 
	   1. `site:example.com` 
	   2. `link:example.com` returns all he pages on other websites and application that contain a link to the target
   3. view cached version of the application, including the content that is no longer present, or directly not accessed with out auth.
 
 - another public source of useful information about the target application is ,`post that developers and others have made to internet forms`
### HACK Step 
- Compile list containing every `name and email` that can be discovered by `comments`, `github repositories`,`contact information`
- search for each identified names to find any questions and answers they have posted to internet forums

## Application pages versus Functional Path
- due to evolution of the web application most of the functions access via `Unique URL` which is usually name of the server side script that implements the functions.

### HACK Steps
1. identify any instances where application functionality is accessed not by requesting a specific page for that function eg.`/admin/editUser.jsp `is not access by requeting page then we bypass the name of function in a parameter i.e `/admin.jsp?action=editUser`
2.  Modify automated techniques described for discovering URL specified Content to work on the Content-access mechanism in the use within the application.
EG. `if application use parameters that specify servlet / method names, first determine its behavior when an
invalid servlet and/or method is requested, and when a valid method is
requested with other invalid parameters. Try to identify attributes of the
server’s responses that indicate “hits” — valid servlets and methods. If
possible, find a way of attacking the problem in two stages, first enumer-
ating servlets and then methods within these. Using a method similar to
the one used for URL-specified content, compile lists of common items,
add to these by inferring from the names actually observed, and generate
large numbers of requests based on these`

## Discovering Hidden Parameter
-  sometime some parameters are not linked to the hyperlink but that affect the application's logic input validation, eg. `debug=true` parameter may be there and also can bypass some input validation
### HACK Step 
  1. get list of common debug parameters like `debug, test, hide, source` etc and common values `true,1, yes,on` etc, makes large number of requests to known application page or function, iterating through all permutations of name and value
  2. we use burp intruder for this work
  3. monitor response to see if added parameter affect the application or not
   **NOTE:** we can use `param miner` burp extension for finding hidden parameter in JSON, also use to guess headers
 
 ## Analyzing Application
 - enumerating more & more content's of application is one element of `mapping process`
 - another Equally important task is to `analyzing the key attack surface` it expose and begin formulating an approach to probing the application for exploitable vulnerabilities
 - **some key areas to investigate**
	 1. Application's Core Functionality, the actions that can be leveraged to perform when used as intended
	 2. Other, More peripheral application behavior including `off-site links`, `error message`, `administrative and loging functions`, use of `redirects`
	 3. The core security mechanisms and how they function, in particular management of `session state`, `access control`, and `authentication mech'sm` and `supporting logic`
	 4. all the location at application process `user-supplied input` eg. `URL, Query string parameter, POST data, cookie`
	 5. Technology employed on the client side `forms, scripts` thick client component `java applets, ActiveX control, flash` and cookies
	 6. Technology on server side, `static/dynamic page`, `requets parameters`, `web server software`, `interaction with DataBase`, `email system`, other back-end component
	
## Identifying Entry Points of User Input
  - Every URL String up to the query string Marker
  - Every Parameter submitted within the URL query String
  - Every parameter submitted within the body of POST request
  - Every Cookie
  - Every HTTP headers that application might `process` eg. user-Agent, Referer,Accept, Accept-Language, Host headers
 
### URL File PATH
- REST Style URL could have this format
 `https://google.com/browse/electronic/nokia`
 - in this example string `electronics` and `nokia` should be treated as parameters to store a search function
 - some application might have deployed this type of REST-Style URLs for interaction its depends upon the author/developer

### Request Parameters
- Parameter submitted within `URL String`, `Message Body`, and `HTTP Coookie` are most obvious entry point for user input.
- some application does not employ std.`name=value` pair, they may have their own Scheme, which may use non-standard query String eg. `XML within Parameter field`
##### Examples of some non Standard parameter formats 
 - /dir/files;foo=bar&foo2=bar2
 - /dir/file?foo=bar$foo2=bar2
 - /dir/file/foo%3dbar%26foo2%3dbar2
 - /dir/foo.bar/file
 - /dir/foo=bar/file
 - /dir/file?param=foo:bar
 - /dir/file?data=%3dfoo%3ebar%3c%2ffoo%3cfoo2%3e%3cfoo2%3ebar2%c%2ffoo2%3e
 
- when application used non standard parameter format we need to take care while probing the application that we might miss the parameter in non standard format, that leads to critical vulnerability

### HTTP Headers
- application present different content to user to access the application via different `devices, laptop, cellphone`, this achieved by inspecting `User-Agent` header, this place may used to inject `input in the application`
**TIP:** burp intruder have large number of user-agent list we use them to make request and compare response with burp comparear
- if site resides some load balancer or proxy then they use `x-forwarded-for` header , which may process in dangerous ways, by crafting header we might able to `SQL injection` and `stored XSS`

## Identifying Server-Side Technologies
- ormally it is possible to figerprint the technologies employed on the server via various clues and indicators

### Banner Grabbing
- Many web servers disclose fine-grained version information, both about the web server software it self and about other components that have been installed.
- eg. `HTTP server` header disclose huge amount of details about some installations
```js
server: Apcache/1.2.32 (unix)  PHP/4.3.1 
``` 
etc, other locations
- Templates Used to Build HTML pages
- Custom HTTP headers
- URL Query String Parameter

### HTTP FingerPrinting
- some time HTTP request/response leaks the server and technologies used 
- tool:   httprecon

### File Extension
- File extension used within URL often disclose the platform or programming language used to implement the relevant functionality
	- asp = Microsoft Active Server pages
	- aspx = microsoft ASP.NET
	- jsp  = Java Server Pages
	- cfm  = Cold Fusion
	- php  = The PHP Language
	- d2w  = WebSphere
	- p1   = Perl Language
	- py   = Python Language
	- dll  = Usually Compiled Native Code(C or C++)
	- nsf or ntf   = Lotus Domino
- if application does not have file extension, we verify that by requesting `nonexist.extensin` and compare the response with `extension` and `with out extension` that may revel the technology
- with automated content discovery we send multiple request with diff extension and compare the response and verify the server

### Directory Names
- common for encounter subdirectory names that indicate the presence of an associated technology
- example
	- servlet = Java Servlets
	- pls     = Oracle Application Server Pl/SQL gateway
	- cfdocs or cfide  = Cold fusion
	- SilverStream   = SilverStream web server
	- WebObjects or {function}.woa  = apple WebObjects
	- rails  =  Ruby on Rails

### Session Tokens
- Many webservers/application platforms generates session tokens by default with names that provide info,of technology used
	- JSESSIONID  = Java Platform
	- ASPSESSIONID  = Microsoft Platform
	- ASP.NET_SessionId  = Microsoft ASP.NET
	- CFID/CFTOKEN  = Cold Fusion
	- PHPSESSIONID   = PHP
### Third-Party Code Components
#### HACK STEPS
   1. Identify all entry points of UserInput that processed by the application
   2. Examine Query string used by application, if it does not have standard format, then understand how name/value pair encapsulated into non-std URLs
   3. Identify Out-of-Band Channels
   4. View HTTP server Banner returned by application, note that different areas of application are handled by different back-end components, so different `server ` header being received
   5. any other software identifier in Custom HTTP headers
   6. if fine-grained information obtained about server, then research and identify any vulnerabilities
   7. Review map of application URLs to identify interesting looking File extension, directories, etc
   8. Review Names of All Session Token
   9. search on google for unusual cookies and headers etc for more info about that. 

## Identifying Server-Side Functionality

### Dissecting Request
- eg. `https://example.com/calander.jsp?name=new%20applicattents&isExpired=0&startDate=22%2F09%2F2010&endDate=22%2F03%2F2011&OrderBy=name` 
- extension: .jsp indicate Java server page
- `OrderBy` parameter indicate back-end database is used, and value may be SQL Query that may be vulnerable to SQL injection
- `edit` parameter that change 
**TIP:** 
	- eg. http://eis/pub/media/117/view 
	this Url handling functionality is equivalent 
	- http://eis/manager?scheme=pub&type=media&id=117&action=view
	so we check some other URL and confirm this, and also we can alter the URL to change `view` to `add, edit` change `117`to other `value` etc
	
### Extrapolating Application Behavior
#### HACK Step
   1. Try to identify any locations within the application that may contain clues about the internal structure and functionality of other areas
   2. it may be possible to draw any firm conclusion here; however the cases identified may prove useful at a later stage of the attack when attempting to exploit any potential vulnerabilities.
### Isolating unique Application behavior
#### hack Step 
   1. make a note of any functionality that diverges from the std GUI appearance, parameter naming or navigation mechanism, used within rest of application
   2. also note functionality eg. debug functions, usage-tracking, third party code
   3. perform full review of these areas and not assume that std defense used elsewhere in application
  
# Mapping the Attack Surface
- final stage of mapping process is finding attack surface exposed by application and vulnerabilities that are commonly associated with each one.
	1. Clint-Side Validation -Checks may not be replicated on the server
	2. DataBase Interaction - SQL Injection
	3. File-Upload and Downloading - Path traversal vulnerabilities, Stored XSS, RCE, XXE, RCE, etc
	4. Display of user-supplied Data - XSS
	5. Dynamic Redirects - Redirection and Header Injection attacks
	6. Social Networking Features - Username enumeration,  Stored XSS
	7. Login - Username enumeration, Weak password, ability to bruteforce
	8. MultiStage Login - Login Flaws
	9. Session State - Predictable Tokens, Insecure Handling of token
	10. Access Controls - Horizontal & Vertical Privilege Escalation
	11. User Impersonation functions - privilege Escalation
	12. Use Of Cleartext Communications - Session Hijacking, Capture of credentials and other sensitive data
	13. Off-site-links - leakage of Query string parameters in the Referer header
	14. Interface to External Systems - Shortcuts in the Handling of sessions and/or access controls
	15. Error Messages - Information Leakage
	16. E-Mail Interaction - Email, Command injection
	17. Native Code Components or Interaction - Buffer Overflow
	18. Use of Third party application components - Known vulnerabilities
	19. Identifiable Web server Software - Common Configuration Weaknesses, Known Software Bugs
## HacK Step
   1. Understand Core Functionality implemented within the application and main security mechanism in use.
   2. Identify all features of the application's functionality and behavior that are often associated with common vulnerabilities
   3. check 3rd party  code against public  vulnerability databases eg. osvdb.org, cve-details, exploit.db
   4. Formulate a plan of attack, prioritizning the most interesting-looking functionality and the most serious of associated potential vulnerabilities.


