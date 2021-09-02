# Attacking Access Control
- access controls are logically built on authentication and session management.
- when access controls are defective, attacker can compromise entire application, tacking controls of administrative functionality, and accessing sensitive data belonging to every other user.

# Common Vulnerabilities
## Access-controls 
   1. Vertical Access control
   2. Horizontal access Control
   3. Context-Dependent access Controls- ensure that user's access is restricted to what is permitted given  the current application state.

## Vulnerability
- Vertical Privilege Escalation
- Horizontal Privilege Escalation
- Business Logic Exploitation eg. user bypass the payment step in shoping checkout
 
 # Completely Unprotected Functionality
 - some time some url have some functionality but we cant predict the URL, but some time in the javascript for UI/UX accedently leak the URL 
 - eg.  var isAdmin=false;
     if (isAdmin)
	 {adminMenu.addItem("/menus/secure/ff456/addNewPortalUser2.jsp","create a new user");}
 - here attacker can view JS to Identify URLs for administrative Functionality

## Direct Access to Methods
- some methods are accessible to all the user, or any access control are not set on this methods
- eg. getAllUserRoles, getCurrentUserRoles etc

# Identifier-Based Functions
**TIP:** this type of vuln arise when main application interface with an external system or back-end component, if can be difficult to share a session-based security model between different systems that may be based on diverse technology, for this dev used client-submitted parameters to make access control decisions

**NOTE:** some application implement URL as function call or parameter like `http://example.com/users/1234/view` here there is possible parameter in URL are users, UiD=1234 and type=view etc. so look for endpoints like this

## Multistage Functions
- if there is resource that need to reach with 2 or more steps and the privilege is check in stage 1 and 2 then attacker directly request to stage 3 for resource and bypass entire verification process, since developer assume that no one can directly request to stage 3, this can be exploited by attacker

## Static Files
- some time resource to static files are not prevented by access control, 
- eg. we have purchased a e book and after purchase the publisher give download URI to download book
- `http://example.com/download/123456765.pdf`
- since this is served in the normal web server and may be not implemented any access control, so here we can bruteforce names, no, to guess the resource available to attacker without auth.

- software vendors provide downloadable binaries, or any kind of files are vulnerable to this kind of attacks

## Platform Mis-configuration
- platform level configuration rules that allow or deny access on the following 
	- HTTP Request Methods
	- URL Path
	- User Role
- sometimes change the method type from `GET` to `POST` or `POST` to `GET` bypass access control to privilege escalation also
- some application restrict to change `post`to `get`, so alternative `HEAD` method , this methods works as same as `GET` but just as returning whole message body in `GET` requests response, the `HEAD` request's response simply responded with the `headers only`but the parameter passed with this may be executed on the server side function, but we didn't get response, but we verify that it later.
- some time server unrecognize methods and forward it to `GET` this bypass the access control, by sending invalid or unrecognize `HTTP Method`

# Insecure Access Control Methods
## Parameter-Based Access Control
- some application determined role or access level at time of login and from this , it trasnmitted to server via `hidden` form, this can be manipulated to exploit access control
- some times we need to bruteforce the parameter to find , since they can not seen by lower privilege user

## Referer-Based Access Control
- if user request some admin function, the application check the `referer` header to know from where the request referred, since they assume that user previously verified, this can be manipulated
-  and it is under user control, so we put any thing here

## Location-based Access Control
- some application restrict some application functionality to access from perticular location.
- bypassing methods
	- using web proxy that is based in the required location
	- using VPN that terminated the required location
	- using Mobile device that supports data roaming
	- Direct manipulation of Client-side Mechanisms for geo-location

# Attacking Access Controls
## Before attack check this
   1. Do application functions give individual users access to particular subset of data that belongs to them?
   2. Are there different levels of users such as  manager, supervisors, guests etc, who are granted access to different functions?
   3. Do administrator use functionality that is built into the same application to configure and monitor it?
   4. What functions/data resources within the application have we identified that would most likely enable us to escalate current privileges?
   5. Are there any identifiers that signal a parameter is being used to track access level

## Testing With Different User Account
- test same post parameter request with another user's session token, to see if we get the resource of same as without changing access token
- we automate the test using Burp suite, and do focus on other testing functionality
```bash
1st we crowl site using higher prirv, or with user 1d
2nd change the access token in session handling rule with lower priv, or other user to send all the request from that user
3rd compare the result and see any response or resources
```
for this we use burp comparer for comparing site mapes 

## Testing Multistage Processes
- some time site map doesn't test for multistage process, so it need manual testing
- check validation of the data , at which step, like initially application check for access control, but then it does not check for access control right
- this can be exploited by directly sending request to step 2 or step from where the validation is not there
- when testing this process, use multiple browser or browser with different container to seperate user logins

## testing with Limited Access
- some times old functionality does not have been removed or new functionality has not been implemented, we need to discover hidden content

## Testing Direct Access To Methods
- some application request directly to API function, if java application then check for `get,set,add,update,is,has` followed by capitalword.or com.companyname.xxx.yyy.ClassName etc

## Testing Controls Over Static Resources
- some static resources that the application is protecting are ultimately accessed directly via URLs, simply go to that url using Unauthenticated user

## Testing Restriction on HTTP Methods
### HACK Step
   1. Using high privileged account identify some privileged request that perform sensitive action, i.e add new user, change user's role
   2. if these request are not protected by Any anti CSRF tokens or similar feature , use high priv acc to determine whether the application still carries out the requested action if the method if modified. , test following
	   1. POST
	   2. GET
	   3. Head
	   4. Arbitrary invalid http method
   3. if application honors any request using different HTTP method than orignal method , test access controls over those requests using the standerd methodology 

# SEcuring Access Controls
- securing Access Controls is one of toughest areas of web application security
	- DO not rely on users ignorance of application, assume users know every parameter, URL  PATH
	- Do not Trust user-submitted parameter to signify access right(admin=false)
	- Do not assume that user use application pages in the intended sequence
	- Do not trust the user not to temper with any data that is transmitted value without validation

- use central application component to access check access Controls
- process every client request via this component to validate that the user making the request is permitted to access the functionality and resources being requested
## Multilayered Privilege Model
  - application server can be used to control access to entire URL paths on the basis of user roles that are defined at the application server tier.
  - application employ different database account when carying out the actions of different users
  