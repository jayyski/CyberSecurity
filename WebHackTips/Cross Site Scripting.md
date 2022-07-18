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

## Testing Reflections to Introduce Script

You must manually investigate each instance of reflected input that you have identified to verify whether it is actually exploitable. In each location where data is reflected in the response, you need to identify the syntactic context of that data. You must find a way to modify your input such that, when it is copied into the same location in the application’s response, it results in execution of arbitrary script. Let’s look at some examples.

Example 1: A Tag Attribute Value
Suppose that the returned page contains the following:
 
```<input type=”text” name=”address1” value=”myxsstestdmqlwp”>```

One obvious way to craft an XSS exploit is to terminate the double quotation marks that enclose the attribute value, close the <input> tag, and then employ some means of introducing JavaScript, such as a <script> tag. For example:

 ```“><script>alert(1)</script>```

An alternative method in this situation, which may bypass certain input filters, is to remain within the <input> tag itself but inject an event handler containing JavaScript. For example:
 
```“ onfocus=”alert(1)```
 
## Probing Defense Filters

Very often, you will discover that the server modifies your initial attempted exploits in some way, so they do not succeed in executing your injected script. 
 
If this happens, do not give up! Your next task is to determine what server-side processing is occurring that is affecting your input. There are three broad possibilities:

 - The application (or a web application firewall protecting the application) has identified an attack signature and has blocked your input.

 - The application has accepted your input but has performed some kind of sanitization or encoding on the attack string.

 - The application has truncated your attack string to a fixed maximum length. We will look at each scenario in turn and discuss various ways in which the obstacles presented by the application’s processing can be bypassed.

## Beating Signature-Based Filters
 
 In the first type of filter, the application typically responds to your attack string
with an entirely different response than it did for the harmless string. For
example, it might respond with an error message, possibly even stating that a
possible XSS attack was detected, as shown in Figure 12-8.
 
 If this occurs, the next step is to determine what characters or expressions within your input are triggering the filter. An effective approach is to remove different parts of your string in turn and see whether the input is still being blocked. Typically, this process establishes fairly quickly that a specific expression such as <script> is causing the request to be blocked. You then need to test the filter to establish whether any bypasses exist. There are so many different ways to introduce script code into HTML pages that signature-based filters normally can be bypassed. You can find an alternative means of introducing script, or you can use slightly malformed syntax that browsers tolerate. This section examines the numerous different methods of executing scripts. Then it describes a wide range of techniques that can be used to bypass common filters.
 
 ## Bypassing Filters:HTML
 
Starting with the opening tag name, the most simple and naïve filters can be bypassed simply by varying the case of the characters used:

 ```<iMg onerror=alert(1) src=a>```

Going further, you can insert NULL bytes at any position:
```
<[%00]img onerror=alert(1) src=a>
<i[%00]mg onerror=alert(1) src=a>
```
(In these examples, [%XX] indicates the literal character with the hexadecimal ASCII code of XX. When submitting your attack to the application, generally you would use the URL-encoded form of the character. When reviewing the application’s response, you need to look for the literal decoded character being reflected.)

 TIP The NULL byte trick works on Internet Explorer anywhere within the HTML page. Liberal use of NULL bytes in XSS attacks often provides a quick way to bypass signature-based filters that are unaware of IE’s behavior. Using NULL bytes has historically proven effective against web application firewalls (WAFs) configured to block requests containing known attack strings. Because WAFs typically are written in native code for performance reasons, a NULL byte terminates the string in which it appears. This prevents the WAF from seeing the malicious payload that comes after the NULL (see Chapter 16 for more details)
 
 References: https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwim7cD06oL5AhWZk4kEHcDGAzMQFnoECAYQAQ&url=http%3A%2F%2Fwww.xss-payloads.com%2F&usg=AOvVaw1-hKbfrHEIcldlShn1bjoC
