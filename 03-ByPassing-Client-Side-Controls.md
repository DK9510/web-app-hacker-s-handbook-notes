# ByPassing Client Side Control
-  keep an EYE on Hidden Filed in form Submission

-  HTTP Cookie
- cookies may be have some values eg. `cookie: DiscountAgreed=25` this can be modified.
- some application implement referer header for some functionlity like `admin panel` can also be bypassed using Referer header

### encoded/encryptes/obfuscated data
- some application use encrypte data while transmiting so tempering is not possible but we decrypt the value change its value re-encrypt the and send it to the server.
# TIP: Base64 Decode
- when trying to decode the Base64 Encoded value, common mistake is to begin decoding at the wrong position within the string, Because of How base64 encoding works,
- if we start at the wrong position, the decoded string will contain gibberish.
- Base64 is Block-based format in which every 4 bytes of encoded data translated into 3 bytes of decoded data.
- when decoding base64 string do not give any meaningful, try starting from four adjacent offsets into the encoded string
## asp.net application 
- it have hidden field in `view-state` that contain encoded data about application that can be decoded in server side, we can temper it for changing application logic.

## Length limit in the input field
## Script Based Validation
- intercept the request and modify the request to bypass js filters
	
## Disabled Elements
- if element on the HTML form is flagged `disabled` it appears on screen but usually grayed out and can not sent to the server, while requesting.
`<input type="text" disabled="true" name="price" value="99">`
- since this can not be submitted to the server in the request but we add this parameter in the request to server may it accept the parameter and exploited by further.
``when ever find disabled elements in the HTML try to submit with the request and see the effect of it on the server``

**NOTE:** we use BURP's HTML modification feature.
# Browser Extension 
## Common Browser Extension Tech.
- Compiled to intermediate Bytecode
- execute with in virtual machine that provide sandbox enviornment for execution
- they may use remoting frameworks employing serialization to transmit complex data structure/objects over HTTP

## Handling Serialized Data 
- application may serialize data or object transmitting them within HTTP requests, and we can temper the data of serialized object by deserialize , modify and reserializing and then send it to server so it can break and give parsing error.
- burp plugin `DSer` for serializing and deserialize
	- Java serialization identification : `Content-Type: application/x-java-serialized-object`
	- Flash serialization:`Content-Type: application/x-amf` use AMF format for serialization 
	- SilverLight Serialization: `Content-Type: application/soap+msbin1` it uses `SOAP(NBFS)` format for serialization plugin: `WCF-binary-SOAP`
	
## Obstacles to intercepting Traffic from Browser Extension
- some time the traffic of Browser Extension we not see in the burp proxy,  this is due to component's handling of HTTP proxies or SSL this can be handled via some careful configuration of tools
- 1st problem, client component may not honor the proxy con-figuration you have specified in your browser or your computer’s settings. This is because  Components may issue their own HTTP requests, outside of the APIs provided by the browser itself  or the extension framework.
- 2nd problem is Certificate that extension does not accept, this also circumvented

**NOTE:** if we didn't see traffic from the browser extension then we use `network sniffing` tools such as wireshark for see the traffic of the extension and may identify the traffic

## Decompiling Browser Extension
- some times it is defecult to read due to defensive technique used

### Downloading The Byte COde
- usually it is specified in the HTML Source COde, java applets loaded uisng `<applet>` tag
	- Some proxy tools apply filters to the proxy history to hide from view Items such as Images, Stylesheet files, that we are less interested, if we cannot find request to Browser extension, then we need to modify the display filters
	- Browsers usually cache the downloaded byteCOde for extension components, and if it is already loaded then, we need to close all the instance of browser and clear all the cache of the browser in order to make request to bytecode by the application and we can intersept it.
	
**TIP:**
- If you have identified the request for the bytecode in your Burp Proxy history, and the server’s response contains the full bytecode (and not a reference to an earlier cached copy), you can save the bytecode directly to file from within Burp. The most reliable way to do this is to select the Headers tab within the response viewer, right-click the lower pane containing the response body, and select Copy to File from the context menu.
### Decompiling The ByteCOde
- java applet packages as `.jar(java Archives)` , after unzip bytecode contains in `.class` files
- SilverLight Objects are packaged as `.xap` files, after unzip bytecode contains in `.dll` files
#### Java Tools
- java decompiler
#### flash Tools
- Flasm
- Flare
- SWFScan
#### SilverLight Tools
- .NET Reflector
### Working on The source COde
items to look at while reviewing source code
- Input Validation or Other Security-relevant logic, event that occur on client side
- Obfuscation or encryption Routines being used to wrap user-supplied data before it is sent to the server
- Hidden Client-side Functionality That is not visible in User-interface, might be unlocked by modifying the component
- Reference to server SIde functionality That we have not previously identified via application Mapping
### Recompiling and Executing Within the Browser
- Java use `javac` in JDK to recompile
- Flash use `flasm` to reassemble from Adobe to recompile modified ActionScript Source code
- SilverLight `Visual Studio` for recompile

### Manipulating the Original Component using Javascript
- in some case, it is not necessary to modify the component's bytecode, instead, we may be able to achieve necessary objectives by modifying the JavaScript within   the HTML page that interacts with the component 
#### HACK Step
   1. Download the compiled byteCOde, and unpack it 
   2. Review the source & understand Functionality
   3. if the component contains any public methods that can be manipulated to achieve our objective, intercept HTML response that interacts with component, add some javascript to invoke the appropriate methods using our input
   4. if not modify the source code to achieve the obejctive, recompile it and execute it
   5. if the application being used to submit obfuscated data or encrypted data to server, use modified version to check for obfuscated attack

# Handling Clint Side Data Securely
## Transmitting Data Via the Client
-  Some ways of using signed or encrypted data may be vulnerable to replay attacks. For example, if the product price is encrypted before being stored in a hidden fi eld, it may be possible to copy the  encrypted price of a cheaper product and submit it in place of the original price. To prevent this attack, the application needs to include sufficient context within the encrypted data to prevent it from being replayed in a different context. For example, the application could concatenate the product code and price, encrypt the result as a single item, and then validate that the encrypted string submitted with an order actually matches the product being ordered.
-  If users know and/or control the plaintext value of encrypted strings that are sent to them, they may be able to mount various cryptographic attacks to discover the encryption key the server is using. Having done this, they can encrypt arbitrary values and fully circumvent the protection offered by the solution.

## Validating Client-Generated Data
- Lightweight Client-side controls Such as HTML form fields and JavaScript can be Circumvented easily and provided no assurance about the input that the server receives
- Controls Implemented in Browser extension components are sometimes more difficult to circumvent, but this may merely slow down an attacker for short Period of time
- Using  Heavily Obfuscated or packed client-side Code provides additional obstacles, however a determined attacker can always overcome these.
the only secure way to validate client-generated data is on the server side of the application. Every item of data received from the client should be regarded as tainted and potentially malicious

# Common MYTH
- It is sometimes believed that any use of client-side controls is bad. In particular, some professional penetration testers report the presence of client-side controls as a “finding” without verifying whether they are replicated on the server or whether there is any non-security explanation for their existence. In fact, despite the significant caveats arising from the various attacks described in this chapter, there are nevertheless ways to use client-side controls that do not give rise to any security vulnerabilities:
	- Client side scripts can be used to validate input as means of enhancing usability, avoiding the need for round-trip communication with server.
	- Sometimes client-side Data validation can be effective as security measure., for Dom based Vulnerability like DOM-XSS etc
	
# logging and Alerting
- if some employ bypass client side validation and submit data then this action should be alerted to admin so they can monitor suspicious activity etc

**NOTE:** 
 - In some cases where JavaScript is employed, the application still can be used by users who have disabled JavaScript within their browsers. In this situation, the browser simply skips JavaScript based form validation code, and the raw input entered by the user is submitted. To avoid false positives, the logging and alerting mechanism should be aware of where and how this can arise.

