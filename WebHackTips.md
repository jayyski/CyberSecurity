# File Extensions

File extensions used within URLs often disclose the platform or programming 
language used to implement the relevant functionality. 
For example:
```
  asp — Microsoft Active Server Pages
  aspx — Microsoft ASP.NET
  jsp — Java Server Pages
  cfm — Cold Fusion
  php — The PHP language
  d2w — WebSphere
  pl — The Perl language
  py — The Python language
  dll — Usually compiled native code (C or C++)
  nsf or ntf — Lotus Domino
```

# Attacking Authentication

```
1. Manually submit several bad login attempts for an account you control, 
monitoring the error messages you receive.

2. After about 10 failed logins, if the application has not returned a message 
about account lockout, attempt to log in correctly. If this succeeds, there 
is probably no account lockout policy.
 
3. If the account is locked out, try repeating the exercise using a different 
account. This time, if the application issues any cookies, use each cookie 
for only a single login attempt, and obtain a new cookie for each subsequent login attempt.

4. Also, if the account is locked out, see whether submitting the valid password causes any difference in the application’s behavior compared to an 
invalid password. If so, you can continue a password-guessing attack even 
if the account is locked out.

5. If you do not control any accounts, attempt to enumerate a valid username (see the next section) and make several bad logins using this. 
Monitor for any error messages about account lockout.

6. To mount a brute-force attack, first identify a difference in the application’s behavior in response to successful and failed logins. You can use 
this fact to discriminate between success and failure during the course of 
the automated attack.

7. Obtain a list of enumerated or common usernames and a list of common 
passwords. Use any information obtained about password quality rules to 
tailor the password list so as to avoid superfluous test cases.

8. Use a suitable tool or a custom script to quickly generate login requests 
using all permutations of these usernames and passwords. Monitor 
the server’s responses to identify successful login attempts. Chapter 14 
describes in detail various techniques and tools for performing customized attacks using automation.
 
9. If you are targeting several usernames at once, it is usually preferable 
to perform this kind of brute-force attack in a breadth-first rather than 
depth-first manner. This involves iterating through a list of passwords 
(starting with the most common) and attempting each password in turn 
on every username. This approach has two benefits. First, you discover 
accounts with common passwords more quickly. Second, you are less 
likely to trigger any account lockout defenses, because there is a time 
delay between successive attempts using each individual account.
```
## Verbose Failure Messages 

```
When a login attempt fails, you can of course infer that at least one piece of 
information was incorrect. However, if the application tells you which piece of 
information was invalid, you can exploit this behavior to considerably diminish 
the effectiveness of the login mechanism.

In the simplest case, where a login requires a username and password, an 
application might respond to a failed login attempt by indicating whether the 
reason for the failure was an unrecognized username or the wrong password, 
as illustrated in Figure 6-3

Ex: Applying a username and application resopnding: "User is not recognized"
```

## Enum Valid Usernames 
```
1. If you already know one valid username (for example, an account you 
control), submit one login using this username and an incorrect password, 
and another login using a random username.

 2. Record every detail of the server’s responses to each login attempt, 
including the status code, any redirects, information displayed onscreen, and any differences hidden in the HTML page source. Use your 
intercepting proxy to maintain a full history of all traffic to and from the 
server.

 3. Attempt to discover any obvious or subtle differences in the server’s 
responses to the two login attempts.

 4. If this fails, repeat the exercise everywhere within the application where 
a username can be submitted (for example, self-registration, password 
change, and forgotten password).

 5. If a difference is detected in the server’s responses to valid and invalid 
usernames, obtain a list of common usernames. Use a custom script or 
automated tool to quickly submit each username, and filter the responses 
that signify that the username is valid.

 6. Before commencing your enumeration exercise, verify whether the application performs any account lockout after a certain number of failed login 
attempts (see the preceding section). If so, it is desirable to design your 
enumeration attack with this fact in mind. For example, if the application 
will grant you only three failed login attempts with any given account, you 
run the risk of “wasting” one of these for every username you discover 
through automated enumeration. Therefore, when performing your enumeration attack, do not submit a far-fetched password with each login 
attempt. Instead, submit either a single common password such as password1 or the username itself as the password. If password quality rules 
are weak, it is highly likely that some of the attempted logins you perform 
as part of your enumeration exercise will succeed and will disclose both 
the username and password in a single hit. To set the password field to 
be the same as the username, you can use the “battering ram” attack 
mode in Burp Intruder to insert the same payload at multiple positions in 
your login request.
```

## Password Change Functionality

```
 Vulnerabilities that are deliberately avoided in the main login function often reappear in the 
password change function. Many web applications’ password change functions 
are accessible without authentication and do the following:

  -Provide a verbose error message indicating whether the requested username is valid.
  
  -Allow unrestricted guesses of the “existing password” field.
  
  -Check whether the “new password” and “confi rm new password” fields
have the same value only after validating the existing password, thereby 
allowing an attack to succeed in discovering the existing password 
noninvasively.

 1. Identify any forgotten password functionality within the application. If 
this is not explicitly linked from published content, it may still be implemented (see Chapter 4).

 2. Understand how the forgotten password function works by doing a 
complete walk-through using an account you control.

 3. If the mechanism uses a challenge, determine whether users can set or 
select their own challenge and response. If so, use a list of enumerated or 
common usernames to harvest a list of challenges, and review this for any 
that appear easily guessable.

 4. If the mechanism uses a password “hint,” do the same exercise to harvest 
a list of password hints, and target any that are easily guessable.

 5. Try to identify any behavior in the forgotten password mechanism that 
can be exploited as the basis for username enumeration or brute-force 
attacks (see the previous details).

 6. If the application generates an e-mail containing a recovery URL in 
response to a forgotten password request, obtain a number of these URLs, 
and attempt to identify any patterns that may enable you to predict the 
URLs issued to other users. Employ the same techniques as are relevant to 
analyzing session tokens for predictability (see Chapter 7).

```
