# Attacking Application Architecture
- security threat arise in environment where multiple applications are hosted on the same infrastructure, or even share common components of a wider overarching application,  so malicious  code run in one application may compromise entire environment

# Tiered Architecture
- common three tire architecture for web application
	- Presentation layer, which implements the application's interface
	- Application layer, which implements the core application logic
	- Data layer, which stores and process application's data
 - some enterprise applications employ a more fine grained division between tiers, depending on their logic or application 

# Attacking Tiered Architecture
top 3 ways to attacking tiered architecture
 1. exploiting Trust relationships between different tiers to advance an attack from one tier to another
 2. if different tiers are inadequetely segregated, we may be able to leverage defect within one tier to directly undercut the security protections implemented at another tier
 3. Having achieved a limited compromise of one tier, we may be able to directly attack the infrastructure supporting other tiers and therefore extended our compromise to those tiers

## Exploiting Trust Relationships between Tiers
- if we compromise 1 tier then some further exploitation is easy since there is trust relation between tiers which we can abuse to exploit it further, eg.  this does not need any high privilege to access further we abuse trust relation ship between tiers to exploit it
**TIP:** if attacker has `file-write access` ,try to write to the application's configuration or write to a hosted virtual directory to get command execution , see `nslookup ch 10`

### Using Local file inclusion to Execute Commands
- eg. application takes user input within `country` parameter in this following URL 
- `https://eis/mdsecportal/prefs/preference_2?country=en-gb`
- a user can modify the `country` parameter to include arbitrary files, 
- interesting method exploiting architectural quirk in PHP is that php session variable are written to file in cleartext, named using session token
- `/var/lib/php5/sess_98329579486578fb3y75r`
- may contain following content 
- `logged_in|i:1;id|s:2:”24”;username|s:11:”manicsprout”;nickname|s:22: “msp”;privilege|s:1:”1”;`
- attacker change his username to `<?php passsthru(id);?>` , and then he can include his session file to cause the `id` command to be executed using URL
- `http://eis/mdsecportal/prefs/preference_2.php?country=../../../../../../../../var/lib/php5/sess_9ceed0645151b31a494f4e52dabd0ed7%00`

#### Command execution by log poisoning previous example
#### Back door scripts example
 `http://net-square.com/papers/one_way/one_way.html#4.0`
 
 ## Shared hosting/ virtual hosting
 
 ### Attacks between ASP(application service providers) application components
 
 ## ATTACKING Cloud
- **TIP:** if we have access to run arbitrary PHP commands on server, use `phpinfo()` to return details of PHP environment's configuration, we review this for applyied defense

