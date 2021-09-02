# Attacking Authentication
- Authentication is First step towards the defense mechanism to prevent from the Unauthorized Access of the application's data.
- it also lies at the heart of application's protection against malicious attacks.
- In real world Authentication is the Weakest link, which enables an attacker to gain unauthorized access. 

# Authentication Technology
- HTML form Based Authentication
- MultiFactor Authentication mechanism
- Client SSL Certificate/smartCards
- HTTP basic and Digest Authentication
- Windows-integrated Authentication Using NTLM or Kerberos
- Authentication service 
- Third Party Authentication service

# Design Flaws in Authentication Mechanisms
## Bad Password
- many application employ no or minimal control over the quality of User's Passwords
	- Very short or Blank Password
	- Common dictionary words or Names
	- same as the username
	- Still set to default value
### Hack Step
- review the website for any description of the rules
- if self registration is possible, attempt to register several accounts with different kinds of weak password to discover what rules are in place
- try to change password to weak values
## Brute-Forcible Login
### HACK Step 
   1. Manually submit several bad Login attempts for an account we control, monitoring the error messages we receive
   2. After about 10 Failed logins, if the  application has not returned a message about account lockout, attempt to log in in correctly. if this succeeds, there is probably no account lockout policy.
   3. if account is locked out, try using different account and if application issue any cookies use, each cookies for only single login attempts
   4. also if the account is locked out, then submit valid password to see if any difference is detected after lockout.
   
 ## Verbose error message
   - is we submit wrong username then error says username is incorrect and  for password also same then we exploited this by enumerating valid usernames and then bruteforce them with password list
 ## Password Change Functionality
   - this is main focus of testing, may vulnerable to change of password for another user and another business logic functionality
 ### Hack Steps
   1. Identify any password change functionality within the application , this is not explicitly linked from published content, it still may be implemented, we need to discover that hidden field, by guessing path or functions
   2.  make various to password reset function using invalid usernames, invalid existing password, mis-matched new password, confirm new password
   3.  try to identify any weird behavior and understand the functionality to examine the behavior

## Forgotten Password Functionality
- **TIP:**  Even if the application does not provide an on-screen field for you to provide an e-mail address to receive the recovery URL, the application may transmit the address via a hidden form field or cookie. This presents a double opportunity: you can discover the e-mail address of the user you have compromised, and you can modify its value to receive the recovery URL at an address of your choosing

### HACK Step
   1. Identify any forgotten password functionality within the application, if this is not explicitly linked from published content, it may still be implemented
   2. Understand how the forgotten password function works by doing a complete walk-through using an account you control.
   3. if the mechanism uses a challenge, determine whether users can set or select their own challenges, and review this for any that appear easily guessable
   4. if the mechanism uses a password `hint`, do the same exercise to harvest a list of password hints, and target any that are easily guessable
   5. Try to identify any behavior in the forgotten password mechanism that can be exploited as the basis for username enumeration,or brute-force attacks
   6. if the application generates an e-mail containing a recovery URL in response to forgotten password request, obtain a number of these URLs, and Attempts to identify any patterns that may  enable you to predict the URLs issued to other Users.

## Remember Me Functionality
- some remember me functions are implemented using simple persistent cookie eg. `RememberUser=def` when this cookie is submitted to the initial application page, the application trusts the cookie to authenticate the user, and it creates an application session for that person, bypassing the login, An attacker can use a list of common or enumerated usernames to gain full access to the application without any authentication
### Hack Step
1. Activate `Remember me` functionality and see effect of request and response in the application.
2. Closely inspect Persistent Cookie that are set, and also data that is persisted in other local storage mechanisms, such as browser's local storage that identifier for the user
3. Even Stored data appears to be heavily encoded or obfuscated, review this closely, compare the result's of `remembering` several very similar username/passwords to identify any opportunities to reverse-engineer the original data, 
4. attempt to modify the content of persistent cookie to convince application that another user has saved his details on our computer

## User Inpersonation Functionality
some application implement the facility for privileged user of the application to impersonate other users in order to access data and carry out actions within their user context.
eg. banking applications allow helpdesk operators to verbally authenticate a telephone user and then switch their application session into that user's context to assist him
  - it may be implemented as `hidden` function, which is not subject to proper access controls, eg. any know or guess `/admin/ImpersonateUser.jsp` may able to use of that function and impersonate any other user
  - application trust user-controllable data, when determining whether the user is performing impersonation
  - if application allows administrative users to be impersonated, any weakness in the impersonation logic may result in vertical Privilege escalation vuln.
### HACK Step
   1. Identify any impersonation functionality within application
   2. Attempt to use the impersonation functionality directly to impersonate other users.
   3. Attempt to manipulate any user-supplied data that is processed by the impersonation function in an attempt to impersonate other user, pay attention where username is being submitted other than during normal login
   4. if we got success in using functionality, try to impersonate administrative users to elevate privilege
   
 ## Incomplete Validation of Credentials
 ### HACK Step
 - using Account we control, attempt to log in with variation on own password, removing last character, changing the case of character, removing any special character if any of this successful , then continue experiment and understand validation in actually occuring

## NonUnique Usernames
### HACK Step
 - if self registration is available then register with same username twice with different password
 - if not possible then it can be used to username enumeration
 - if registration of duplicate succeeds, attempt to register same username, twice with same password, and determine application's behavior
 
 ## Predictable initial Passwords
 ### HACk step
  - if application generates password, try to obtain several in quick succession and determine whether any sequence or pattern can be discerned
  - if it can, extrapolate the pattern to obtain a list of passwords for other application users
  
## Insecure Distribution of Credentials
### HACK step
- obtain new account, if it not require to set credentials during registration, determine the means by which the application distributes credentials to new users
- if account activation URL is used, try to register several new accounts in close succession, identify any sequence in the URL we receive, attempt to these URL To ATO
- Try to use Single Activation Link Multiple Times, and see if application allows this or not

# Implementation Flows in Authentication
## DEfects in Multistage Login mechanisms
- application may assume that user who access stage 3 must have cleared stage 1 & 2, therefore it may authenticate an attacker who proceeds directly from stage one to 3 and correctly complete it,enabling attacker to log in with only one part of various credentials normally required
- some case application verify users data on stage 1 but since this is verified they didn't verify now and this place attacker manipulate the data, and got privilege escalation also.
### HACK Step
   1. perform complete, valid login using account we control, record every piece of data submitted to the application using proxy
   2. identify each distinct stage of login and data controlled at each stage, find single information collected more than one time or not,via hidden form, cookie, or preset URL parameter
   3. Repeat login process,with various malformed request
	   1. try performing login steps in different sequence
	   2. try proceeding directly to any given stage, continue from there
	   3. try skipping each stage and continue with next
	   4. think differently
	4. if data is submitted more than once, try submitting different data values,
## Insecure Storage Of Credentials
### HAck Step
  1. Review all of the application's authentication-related functionality,if we found any instance in which user's password is transmitted back to the client, this indicates that password is being stored in the plainText, or insecurely, or in reversible encryption
  2. if any kind of arbitrary command or query execution vulnerability is identified within the application, attempt to find the location within the application's database or filesystem where user creds are stored.

# Secure Authentication
## USe Strong Credentials
## Handle Credentials Secretively
## Validate Credentials Properly
## Prevent Information Leakage
## prevent Brute-Force attacks
## Prevent Misuse of Password Change Function
## Prevent Misuse of Account Recovery Function
## Log, Monitor and Notify


