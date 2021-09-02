# Attacking Native Compiled Application
- web application that run rub on hardware device such as printer and switches often contains some native code, like C & C++ 
- books
	-  The Shellcoder’s Handbook, 2nd Edition, by  chris Anley, John Heasman, Felix Linder, and Gerardo Richarte (Wiley, 2007)
	-  The Art of Software Security Assessment by Mark Dowd, John McDonald, and Justin Schuh (Addison-Wesley, 2006)
	-  Gray Hat Hacking, 2nd Edition, by Shon Harris, Allen Harper, Chris Eagle, and Jonathan Ness (McGraw-Hill Osborne, 2008)



# Buffer OverFlow Vulnerability
- it arise when application copies user controllable data into a memory buffer that is not sufficiently large to accommodate it, resulting in adjacement memory being overwritten with user's data.

## Stack Overflows
-  where lang use `strcpy` in user supplied input

## heap Overflow
- when destination buffer is allocated on the heap, not in the stack

### EXploitation of blind buffer 
www.ngssoftware.com/papers/NISR.BlindExploitation.pdf

# Integer Vulnerability
## Integer Overflows
- it occur when operation on integer values causes it to increase above its maximum possible value or decrease below its minimum possible value, when this happens the number wraps, so large number becomes very small or vice versa

## Signedness Errors
- arise when application use both signed and unsigned integers to measure the lengths of buffers and confuse them at some point,

## Detecting integer Vulnerabilities
- primary location to probe for integer vuln is instance where an integer values is submitted from the client to the server
	- application pass integer values in normal way as param within query string,cookie or msgbody, the most likely targets for testing are field that appear to represent the length of a string that is also being submitted
	-  The application may pass integer values embedded within a larger blob of binary data. This data may originate from a client-side component such as an ActiveX control, or it may have been transmitted via the client in a hidden form field or cookie (see Chapter 5). Length-related integers may be harder to identify in this context. They typically are represented in hexadecimal form and often directly precede the string or buffer to which they relate. Note that binary data may be encoded using Base64 or similar schemes for transmission over HTTP

### HACK  STEP
  1. having identifies targets for testing, we need to submit suitable payloads designed to trigger any vulnerabilities, for each item of data being targeted, send a series of different values in turn, representing boundary cases for the signed and unsigned versions of different sizes of integer. For example:
	  1.  0x7f and 0x80 (127 and 128)
	  2.  0xff and 0x100 (255 and 256)
	  3.  0x7ffff and 0x8000 (32767 and 32768)
	  4.  0xffff and 0x10000 (65535 and 65536)
	  5. 0x7fffffff and 0x80000000 (2147483647 and 2147483648)
	  6. 0xffffffff and 0x0 (4294967295 and 0)
   2. monitor application's responses for anomalous events

# Format String Vulnerability
- it arise when user controllable input is passwd as the format string prameter to a function that takes format specification that may be misused ,eg, printf('%d',variable)

## Detecting Format String Vulnerabilities
- most reliable way to detect format string bugs in remote application is to submit data containing various format specifiers and monitor for anomalies in the application's behavior
### HACK STEP
  1. targeting each parameter in turn, submit steing containing large number of format specifiers ``%n`` and ``%s`` i.e
 ```bash
 %n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n
 %s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s
 ```
  3. the windows `FormatMessage` function uses specifiers in a different way than the printf family , to Test for vulnerable calls to this function, we use following strings
  ```bash
  %1!n!%2!n!%3!n!%4!n!%5!n!%6!n!%7!n!%8!n!%9!n!%10!n! etc...
  %1!s!%2!s!%3!s!%4!s!%5!s!%6!s!%7!s!%8!s!%9!s!%10!s! etc...
  ```
  4. remember to `URL Encode` the `%` character as `%25`
  5. monitor the application's response

**NOTE:** In contrast to most other types of web application vulnerabilities, even the act of probing for classic software flaws is quite likely to cause a denial-of-service condition if the application is vulnerable. Before performing any such testing, you should ensure that the application owner accepts the inherent risks involved


   
