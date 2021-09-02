# NoSQL injection
- they don't have specific schema for database and also syntax is differ 
- commmon query methods used by NoSQL data stored
	- Key/Value lookup
	- XPATH 
	- Programming Language such as JavaScript
- exploitable vulnerabilities arise in how NOSQL data stores are used in web-applications

# Injecting into MongoDB
- the value pass through JS function which we can take advantage 
- eg
```js
$obj=$collection->findOne(array('$where'=> $js));
if(isset($obj["uid"]))
{
$logged_in=1;  }
else{ $logged_in=0;}
```
we can bypass login by supplying `username' //` in user name field which cause rest field after username is comment and does not check for Password

# Injecting Into XPATH
- xml path language is interpreted language used to navigate around xml documents and to retrieve data from within them.
```xml
<addressBook><address><firstname>DK</firstname>
	<password>hello</password>
	</address></addressBook>
```

here we simply inject address/email/text() to manipulate query and retrieve all the data 
query used to return all details of user=dk
```
//address[surname/text()='dk']
```

## Subverting Application logic
- application function that retrieves user's stored CC number based on `username and password`
- following XPATH query verifies user supplyied credentials
 `//address[surname/text()='dk'and password/text()='hello']/card/text()`
- in this case we bypass it same as SQLI 'or '1'='1
- resultent XPATH
`//address[surname/text()='dk' and password/text()=''or '1'='1']/card/text()`

## Informed XPATH Injection
- XPath language can also make conditional response like SQLI , and it also support `substring` function so we retrive data one byte at time
```bash
' or //address[surname/text()='gates' and substring(password/text(),1,1)='M'] and 'a'='a
```

# Blind XPath Injection
- XPath queries can contain steps that are relative to the current node within the XML document, so from the current node, it is possible to navigate to the parent node or to a specific child node,
- XPath contains function to query meta-information about the document, including the name of the specific element,using these tech, extract the names and values of all nodes.
```bash
'or substring(name(parent::*[position()=1]),1,1)='a
```
this input generates result, because the first letter of the address node is a, 
- this way we extracted address node, and then we cycle through each of its child nodes,extracting all names and values
```bash
//address[position()=3]/child::node()[position()=4]/text()
```
- this technique can be used in a completely blind attack, where no result are returned within the application's responses, by crafting an injected condition that specifies the target node by index,
```bash
' or substring(//address[position()=1]/child::node()[position()=6]/text(),1,1)='M' and 'a'='a
```
 by supplying above payload, if the result returned then we know that 1st char of Gates password is `M`
 
 **TIP:**
   - Xpath contains 2 use full functions that can help us to automate the preceding attack and quickly iterate through all nodes and data in the xml doc
	   - `count()` returns number of child nodes of given element, which can be used to determine range of position() values to iterate over
	   - `string-length()` returns the length of a supplied string, which can be used to determine the range of `substring()` values to iterate over


# Finding XPath Injection Flows
- probing with `'` and `'--` this will invalidate the XPath query and generates Error
- one or more of following strings typically result in some change in the application's behavior with out causing error, same way they do in `relation to SQLI`
```bash
' or 'a'='a
' and 'a'='b
or 1=1
and 1=2
```

## HACK STEP
  1. Try submitting following values, and determine whether these result in different application behavior without causing an error
	  1. `' or count(parent::*[position()=1])=0 or 'a'='b`
	  2. `' or count(parent::*[position()=1])>0 or 'a'='b`
	  3. if parameter is numeric, also try the following test strings
	  4. `1 or count(parent::*[position()=1])=0`
	  5. `1 or count(parent::*[position()=1])>0`
   2. if any of the preceding strings cause differential behavior within the application without causing error, it is likely that you can extract arbitrary data by crafting test conditions to extract one byte of information at a time, use a series of conditions with the following form to determine name of the current node's parent
	   1. `substring(name(parent::*[position()=1],1,1)='a'`
	3. Having extracted name of parent node, use a series of conditions with following form to extract all the data within XML tree
		1. `substring(//parentnodename[position()=1]/child::node()[position()=1/text(),1,1)='a'`

# Preventing XPath Injection
- filter `() = '[]:,*/'` this character before taking it input from the user

# Injecting Into LDAP
- the Light Weight Directory Access Protocol(LDAP) is used to access directory service over a network.
- it may contain personal data i.e, names,telephone no, email addr, job function 
- common examples of LDAP are Active Directory used within windows domain, and OpenLDAP used in various situation
- Each LDAP query uses one or more search filters, which determine directory entries that are returned by query,
- most common search filters are
	- **Simple Match condition:** match the value of a single attribute, eg. application that searches for user via his username might use filter `(username=daf)`
	- **Disjunctive queries:** specify multiple conditions, any one of which must be satisfied by entries returned, eg. search function looks up a user-supplied search term  might use `(|(cn=searchterm)(sn=searchterm)(ou=searchterm))`
	- **Conjunctive queries:** specify multiple conditions, all of which must be satisfied by entries returned ,eg. login mechanism `(&(username=dk)(password=secret)`

- In general LDAP injection vulnerability are not as readily exploitable as SQLI due to 
	- search filter employs logical operator to specify a conjunctive or disjunctive query, this usually appears before the point where user-supplied data is inserted and therefor cannot be modified, hence simple match conditions and conjunctive queries don't have an equivalent to `'or1=1'` types of attack that arise with SQLI
	- in LDAP implementation that are in common use, the directory attributes to be returned are passed to LDAP API's as a separate parameter from the search filter and normally are hard-coded within application, hence not possible to manipulate user-supplied input to retrieve different attributes than the query was intended to retrieve
	- Applications rarely return informative error messages, so vulnerability need to be exploited `Blind`

# Exploiting LDAP Injection
## Disjunctive Queries
- eg. application lets users list employees within a special department of the business, search result are restricted to the geographic locations, that the user is authorized to view,
	- if application performs following disjunctive query
	- `(|(department=London sales)(department=reading sales))`
	- in this can be exploited to see all locations by submitting
	- `)(department=*`
	- this `*` is wild card, this query is formed
	- `(|(department=London)(department=*)(department=Reading)(department=*))`
	
## Conjunctive Queries
- application function the allows user to search for employee by name, again within geo location region,
	- `(&(givenName=dk)(department=London*))`
	- if we give another input as
	- `*))(&(givenName=daf`
	- this result in embedded into the original search filter, it become
	- `(&(givenName=*))(&(givenName=daf)(department=London*))`
- this now contain two search filters, 1st contains single wildcard, thus the details of all employees are returned from all location, thereby bypassing access control

**NOTE:** this technique if injecting a second search fileter is also effective  against simple match conditions that do not employ any logical operator, provided that the back-end implementation accepts multiple search filters

- 2nd type of attack against conjunctive queries exploits how many LDAP implementations handle `NULL` bytes, because these implementations typically are written in native Code ,` NULL Byte` within a search filter effectively terminates the string, and any characters coming after the `NULL BYte` are ignored.
	- eg if we supply following input
	- `*))%00` 
	- the sequence is decoded by the application server into a literal `NULL BYte`
	- `(&(givenName=*))[NULL](department=London*))`
	- this return all employee details by commenting query after `[NULL]`

# Finding LDAP Injection Flows
- supplying invalid input in LDAP operation typically does not result in an informative Error message, in general, the evidence available to you in diagonosing vulnerability including the results returned by search function & occurance of error i.e HTTP 500 

### HACK STEP 
  1. Try entering just `*` as search term, this character function as wild card in LDAP but not in SQL, if large number of results are returned , it indicates it dealing with LDAP query
  2. try entering number of closing brackets
	  1. `)))))))))`
  3. This input closes any brackets enclosing your input, as well as those that encapsulate the main search filter itself. This results in unmatched closing brackets, thus invalidating the query syntax. If an error results, the application may be vulnerable to LDAP injection. (Note that this input may also break many other kinds of application logic, so this provides a strong indicator only if you are already confident that you  re dealing with an LDAP query.)
  4. Try entering various expression designed to interfere with different types of queries , and see if it infulence result
```bash
)(cn=*
*))(|(cn=*
*))%00
```

# Preventing LDAP injection
- if it is necessary to inject input in LDAP query then pur filters to remove harmfull charecter like
- `( ) ; , * | & = `

