## Finding and Exploiting XSS Vulnerabilities

A basic approach to identifying XSS vulnerabilities is to use a standard proof-of-concept attack string such as the following:
```
“><script>alert(document.cookie)</script>
```
This string is submitted as every parameter to every page of the application, and responses are monitored for the appearance of this same string. If cases are found where the attack string appears unmodified within the response, the application is almost certainly vulnerable to XSS.

If your intention is simply to identify some instance of XSS within the application as quickly as possible to launch an attack against other application users, this basic approach is probably the most effective, because it can be easily auto- mated and produces minimal false positives. However, if your objective is to perform a comprehensive test of the application to locate as many individual vulnerabilities as possible, the basic approach needs to be supplemented with more sophisticated techniques. There are several different ways in which XSS vulnerabilities may exist within an application that will not be identified via the basic approach to detection:

- Many applications implement rudimentary blacklist-based filters in an attempt to prevent XSS attacks. These filters typically look for expressions such as <script> within request parameters and take some defensive action such as removing or encoding the expression or blocking the request. These filters often block the attack strings commonly employed in the basic approach to detection. However, just because one common attack string is being filtered, this does not mean that an exploitable vulnerability does not exist. As you will see, there are cases in which a working XSS exploit can be created without using <script> tags and even without using commonly filtered characters such as “ < > and /.
 
- The anti-XSS filters implemented within many applications are defective and can be circumvented through various means. For example, suppose that an application strips any <script> tags from user input before it is processed. This means that the attack string used in the basic approach will not be returned in any of the application’s responses. However, it may be that one or more of the following strings will bypass the filter and result in a successful XSS exploit:
 
```  
“><script >alert(document.cookie)</script >
“><ScRiPt>alert(document.cookie)</ScRiPt>
“%3e%3cscript%3ealert(document.cookie)%3c/script%3e
“><scr<script>ipt>alert(document.cookie)</scr</script>ipt>
%00“><script>alert(document.cookie)</script>
```
