# web browsers
 1. Firefox
 2. CHrome
# proxy 
1. Burp Suite
2. Zed Attack Proxy

**TIP:** some thick client directly communicate with the server outside the browser which we can't intercept but we change `/etc/hosts` file 
`127.0.0.1  attacking-web.com` to divert all traffic to the localhost and burp can listen on them

### FOr invisible traffic outside the browser
- Invisible mode proxying supports both HTTP and HTTPS requests. To prevent fatal certificate errors with SSL, it may be necessary to configure your invisible proxy listener to present an SSL certificate with a specific hostname which matches what the thick client expects. The following section has details on how you can avoid certificate problems caused by intercepting proxies.
- For each hostname you have redirected using your hosts file, configure Burp to resolve the hostname to its original IP address. These settings can be found under Options ÿ Connections ÿ Hostname Resolution. They let you specify custom mappings of domain names to IP addresses to override your computer’s own DNS resolution. This causes the outgoing requests from Burp to be directed to the correct destination server. (Without this step, the requests would be redirected to your own computer in an infinite loop.)

## Alternative to the intercepting Proxy
- some time its not possible to use proxy or some extension denied access to the proxy that need custom Corporate Proxy 
- here we user `browsers developer tools`

## Other tools
- nikto
- curl
- wget
- Stunnel - provide SSL tunnel for custom scripts that we built
- 