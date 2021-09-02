# Finding Vulnerabilities in source code /Static Code Analysis
- some application are open source or use open source components, enabling us to download their code from the relevant repo
- if we performing a pentest in consultancy context, the application owner may grant access to his source code to maximize the effectiveness of our audit
- we may discover a files disclosure vulnerability within an application that enables us to download its source code
- most application use Client-side code as JAvaScript, which is accessible without requiring any privileges access

# Approaches to COde Review
   ### HACK STEP
   1. Tracing user-COntrollable data from its entry points into the application, and reviewing the code responsible for processing it
   2. Searching the codebase for signatures that may indicate the presence of common vulnerabilities, and reviewing these instances to determine whether an actual vulnerability exists
   3. Performing line-by-line review of inherently risky code to understand the application's logic and find any problems that may exist within it.
  
  
  ## Cross site Scripting
   - any place where user input placed in HTML Markup 

  ## SQLI
  - search for Hard coded SQL query string that take user input and append in the string
  - search for key word
  ```bash
  SELECT
  INSERT
  DELETE
  AND
  OR
  WHERE
 ORDER BY
 ```
 
## PATH Traversal
- user controllable input is passed to File System API Without any validation of the input verification
- close attention to upload and download files where filesystem APIs are being Invoked

## Arbitrary Redirection
- also some times Arbitrary redirect find using Client side java script,

## OS COmmand Injection 
- code that interface with external system 
- function in php like `system(), eval()`, `exec()` in JS and many more that interact with OS API.

## Source COde COmments
- many software vulnerabilities are actually documented within source code comments, this often occurs because developers are aware that a particular operation is unsafe and they record a reminder to fix the problem later, but never get around.
- searching String in comment
```bash
bug
problem
bad
hope
todo
fix
overflow
crash
inject
xss
trust
```

# refer ch 19 in WAHH
 