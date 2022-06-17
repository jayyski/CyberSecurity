##  Weaknesss in Token Generation

Session management mechanisms are often vulnerable to attack because tokens are generated in an unsafe manner that enables an attacker to identify the values of tokens that have been issued to other users.

There are numerous locations where an application’s security depends on the unpredictability of tokens it generates. Here are some examples:
- Password recovery tokens sent to the user’s registered e-mail address n Tokens placed in hidden form ﬁ elds to prevent cross-site request forgery attacks (see Chapter 13) 
- Tokens used to give one-time access to protected resources 
- Persistent tokens used in “remember me” functions  
- Tokens allowing customers of a shopping application that does not use authentication to retrieve the current status of an existing order

The considerations in this chapter relating to weaknesses in token generation apply to all these cases. In fact, because many of today’s applications rely on mature platform mechanisms to generate session tokens, it is often in these other areas of functionality that exploitable weaknesses in token generation are found.

## Meaningful Tokens 

Some session tokens are created using a transformation of the user’s username or e-mail address, or other information associated with that person. This information may be encoded or obfuscated in some way and may be combined with other data. For example, the following token may initially appear to be a long random string:
```
757365723d6461663b6170703d61646d696e3b646174653d30312f31322f3131 
```
However, on closer inspection, you can see that it contains only hexadecimal characters. Guessing that the string may actually be a hex encoding of a string of ASCII characters, you can run it through a decoder to reveal the following:
```
user=daf;app=admin;date=10/09/11
```
Attackers can exploit the meaning within this session token to attempt to guess the current sessions of other application users. Using a list of enumerated or common usernames, they can quickly generate large numbers of potentially valid tokens and test these to conﬁ rm which are valid. Tokens that contain meaningful data often exhibit a structure. In other words, they contain several components, often separated by a delimiter, that can be extracted and analyzed separately to allow an attacker to understand their function and means of generation. Here are some components that may be encountered within structured tokens: 
- The account username 
- The numeric identiﬁer that the application uses to distinguish between accounts 
- The user’s ﬁrst and last names 
- The user’s e-mail address 
- The user’s group or role within the application 
- A date/time stamp 
- An incrementing or predictable number 
- The client IP address 
Each different component within a structured token, or indeed the entire token, may be encoded in different ways. This can be a deliberate measure to obfuscate their content, or it can simply ensure safe transport of binary data via HTTP. Encoding schemes that are commonly encountered include XOR, Base64, and hexadecimal representation using ASCII characters (see Chapter 3). It may be necessary to test various decodings on each component of a structured token to unpack it to its original form

When an application handles a request containing a structured token, it may not actually process every component with the token or all the data contained in each component. In the previous example, the application may Base64-decode the token and then process only the “user” and “date” components. In cases where a token contains a blob of binary data, much of this data may be padding. Only a small part of it may actually be relevant to the validation that the server performs on the token. Narrowing down the subparts of a token that are actually required can often considerably reduce the amount of apparent entropy and complexity that the token contains.

HACK STEPS
1. Obtain a single token from the application, and modify it in systematic ways to determine whether the entire token is validated or whether some of its subcomponents are ignored. Try changing the token’s value one byte at a time (or even one bit at a time) and resubmitting the modified token to the application to determine whether it is still accepted. If you find that certain portions of the token are not actually required to be correct, you can exclude these from any further analysis, potentially reducing the amount of work you need to perform. You can use the “char frobber” payload type in Burp Intruder to modify a token’s value in one character position at a time, to help with this task. 

2. Log in as several different users at different times, and record the tokens received from the server. If self-registration is available and you can choose your username, log in with a series of similar usernames containing small variations between them, such as A, AA, AAA, AAAA, AAAB, AAAC, AABA, and so on. If other user-specific data is submitted at login or stored in user profiles (such as an e-mail address), perform a similar exercise to vary that data systematically, and record the tokens received following login. 

3. Analyze the tokens for any correlations that appear to be related to the username and other user-controllable data. 

4. Analyze the tokens for any detectable encoding or obfuscation. Where the username contains a sequence of the same character, look for a corresponding character sequence in the token, which may indicate the use of XOR obfuscation. Look for sequences in the token containing only hexadecimal characters, which may indicate a hex encoding of an ASCII string or other information. Look for sequences that end in an equals sign and/or that contain only the other valid Base64 characters: a to z, A to Z, 0 to 9, +, and /.  

5. If any meaning can be reverse-engineered from the sample of session tokens, consider whether you have sufficient information to attempt to guess the tokens recently issued to other application users. Find a page of the application that is session-dependent, such as one that returns an error message or a redirect elsewhere if accessed without a valid session. Then use a tool such as Burp Intruder to make large numbers of requests to this page using guessed tokens. Monitor the results for any cases in which the page is loaded correctly, indicating a valid session token.

## Predictable Tokens

Some session tokens do not contain any meaningful data associating them with a particular user. Nevertheless, they can be guessed because they contain sequences or patterns that allow an attacker to extrapolate from a sample of tokens to ﬁ nd other valid tokens recently issued by the application. Even if the extrapolation involves some trial and error (for example, one valid guess per 1,000 attempts), this would still enable an automated attack to identify large numbers of valid tokens in a relatively short period of time. Vulnerabilities relating to predictable token generation may be much easier to discover in commercial implementations of session management, such as web servers or web application platforms, than they are in bespoke applications. When you are remotely targeting a bespoke session management mechanism, your sample of issued tokens may be restricted by the server’s capacity, the activity of other users, your bandwidth, network latency, and so on. In a laboratory environment, however, you can quickly create millions of sample tokens, all precisely sequenced and time-stamped, and you can eliminate interference caused by other users. In the simplest and most brazenly vulnerable cases, an application may use a simple sequential number as the session token. In this case, you only need to obtain a sample of two or three tokens before launching an attack that will quickly capture 100% of currently valid sessions.

In other cases, an application’s tokens may contain more elaborate sequences that take some effort to discover. The types of potential variations you might encounter here are open-ended, but the authors’ experience in the ﬁ eld indicates that predictable session tokens commonly arise from three different sources:

- Concealed sequences  
- Time dependency 
- Weak random number generation

## Concealed Sequences

There is still no visible pattern. However, if you subtract each number from the previous one, you arrive at the following:
```
FF97C4EB6A 
97C4EB6A 
FF97C4EB6A 
97C4EB6A 
FF97C4EB6A 
FF97C4EB6A
```
which immediately reveals the concealed pattern. The algorithm used to g enerate tokens adds 0x97C4EB6A to the previous value, truncates the result to a 32-bit number, and Base64-encodes this binary data to allow it to be transported using the text-based protocol HTTP. Using this knowledge, you can easily write a script to produce the series of tokens that the server will next produce, and the series that it produced prior to the captured sample.

## Time Dependency

```
3124553-1172764800468 
3124554-1172764800609 
3124555-1172764801109 
3124556-1172764801406 
3124557-1172764801703 
3124558-1172764802125 
3124559-1172764802500 
3124560-1172764802656 
3124561-1172764803125 
3124562-1172764803562
```
Comparing this second sequence of tokens with the ﬁrst, two points are immediately obvious: 
- The ﬁrst numeric sequence continues to progress incrementally; however, ﬁ ve values have been skipped since the end of the ﬁrst sequence. This is presumably because the missing values have been issued to other users who logged in to the application in the window between the two tests. 
- The second numeric sequence continues to progress by similar intervals as before; however, the ﬁrst value we obtain is a massive 539,578 greater than the previous value.

This second observation immediately alerts us to the role played by time in generating session tokens. Apparently, only ﬁ ve tokens have been issued between the two token-grabbing exercises. However, a period of approximately 10 minutes has elapsed. The most likely explanation is that the second number is time-dependent and is probably a simple count of milliseconds. Indeed, our hunch is correct. In a subsequent phase of our testing we perform a code review, which reveals the following token-generation algorithm
```
String sessId = Integer.toString(s_SessionIndex++) +    “-” +    System.currentTimeMillis()
```
Given our analysis of how tokens are created, it is straightforward to construct a scripted attack to harvest the session tokens that the application issues to other users: 
- We continue polling the server to obtain new session tokens in quick succession. 
- We monitor the increments in the ﬁ rst number. When this increases by more than 1, we know that a token has been issued to another user. 
- When a token has been issued to another user, we know the upper and lower bounds of the second number that was issued to that person, because we possess the tokens that were issued immediately before and after his. Because we are obtaining new session tokens frequently, the range between these bounds will typically consist of only a few hundred values. 
- Each time a token is issued to another user, we launch a brute-force attack to iterate through each number in the range, appending this to the missing incremental number that we know was issued to the other user. We attempt to access a protected page using each token we construct, until the attempt succeeds and we have compromised the user’s session. 
- Running this scripted attack continuously will enable us to capture the session token of every other application user. When an administrative user logs in, we will fully compromise the entire application

## Weak Random Number Generation

  Some of the algorithms used produce sequences that appear to be stochastic and manifest an even spread across the range of possible values. Nevertheless, they can be extrapolated forwards or backwards with perfect accuracy by anyone who obtains a small sample of values. When a predictable pseudorandom number generator is used to produce session tokens, the resulting tokens are vulnerable to sequencing by an attacker.
  Jetty is a popular web server written in 100% Java that provides a session management mechanism for use by applications running on it. In 2006, Chris Anley of NGSSoftware discovered that the mechanism was vulnerable to a  session token prediction attack. The server used the Java API java.util.Random to generate session tokens. This implements a “linear congruential generator,” which generates the next number in the sequence as follows:
```
synchronized protected int next(int bits) {      seed = (seed * 0x5DEECE66DL + 0xBL) & ((1L << 48) - 1);      return (int)(seed >>> (48 - bits)); }
```
  This algorithm takes the last number generated, multiplies it by a constant, and adds another constant to obtain the next number. The number is truncated to 48 bits, and the algorithm shifts the result to return the speciﬁ c number of bits requested by the caller. 
  Knowing this algorithm and a single number generated by it, we can easily derive the sequence of numbers that the algorithm will generate next. With a little number theory, we also can derive the sequence that it generated previously. This means that an attacker who obtains a single session token from the server can obtain the tokens of all current and future sessions.
  
## Test Quality Of Randomness

1. Determine when and how session tokens are issued by walking through the application from the first application page through any login functions. Two behaviors are common: 
- The application creates a new session anytime a request is received that does not submit a token. 
- The application creates a new session following a successful login.
To harvest large numbers of tokens in an automated way, ideally identify a single request (typically either GET / or a login submission) that causes a new token to be issued. 

2. In Burp Suite, send the request that creates a new session to Burp Sequencer, and configure the token’s location. Then start a live capture to gather as many tokens as is feasible. If a custom session management mechanism is in use, and you only have remote access to the application, gather the tokens as quickly as possible to minimize the loss of tokens issued to other users and reduce the influence of any time dependency. 

3. If a commercial session management mechanism is in use and/or you have local access to the application, you can obtain indefinitely large sequences of session tokens in controlled conditions

4. While Burp Sequencer is capturing tokens, enable the “auto analyse” setting so that Burp automatically performs the statistical analysis periodically. Collect at least 500 tokens before reviewing the results in any detail. If a sufficient number of bits within the token have passed the tests, continue gathering tokens for as long as is feasible, reviewing the analysis results as further tokens are captured. 

5. If the tokens fail the randomness tests and appear to contain patterns that could be exploited to predict future tokens, reperform the exercise from a different IP address and (if relevant) a different username. This will help you identify whether the same pattern is detected and whether tokens received in the first exercise could be extrapolated to identify tokens received in the second. Sometimes the sequence of tokens captured by one user manifests a pattern. But this will not allow straightforward extrapolation to the tokens issued to other users, because information such as source IP is used as a source of entropy (such as a seed to a random number generator). 

6. If you believe you have enough insight into the token generation algorithm to mount an automated attack against other users’ sessions, it is likely that the best means of achieving this is via a customized script. This can generate tokens using the specific patterns you have observed and apply any necessary encoding. See Chapter 14 for some generic techniques for applying automation to this type of problem. 

7. If source code is available, closely review the code responsible for generating session tokens to understand the mechanism used and determine whether it is vulnerable to prediction. If entropy is drawn from data that can be determined within the application within a brute-forcible range, consider the practical number of requests that would be needed to bruteforce an application token.

## Disclosure of Tokens on the Network

1. Walk through the application in the normal way from first access (the “start” URL), through the login process, and then through all of the application’s functionality. Keep a record of every URL visited, and note every instance in which a new session token is received. Pay particular attention to login functions and transitions between HTTP and HTTPS communications. This can be achieved manually using a network sniffer such as Wireshark or partially automated using the logging functions of your intercepting proxy, as shown in Figure 7-10.

2. If HTTP cookies are being used as the transmission mechanism for session tokens, verify whether the secure flag is set, preventing them from ever being transmitted over unencrypted connections. 

3. Determine whether, in the normal use of the application, session tokens are ever transmitted over an unencrypted connection. If so, they should be regarded as vulnerable to interception. 

4. Where the start page uses HTTP, and the application switches to HTTPS for the login and authenticated areas of the site, verify whether a new token is issued following login, or whether a token transmitted during the HTTP stage is still being used to track the user’s authenticated session. Also verify whether the application will accept login over HTTP if the login URL is modified accordingly

5. Even if the application uses HTTPS for every page, verify whether the server is also listening on port 80, running any service or content. If so, visit any HTTP URL directly from within an authenticated session, and verify whether the session token is transmitted. 
 
6. In cases where a token for an authenticated session is transmitted to the server over HTTP, verify whether that token continues to be valid or is immediately terminated by the server.

## Disclosure of Tokens in Logs

1. Identify all the functionality within the application, and locate any logging or monitoring functions where session tokens can be viewed. Verify who can access this functionality–for example, administrators, any authenticated user, or any anonymous user. See Chapter 4 for techniques for discovering hidden content that is not directly linked from the main application. 

2. Identify any instances within the application where session tokens are transmitted within the URL. It may be that tokens are generally transmitted in a more secure manner but that developers have used the URL in specific cases to work around particular difficulties. For example, this behavior is often observed where a web application interfaces with an external system. 

3. If any session tokens are captured, attempt to hijack user sessions by using the application as normal but substituting a captured token for your own. You can do this by intercepting the next response from the server and adding a Set-Cookie header of your own with the captured cookie value. In Burp, you can apply a single Suite-wide configuration that sets a specific cookie in all requests to the target application to allow easy switching between different session contexts during testing. 
 
## Vulnerable Mapping of Tokens to Sessions

