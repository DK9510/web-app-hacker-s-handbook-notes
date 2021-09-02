# Attacking The Application Server
- web application depends on the other layers of the technology stack that support it, including application or web server, OS, Networking infrastructure
- attacker may target any of these components, compromising the technology on which an application depends very often enables an attacker to fully compromise the application itself.

# Vulnerable Server Configuration
- some application shipped with insecure configuration by default, which present opportunity for attack unless they are explicitly hardened

## DEfault Credentials
- some application use default creds for admin, and dont change it, and this panel may hosted in port 8080, 8443 etc
- we reference to internet for default credentials for specific admin interface using their documentation

## Default COntent
- some default application functionality that leverage to attack the server , which come during shipped
	- Debug and test functionality designed for use by administrator
	- simple functionality designed to demonstrate certain common tasks
	- Powerful functions not intended for public use but unwittingly left accessible 
	- Server manuals that may contain useful information that is specific to the installation itself
### Debug Functionality
- it is great value for attacker, it may contain useful information about the configuration and runtime state of the server and applications running on it.
- `phpinfo.php` exist on many apache installation, this execute phpinfo() and returns the output .
- it contains wealth of information about the PHP env., configuration settings, web server modules and file path

### Sample Functionality
- many server include sample scripts and pages designed to demonstrate how certain application server functionality and API can be used.
	- many sample script contain security vulnerability that can be exploited to perform actions not intended by the script's authors
	- Many sample scripts actually implement functionality that is of direct use of an attacker

### Powerful Functions
- some functions are default by public , that should not be accessible from the outer internet, like uploading web archive can lead to command injection


### HACK STEP
- use NIkto scanner, it can identify default web content easily
- use search engines and other resources to identify default content and functionality included within the technologies known to be use

## Directory Listing
- when web server receives a request for a directory rather than an actual file, it may respond in one of three way
	- it may return default resource within the directory, i.e index.html
	- it may return an error, such as HTTP status code 403 
	- it may return listing contents of the directory
### HACK Step
- for each directory discover on the web server during application mapping make a request for just this directory, and identify any case where a directory listing is returned


# WebDAV Methods
- the term given to collection of HTTP methods used for Web-Based Distributed Authoring and Versioning
- methods
	- `PUT` uploads the attached file to the specified location
	- `DELETE` deletes the specified resource
	- `COPY` copies the specified resource to the location given in the `Destination`
	- `MOVE` moves the specified resource to the location given in the destination header
	- `SEARCH` searches a directory path for resources
	- `PROPFIND` retrieves information about the specified resource, i.e author, size and Content-Type
	- `TRACE` used to identify which http headers are allowd or hidden we can submit data
- we use `OPTION` method to list the HTTP methods that are permitted in particular directory

**TIP:** if server side script extension is blocked try uploading in `.txt` file if `MOVE` methods spported then try to `MOVE` it with `script` extension.

# Application server as Proxy
- some application implement reverse proxy for fetching resource from the server and forward request, this case if attacker leverage proxy server it can perform various attacks
	- attacker use server to attack 3rd party systems on the internet.
	- attacker may be able to use proxy to connect to arbitrary hosts on the organization's internal network, thereby reaching targets that can not be accessed directly from the outer internet
	- attacker use proxy to connect to back to other services running on the proxy host itself, circumventing firewall restrictions and potentially exploiting trust relationships to bypass authentication
	
### HACK STEP
  1. using both `GET` and `CONNECT `requests, try to use the webserver as proxy to connect to other servers on the internet and retrieve content form them.
  2. using both techniques, attempt to connect to different Ip address and ports within the hosting infrastructure
  3. Using both techniques, attempt to connect to common port numbers on the web server itself by specifying 127.0.0.q as the target host in the request
  
# Misconfigured Virtual Hosting
### HACK STEP
   1. Submit `GET` request to the root directory using 
	   1.  The correct Host Header
	   2.  Arbitrary Host Header
	   3.  The server's IP address in the Host header
	   4.  No Host Header
    2. compare response to these request, when IP address is used in the HOST header, the server mat simply respond with a directory listing, we may find different default content is accessible
    3. if we observe different behavior, repeat application mapping exercise using `Host` header that generated different results, be sure to perform `NIKto` scan using the `-vhost` options to identify any default content that may have been overlooked during initial application mapping 

# Application Framework Flaws

# Filter bypass
1. using `%0A` or `%0a` new line character
2. using `UTF-8/16`encoded charecter that can have similarity between ascii char
3. by double or tripple URL encoding
4. by placing payload in `"payload"` or blocked string in `"double quotes"` 
5. by using angle brackets to place a programming `goto` label before the blocked expression eg. `<<foo>>` `<<foo>>sys.package.procedure`
book: `oracle hacker's handbook`

# Finding Web server Flaws
- if vulnerability already discovered by others we have to look if exploit is available or any common CVE etc
	- www.exploit-db.com
	- www.metasploit.com
	- https://osvdb.org/search


# Defense
# Web Application Firewall
### HACK STEP
  1. presence of WAF can be deduced using the following steps:
	  1. submit arbitrary parameter name to the application with a clear attack payload in the value, ideally somewhere the application includes the name and value on the response, if the application blocks the attack, this is probably due to an external defense
	  2. if various can be submitted that is returned in server response, submit a range of fuzz strings and encoded variants to identify the behavior of the application defenses to user input
	  3. confirm this behavior by performing the same attacks on variables within the application
#### Bypassing 
  1. for all fuzzing strings and requests, use benign strings for payloads that are unlikely to exist in a standard signature database, avoid `/etc/passwd` or `/windows/system32/config/sam` as payloads for file retrieval, also avoid using terms as `<script>` in XSS attack use `print()` or `console.log(document.domain)` for XSS fire
  2. if a particular request is blocked, try submitting same parameter in a different location or context, submit same parameter in URL in `GET` request within the body of `POST` request, and within `URL` in the `POST` request
  3. determine locations where user input is submitted in a nonstandard format such as `serialization` or `encoding`, if none are available, build the attack string by concatenation or by spanning it across multiple variables