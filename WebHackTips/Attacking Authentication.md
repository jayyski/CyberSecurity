# Attacking Authentication


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

## Verbose Failure Messages 


When a login attempt fails, you can of course infer that at least one piece of 
information was incorrect. However, if the application tells you which piece of 
information was invalid, you can exploit this behavior to considerably diminish 
the effectiveness of the login mechanism.

In the simplest case, where a login requires a username and password, an 
application might respond to a failed login attempt by indicating whether the 
reason for the failure was an unrecognized username or the wrong password, 
as illustrated in Figure 6-3

Ex: Applying a username and application resopnding: "User is not recognized"

## Enum Valid Usernames 

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


## Password Change Functionality


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
this is not explicitly linked from published content, it may still be implemented (see Chapter 4).

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


## Incomplete Validation of Credentials

Well-designed authentication mechanisms enforce various requirements on 
passwords, such as a minimum length or the presence of both uppercase and 
lowercase characters. Correspondingly, some poorly designed authentication 
mechanisms not only do not enforce these good practices but also do not take 
into account users’ own attempts to comply with them.

For example, some applications truncate passwords and therefore validate 
only the first n characters. Some applications perform a case-insensitive check 
of passwords. Some applications strip unusual characters (sometimes on the 
pretext of performing input validation) before checking passwords. In recent 
times, behavior of this kind has been identified in some surprisingly high-profile 
web applications, usually as a result of trial and error by curious users.

Each of these limitations on password validation reduces by an order of 
magnitude the number of variations available in the set of possible passwords. 
Through experimentation, you can determine whether a password is being 
fully validated or whether any limitations are in effect. You can then fine-tune 
your automated attacks against the login to remove unnecessary test cases, 
thereby massively reducing the number of requests necessary to compromise 
user accounts.

1. Using an account you control, attempt to log in with variations on your 
own password: removing the last character, changing the case of a character, and removing any special typographical characters. If any of these 
attempts is successful, continue experimenting to try to understand what 
validation is actually occurring.

2. Feed any results back into your automated password-guessing attacks to 
remove superfluous test cases and improve the chances of success.


## Nounique Usernames
Some applications that support self-registration allow users to specify their 
own username and do not enforce a requirement that usernames be unique. 
Although this is rare, the authors have encountered more than one application 
with this behavior.

This represents a design fl aw for two reasons:
  One user who shares a username with another user may also happen to 
select the same password as that user, either during registration or in a 
subsequent password change. In this eventuality, the application either 
rejects the second user’s chosen password or allows two accounts to 
have identical credentials. In the fi rst instance, the application’s behavior 
effectively discloses to one user the credentials of the other user. In the 
second instance, subsequent logins by one of the users result in access to 
the other user’s account.

  An attacker may exploit this behavior to carry out a successful brute-force 
attack, even though this may not be possible elsewhere due to restrictions 
on failed login attempts. An attacker can register a specific username 
multiple times with different passwords while monitoring for the differential response that indicates that an account with that username 
and password already exists. The attacker will have ascertained a target 
user’s password without making a single attempt to log in as that user.
Badly designed self-registration functionality can also provide a means for 
username enumeration. If an application disallows duplicate usernames, an 
attacker may attempt to register large numbers of common usernames to identify the existing usernames that are rejected.

1. If self-registration is possible, attempt to register the same username 
twice with different passwords.

2. If the application blocks the second registration attempt, you can exploit 
this behavior to enumerate existing usernames even if this is not possible 
on the main login page or elsewhere. Make multiple registration attempts 
with a list of common usernames to identify the already registered names 
that the application blocks.

3. If the registration of duplicate usernames succeeds, attempt to register 
the same username twice with the same password, and determine the 
application’s behavior:
  a. If an error message results, you can exploit this behavior to carry out a 
brute-force attack, even if this is not possible on the main login page. 
Target an enumerated or guessed username, and attempt to register 
this username multiple times with a list of common passwords. When 
the application rejects a specific password, you have probably found 
the existing password for the targeted account.
  b. If no error message results, log in using the credentials you specified, and see what happens. You may need to register several users, 
and modify different data held within each account, to understand 
whether this behavior can be used to gain unauthorized access to 
other users’ accounts


## Predictable Usernames 
Some applications automatically generate account usernames according to 
a predictable sequence (cust5331, cust5332, and so on). When an application 
behaves like this, an attacker who can discern the sequence can quickly arrive 
at a potentially exhaustive list of all valid usernames, which can be used as 
the basis for further attacks

1. If the application generates usernames, try to obtain several in quick 
succession, and determine whether any sequence or pattern can be 
discerned.

2. If it can, extrapolate backwards to obtain a list of possible valid usernames. This can be used as the basis for a brute-force attack against the 
login and other attacks where valid usernames are required, such as the 
exploitation of access control flaws.


## Predictable Passwords

In some applications, users are created all at once or in sizeable batches and are
automatically assigned initial passwords, which are then distributed to them
through some means. The means of generating passwords may enable an attacker
to predict the passwords of other application users. This kind of vulnerability is
more common on intranet-based corporate applications — for example, where
every employee has an account created on her behalf and receives a printed
notification of her password.
In the most vulnerable cases, all users receive the same password, or one
closely derived from their username or job function. In other cases, generated
passwords may contain sequences that could be identifi ed or guessed with
access to a very small sample of initial passwords

1. If the application generates passwords, try to obtain several in quick
succession, and determine whether any sequence or pattern can be
discerned.
 
2. If it can, extrapolate the pattern to obtain a list of passwords for other
application users.

3. If passwords demonstrate a pattern that can be correlated with usernames, you can try to log in using known or guessed usernames and the
corresponding inferred passwords.

4. Otherwise, you can use the list of inferred passwords as the basis for a
brute-force attack with a list of enumerated or common usernames.

## Insecure Distribution of Credentials


1. Obtain a new account. If you are not required to set all credentials during
registration, determine the means by which the application distributes
credentials to new users.

2. If an account activation URL is used, try to register several new accounts
in close succession, and identify any sequence in the URLs you receive.
If a pattern can be determined, try to predict the activation URLs sent to
recent and forthcoming users, and attempt to use these URLs to take ownership of their accounts.

3. Try to reuse a single activation URL multiple times, and see if the application allows this. If not, try locking out the target account before reusing
the URL, and see if it now works.

## Implementation Flaws in Authentication


1. Perform a complete, valid login using an account you control. Record
every piece of data submitted to the application, and every response
received, using your intercepting proxy.

2. Repeat the login process numerous times, modifying pieces of the data
submitted in unexpected ways. For example, for each request parameter
or cookie sent by the client, do the following:
 - Submit an empty string as the value.
 - Remove the name/value pair altogether.
 - Submit very long and very short values.
 - Submit strings instead of numbers and vice versa.
 - Submit the same item multiple times, with the same and different
values.

3. For each malformed request submitted, review closely the application’s
response to identify any divergences from the base case.

4. Feed these observations back into framing your test cases. When one
modification causes a change in behavior, try to combine this with other
changes to push the application’s logic to its limits.

## Defects in Multi Stage Authentication


1. Perform a complete, valid login using an account you control. Record every
piece of data submitted to the application using your intercepting proxy.

2. Identify each distinct stage of the login and the data that is collected at
each stage. Determine whether any single piece of information is collected
more than once or is ever transmitted back to the client and resubmitted
via a hidden form field, cookie, or preset URL parameter (see Chapter 5).

3. Repeat the login process numerous times with various malformed
requests:
 a. Try performing the login steps in a different sequence.
 b. Try proceeding directly to any given stage and continuing from there.
 c. Try skipping each stage and continuing with the next.
 d. Use your imagination to think of other ways to access the different
stages that the developers may not have anticipated.

4. If any data is submitted more than once, try submitting a different value
at different stages, and see whether the login is still successful. It may
be that some of the submissions are superfluous and are not actually
processed by the application. It might be that the data is validated at one
stage and then trusted subsequently. In this instance, try to provide the
credentials of one user at one stage, and then switch at the next to actually authenticate as a different user. It might be that the same piece of
data is validated at more than one stage, but against different checks. In
this instance, try to provide (for example) the username and password of
one user at the first stage, and the username and PIN of a different user
at the second stage.
 
5. Pay close attention to any data being transmitted via the client that was
not directly entered by the user. The application may use this data to store
information about the state of the login progress, and the application may
trust it when it is submitted back to the server. For example, if the request
for stage three includes the parameter stage2complete=true, it may
be possible to advance straight to stage three by setting this value. Try to
modify the values being submitted, and determine whether this enables
you to advance or skip stages.

### Question & Answer Authentication Functionality

The application may present a randomly chosen question and store the
details within a hidden HTML form field or cookie, rather than on the
server. The user subsequently submits both the answer and the question
itself. This effectively allows an attacker to choose which question to
answer, enabling the attacker to repeat a login after capturing a user’s
input on a single occasion.

The application may present a randomly chosen question on each login
attempt but not remember which question a given user was asked if he
or she fails to submit an answer. If the same user initiates a fresh login
attempt a moment later, a different random question is generated. This
effectively allows an attacker to cycle through questions until he receives
one to which he knows the answer, enabling him to repeat a login having
captured a user’s input on a single occasion.

The second of these conditions is really quite subtle, and as a result,
many real-world applications are vulnerable. An application that challenges a
user for two random letters of a memorable word may appear at fi rst glance
to be functioning properly and providing enhanced security. However, if the
letters are randomly chosen each time the previous authentication stage is
passed, an attacker who has captured a user’s login on a single occasion can
simply reauthenticate up to this point until the two letters that he knows are
requested, without the risk of account lockout.

1. If one of the login stages uses a randomly varying question, verify whether
the details of the question are being submitted together with the answer.
If so, change the question, submit the correct answer associated with that
question, and verify whether the login is still successful.
 
2. If the application does not enable an attacker to submit an arbitrary
question and answer, perform a partial login several times with a single
account, proceeding each time as far as the varying question. If the question changes on each occasion, an attacker can still effectively choose
which question to answer


## Insecure Storage of Credientials

If an application stores login credentials insecurely, the security of the login
mechanism is undermined, even though there may be no inherent fl aw in the
authentication process itself.

It is common to encounter web applications in which user credentials are
stored insecurely within the database. This may involve passwords being
stored in cleartext. But if passwords are being hashed using a standard algorithm such as MD5 or SHA-1, this still allows an attacker to simply look up
observed hashes against a precomputed database of hash values


1. Review all of the application’s authentication-related functionality, as well
as any functions relating to user maintenance. If you find any instances in
which a user’s password is transmitted back to the client, this indicates
that passwords are being stored insecurely, either in cleartext or using
reversible encryption.

2. If any kind of arbitrary command or query execution vulnerability is
identified within the application, attempt to find the location within the
application’s database or filesystem where user credentials are stored:
 a. Query these to determine whether passwords are being stored in
unencrypted form.
 b. If passwords are stored in hashed form, check for nonunique values, indicating that an account has a common or default password
assigned, and that the hashes are not being salted.
 c. If the password is hashed with a standard algorithm in unsalted form,
query online hash databases to determine the corresponding cleartext
password value.


