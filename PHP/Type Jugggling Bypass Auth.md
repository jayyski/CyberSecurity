# Type Juggling

When using the "==" operator, PHP attempts something called loose comparison (or 'type juggling'). With loose comparison, it's possible for a PHP developer to compare values even if they have a different data type, such as integers and strings.

```
'test' == 0
'test' == TRUE
```

Testing both of these comparisons would return true.

The most common way that this particularity in PHP is exploited is by using it to bypass authentication.
If the PHP code handles authentication looks like this:
```
$_POST["password"] == "Password"
```
Then, submitting an integer input of 0 would successfully log you in as admin, since this will evaluate to True:
```
0 == "Password" -> True
```

## To avoid type juggling issues

When comparing values, always try to use the type-safe comparison operator “===” instead of the loose comparison operator “==”. This will ensure that PHP does not type juggle and the operation will only return True if the types of the two variables also match.
```
0 === "0" -> False
```
