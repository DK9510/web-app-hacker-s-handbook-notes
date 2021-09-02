# Attacking Back End Component
# Injecting OS Command
- some developer develop application , that some functionality are needed to done with OS, some time taking user input and directly put it into functionality that interact with OS, lead to Command Injection Vulnerability, and can compromise entire server by crafting malicious payload
- common way to issue Os command in `php exec() function` and in `ASP wscript.shell` 
- if developer Use API to interact with this function then also it is vulnerable

## Injecting Through Dynamic Execution
- Many web scripting language supports dynamic execution of code that is generated at runtime, this feature enables developers to create application that dynamically modify their own code in response to various data and condition.
### `PHP` function `eval()` used to dynamically execute code that is passed to the function at runtime.
- eg. search function that enables users to create stored searches that are the dynamically generated as links within their UI
- url is looked like this when search function is used
- `/search.php?storedsearch=\$mysearch%3dwahh`
- server-side application implements this functionality by dynamically
- `$storedsearch = $_GET[‘storedsearch’];eval(“$storedsearch;”);`
- in this situation we submit crafted input that is dynamically executed by the `eval() funciton`
- exploit
```js
/search.php?storedsearch=\$mysearch%3dwahh;%20echo%20file_get_contents('/etc/passwd')
or
/search.php?=\$mysearch%3dwahh;%20system('cat%20/etc/passwd')
```

**NOTE:** `perl` also contain `eval` function that can be exploited in the same way, `;` should be URL-Encoded as`%3b` in `ASP` `Execute()` performs similar role

# Finding OS Command Injection Flows
- in mapping application, any field where application appears to be interacting with the underlying operating system by calling external processes or accessing the file system.
- we probe all these function, looking for Command Injection flaws.
-  the application may issue operating system commands containing absolutely any item of user-supplied data, including every URL and body parameter and every cookie. To perform a thorough test of the application, you therefore need to target all these items within every application function

- Two broad Types of Metacharacter used to inject a separate Command into an existing preset Command
	- character `; | & &&` and `new line ` may be used to batch multiple commands, these may doubled with diff char, like in windows `&&` cause second execute if 1 st is execute and `||` execute both regardless
	- the backtick (\`) can be used to encapsulate a separate command within a data item being processed by the original command.  placing an injected command within backticks cause the shell interpreter to execute the command and replace the encapsulated text with the result of this command before continuing to execute the resulting command string

- most reliable way to detect blind command injection that doesn't return any result, we cause time delay like sleep(1000) etc
- 2 nd method is to generate Out-Of-Band channel for data exfilteration using `nslookup` or burpcollaborator for OAST data exfilteration 

### HACK STEP
   1. we normally use `ping` command as means of triggering a time delay by causing server to ping loopback interface for specific period
   2. `|| ping -i 30 127.0.0.1; x  || ping -n 30 127.0.0.1 &`
   3. to maximize chance of detecting a command injection flaw, if application is filtering certain command separators, we submit alternative for filter bypass
   ```bash
   | ping -i 30 127.0.0.1 |
   | ping -n 30 127.0.0.1 |
   & ping -n 30 127.0.0.1 &
   & ping -i 30 127.0.0.1 &
   ; ping 127.0.0.1 ;
   %0a ping -i 30 127.0.0.1 %0a
   `ping 127.0.0.1`
   ```
   
   5. if a time delay occurs, the application may be vulnerable to command injection, repeat test case several time to check that the delay was not the result of network latency  or other anomalies, try change value of `-n` & `-i` parameter
   6. Using whichever of the injection Strings was found to be successful, try injecting a more interesting command i.e ls/dir nslookup etc
   7. unable to retrieve result directly, we use
	   1. we can open Out-of-Band channel back to our computer, try using TFTP to copy to our server, use telnet or netcat 
	   2. we redirect result of command to a file within web root directory, which can then retrieve directly using web browser
	   3. eg. dir > C:\\inetpub\\wwwroot\\foo.txt
     8. when found means of injecting command and retrieving result, we determine privilege by `id or whoami`
		
- Some time it may not possible to inject entirely separate command due to filtering of required characters, or behavior of command API, but it may be possible to perform interfere with the behavior of command being performed
### HACK STEP
   1. `< and >`character used to direct the content of the file to the command's input and to direct the command's output to a file, if giving separate command is not possible, we still able to read and write files
   
  **TIP:** many command injection attacks require us to inject spaces to separate command-line arguments, if find that spaces are being filtered by the application, and then platform we attacking is UNIX-Based, we may be able to use the `$IFS` environment variable instead, which contains the whitespace field separators
  
  # Finding Dynamic Execution Vulnerabilities
  - this is most arise in language such as `PHP, Perl`, but any application platform may pass user-supplied input to a script-based interpreter, sometimes on a different back-end server
### HACK STEP
  1. any user-supplied data may be passed to dynamic execution function, some of the items most commonly used in this way are the names and value of cookie parameters and persistent data stored in the user profiles as the result of previous action
  2. try submitting following values
  ```bash
  ;echo%20111111
  echo%20111111
  response.write%20111111
  :response.write%2011111
  ```
  3. Review application's response, if the string 111111 is returned on its own, application is likely to be vulnerable to injection of scripting commands
  4. if string 111111 is not returned, look for any error messages that indicate that our input is being dynamically executed and that may need to fine-tune our syntax to achieve injection of arbitrary command
  5. if application uses `PHP`, we use test string `phpinfo()`, which, if successful , returns the configuration details of the PHP environment
  6. if application appears to be vulnerable, verify this by injecting some commands, that result into Time delays eg. `system('ping%20127.0.0.1')`

# Preventing OS Command  Injection
- best way is to stop calling out directly system commands via user supplied input, do some filtering of dangerous character in from  the input string

# Manipulating File Path
 - some application accepts file path and also process it this cause vulnerabilities like `PATH traversal` and `Fil inclusion`

# PATH Traversal Vulnerabilities
- it arise when application uses user-controllable data to access files and directories on the application server or another backend filesystem in unsafe way
- by crafted input attacker can read contents of any file on the server, or overwrite sensitive files, ultimately leading to arbitrary command execution on the server.

## Finding and Exploiting Path Traversal Vulnerabilities
- some times Path traversal vulnerability, attacker able to read sensitive information, and also sometime it can compromise entire application and server.
### Locating Target For attack
- during mapping of application we have encounter many areas or parameter, any functionality whose explicit purpose is uploading and Downloading files, 

#### HACK STEP
  1. review information gathered during mapping to identify following 
	  1. any instance where request parameter appears to contain the name of file or directory i.e `include=main.inc` etc
	  2. any application functions whose implementation is likely to involve retrieval of data from a server filesystem, i.e displaying office docx or image
   2. during all testing perform in relation to every other kind of vulnerability, look for error messages or other events that are of interest, find any endpoint where user supplied data passed to file APIS or parameter to OS commands

**TIP:** in whitebox testing , it is easy to identifying targets for path traversal testing, because we monitor all filesystem interaction that application performs

### Detecting Path Traversal Vulnerabilities 
#### HACK STEP
  1. if any parameter is insert any arbitrary directory then try to manipulate it
	  1. file=foo/file.txt
	  2. try submiting this  file=foo/bar../file.txt
	  3. if both response is identical then this may vulnerable to Path traversal, and attempt to directly access different files
   2. if application's behavior is different in this two cases, it may be blocking or striping or sanitizing traversal sequences, resulting in invalid file path


#### HACK STEP
   1. if function we attacking provide write access to a file, it may be more difficult to verify conclusively , one test is to attempt to write two file one-that should be writable by any user and one that should not be writable even by root user
   2. in windows ../../../../../../../../writetest.txt and ../../../../../../../windows/system32/config/sam
   3. on UNIX based platform ../../../../../../../../../tmp/writetest.txt and ../../../../../../../../../tmp
   4. alternative method for verifying a traversal flaw with write access is to try to write a new file within the web root of the webserver and then attempt to retrieve this with a browser, this does not work if we does not know web root directory of server, user have no permission to write there
  
  ### Circumventing Obstacles to Traversal Attacks
  - if initial step is fails to perform path traversal, this because developer is aware of this flow and protect the application from the path traversal attacks, but their may be bypassed by skilled attacker
  - if application sanitize or remove path traversal sequence, this can be bypassed by `encoding`,`using ....// instead of ../` etc

#### HACK STEP:
  1. always try path traversal sequence using both back slashes and forward slashes
  2. try simple URL-encoded representation of traversal sequence using following encoding
```bash
   DOT  - %2e
   Forward slash - %2f
   Backslash    - %5c
   ```
  3. Try using 16-bit Unicode Encoding
```bash
   DOT  - %u002e
   Forward slash - %u2215
   BackSlash     - %u2216
```
  4. Try double Url Encoding
```bash
   DOT - %252e
   Forward slash - %252f
   BackSlash   -  %255c
```
  5. Try overlong UTF-8 uncide encoding
```bash
  DOT  -  %c0%2e, %e0%40%ae, %c0ae so on
  Forward slash  - %c0%af, %e0%80%af, %c0%2f, and so on
  BackSlash    -  %c0%5c, %c0%80%5c, and so on
```
  6. if application is attempting to sanitize user input by removing traversal sequence and does not apply this filter recursively, it may be possible to bypass the filter
```bash
....//
....\/
..../\
....\\
```

- the second type of input filter commonly encountered in defense against path traversal attacks involving verifying whether user-supplied filename contains a `suffix(file type)` or `prefix(starting directory)` that the application expects, 

#### HACK STEP:
  1. some application check whether user-supplied filename ends in a particular file type or set of file types and reject attempts to access anything else, some times this subverted by placing `NULL BYte` at end of request filename followed by filetype that application accepts 
  `eg. ../../../../boot.ini%00.jpg`
  2. some application attempts to control file type being accessed by appending their own-file type suffix to the filename supplied by the user,  in this situation also adding `NULL Byte` or previous exploit works
  3. some application check whether the user-supplied filename start with a particular sub-directory or filename, this also can be bypassed easily by `filename/../../../../../../etc/passwd`
  4. if non of the previous is work, then application might implementing multiple types of filters, there for this can be bypass by `combining several of this attacks simultaneously` both against `traversal sequence filter` and against `file type filter`
  5. if possible, the best approach here is to try to break the problem into separate stages ,eg
  ```js
  diagram.jpg successfull
  /foo/../diagram.jpg fails,  try all possible traversal sequence bypass until a variation on the second request is possible
  if these successful traversal sequence bypass don't enable us to access /etc/passwd , then probe whether file type filtering is implemented and can be bypassed by
  diagram.jpg%00.jpg
  try to probe to understand all the filters being implemented 
  ```
  

## Coping With Custom Encoding
- sometimes the application used custom encoding while uploading and downloading file which is obfuscated we need to deobfuscate that encoding for exploitation
**NOTE:** You may have noticed the appearance of a redundant ./ in the name of our uploaded file. This was necessary to ensure that our truncated URL ended on a 3-byte boundary of cleartext, and therefore on a 4-byte boundary of encoded text, in line with the  base64 encoding scheme. Truncating an encoded URL partway through an encoded block would almost certainly cause an error when decoded on the server.

## Exploiting Traversal Vulnerabilities
- after identified a path Traversal vulnerability that provides read or write access to arbitrary files on the server's file system, 
- what kind of attack that can be performed

### HACK STEP: 
  1. we exploit read access path traversal flaws to retrieve interesting files from the server that may contain directly useful information for further attack or some sensitive info
	  1. password files for the OS & application
	  2. server and application configuration files to discover other vulnerabilities
	  3. include files that may contain data base credentials
	  4. database source used by application, such as MYSQL DB files or XML files
	  5. source code to server-executable pages to perform a code review in search of bugs (.aspx)
	  6. application log files that may contain usernames and session tokens and the like
   2. if you find a path traversal vulnerability that grants write access, your main goal should be to exploited this to achieve arbitrary execution of commands on the server
	   1. Create Scripts in user's startup folders
	   2. Modify files such as `in.ftpd` to execute arbitrary commands when a user next connects
	   3. write scripts to a web directory with execute permissions, and call them from your browser

# Preventing Path Traversal vulnerabilities
- by fat the most effective means of eliminating path traversal vulnerabilities is to avoid passing user-submitted data to any file system API,

# File Inclusion Vulnerabilities
- many scripting language support the use of include files, this facility enables developers to place reusable code components into separate files and to include these within function-specific code files as and when they are needed. 

## Remote File Inclusion
- PHP lang is particularly susceptible to file inclusion vulnerabilities because its include functions can accept a remote file path.
- eg.
```js
https://example.com/main.php?country=Us

the application process the country parameter as follows
$country=$_GET['country'];
include($country.'.php');

this cause environment to load the file US.php that is located
on the webserver filesystem, the contents of this file are 
effectively copied into main.php file and executed
```
this behavior can be exploited by the attacker, by crafting malicious file and include it within application, can also achieve RCE Also

## Local File Inclusion
- in some case, include files are loaded on the basis of user-controllable data, but it is not possible to specify URL to a file, on external server, 
- in this situation we may still able to exploit application's behavior to perform Unauthorized action
	- there may be server-executable files on the server that we can not access through normal route, 
	- there may be static resource on the server that are similarly protected from direct access, if cause these to be dynamically included into other application page, execution env simply copy contents of static resource into its response

### Finding file Inclusion Vulnerabilities
- arise when name of server side file passed explicitly as parameter
#### HACK STEP:
   1. submit in each targeted parameter URL for a resource on a web server that we control, & determine whether any request are received from the server hosting the target application
   2. if 1st test fail, try submitting URL containing nonexistent ip address, & determine whether timeout occur or not
   3. if application found vulnerable to Remote file, construct a malicious script using available APIs in relevant lang
   4. for testing `LFI` check this
	   1. submit name of known executable resource on the server, observer changes
	   2. submit name of known static resource on the server, to see response
	   3. if vuln to LFI, attempt to access any sensitive functionality that we can not reach directly via web server
	   4. test to access files in other directories using traversal techniques 


# Injection into XML Interpreters
## Injecting XML External Entities
- eg request contain xml data we manipulate it to enter external entity to exploit it
```xml
<!DOCTYPE rook [ <!ENTITY dk SYSTEM "file:///etc/passwd"> ]>
<search><searchTerm>&dk;</searchTerm></search>
```
this cause XML parser to fetch content of file /etc/passwd

## Cause
 1. attacker can do SSRF to access internal private network
 2. exploit vuln on back-end application,
 3. test for Open pots on back-end system,
 4. cause Denial of serveice by reading file stream idefinitely `<!DOCTYPE root [<!ENTITY dk SYSTEM "file:///dev/random"> ]>` 
 5. can cause DOS using loading static entity, `xee` unrestricted external entity expansion 

## Injecting into SOAP service
- Simple Object Access Protocol (SOAP) is message-based communication technology that use XML format to encapsulate data,
- it can be use to share information and transmit messages between systems, even if these run on different OS and Architecture
- eg. a banking application transfer funds using request that is then goes to SOAP and then it process with back end
```js
POST /bank/27/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 65
FromAccount=18281008&Amount=1430&ToAccount=08447656&Submit=Submit
```
in behind scene this soap request is initiated after browser sends request to server
```xml
<soap:Envelope xmlns:soap=”http://www.w3.org/2001/12/soap-envelope”>
<soap:Body>
<pre:Add xmlns:pre=http://target/lists soap:encodingStyle=
“http://www.w3.org/2001/12/soap-encoding”>
<Account>
<FromAccount>18281008</FromAccount>
<Amount>1430</Amount>
<ClearedFunds>False</ClearedFunds>
<ToAccount>08447656</ToAccount>
</Account>
</pre:Add>
</soap:Body>
</soap:Envelope>
```
here we see `<clearFunds>false` value is indicate insufficiant balance, but we modify the request while sending to soap to bypass this using 
```js
POST /bank/27/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 119
FromAccount=18281008&Amount=1430</Amount><ClearedFunds>True
</ClearedFunds><Amount>1430&ToAccount=08447656&Submit=Submit
```
here if it pasrse 1 st `<clearfunds>` then this cause it to take value as `TRUE` and we transfer fund, if we dont have balance then also
- we manipulate multiple times until we got desired output 
- some times manipulating `Request` cause error that can be used to craft our payload precisely.

### Finding & EXploiting SOAP Injection
- SOAP injection can be difficult to detect, because supplying XML metacharacters in a noncrafted way breaks the format of SOAP message, often resulting Uniformative Error message

#### HACK STEP:
  1. submit Rouge XML CLosing tag i.e `</dk> or </foo>` in each parameter in turn, if no error occur, our input is not inserted into SOAP messages or input being Sanitized in someway
  2. if error was received, submit instead a valid opeaning & closing tag pari `<foo></foo> or <dk></dk>`, this cause error to disappear, the application may be vulnerable 
  3. In some situations, data that is inserted into an XML-formatted message is subsequently read back from its XML form and returned to the user. If the item you are modifying is being returned in the application’s responses, see whether any XML content you submit is returned in its identical form or has been normalized in some way. Submit the following two values in turn: `test<foo/> or test<foo></foo>`
  4. If the HTTP request contains several parameters that may be being placed into a SOAP message, try inserting the opening comment character ``<!--`` into one parameter and the closing comment character ``!-->`` intoanother parameter. Then switch these around (because you have no way of knowing in which order the parameters appear). Doing so can have the effect of commenting out a portion of the server’s SOAP message. This may cause a change in the application’s logic or result in a different error condition that may divulge information.
  