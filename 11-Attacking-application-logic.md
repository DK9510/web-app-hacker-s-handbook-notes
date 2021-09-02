# Attacking Application logic
- application logic vulnerability is hardest one to determine, because it can be anywhere, in application
- also it depends on the functionality of the application

# Nature of Logic Flaws
- for logic flaws we need to think differenly , like what happened if we break chain of requesting, or we directly make request without doing certain request etc
- developer may assume some thing that can be violated

# Real World Logic Flaws
-  **Example:1** Asking the Oracle that encrypt data with same algo and key that is used to encrypt session token or cookie, here an attacker set cookie that may decrypt in some where in application where, user can see decrypted out put.
-  **Example: 2**: Fooling a Password change function, sometime application procced the fucntion without verifying old password**, or old password is supplied or not etc.
**TIP:** always remove 1 parameter at time to trace the logic of application that is doing

 - **Example: 3:** Proceeding TO Checkout , try forced browsing in some request to bypass some stage that is necessary
 -  application truncating it to 128 characters , and also there is sql defense like application add another `'` after we add `'` so to bypass this we pass `aaaaaaa[127]'` 127 `a` followed by `'` so now our string 128 char and application can not add extra`'` to escape and final gives us sql error
