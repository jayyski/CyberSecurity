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
The first stage in the testing process is to submit a benign string to each entry point and to identify every location in the response where the string is reflected:
 
 1. Choose a unique arbitrary string that does not appear anywhere within the application and that contains only alphabetical characters and there-fore is unlikely to be affected by any XSS-specific filters. For example:

       ```myxsstestdmqlwp```

Submit this string as every parameter to every page, targeting only one parameter at a time.

 2. Monitor the application’s responses for any appearance of this same string. Make a note of every parameter whose value is being copied into the application’s response. These are not necessarily vulnerable, but each instance identified is a candidate for further investigation, as described in the next section.

 3. Note that both GET and POST requests need to be tested. You should include every parameter within both the URL query string and the message body. Although a smaller range of delivery mechanisms exists for XSS vulnerabilities that can be triggered only by a POST request, exploitation is still possible, as previously described.

 4. In any cases where XSS was found in a POST request, use the “change request method” option in Burp to determine whether the same attack could be performed as a GET request.

 5. In addition to the standard request parameters, you should test every instance in which the application processes the contents of an HTTP request header. A common XSS vulnerability arises in error messages, where items such as the Referer and User-Agent headers are copied into the message’s contents. These headers are valid vehicles for delivering a reflected XSS attack, because an attacker can use a Flash object to induce a victim to issue a request containing arbitrary HTTP headers.

 
## Identifying Reflections of User Input
The first stage in the testing process is to submit a benign string to each entry point and to identify every location in the response where the string is reflected.

1. Choose a unique arbitrary string that does not appear anywhere within the application and that contains only alphabetical characters and there-
fore is unlikely to be affected by any XSS-specific filters. For example:

     ```myxsstestdmqlwp```

  Submit this string as every parameter to every page, targeting only one parameter at a time.

2. Monitor the application’s responses for any appearance of this same string. Make a note of every parameter whose value is being copied into the application’s response. These are not necessarily vulnerable, but each instance identified is a candidate for further investigation, as described in the next section.

3. Note that both GET and POST requests need to be tested. You should include every parameter within both the URL query string and the message body. Although a smaller range of delivery mechanisms exists for XSS vulnerabilities that can be triggered only by a POST request, exploitation is still possible, as previously described.

4. In any cases where XSS was found in a POST request, use the “change request method” option in Burp to determine whether the same attack could be performed as a GET request.

5. In addition to the standard request parameters, you should test every instance in which the application processes the contents of an HTTP request header. A common XSS vulnerability arises in error messages, where items such as the Referer and User-Agent headers are copied into the message’s contents. These headers are valid vehicles for delivering a reflected XSS attack, because an attacker can use a Flash object to induce a victim to issue a request containing arbitrary HTTP headers.
