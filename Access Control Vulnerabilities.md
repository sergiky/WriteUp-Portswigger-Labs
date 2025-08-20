
- [What are Access control vulnerabilities?](#what-are-access-control-vulnerabilities)
	- [Vertical access controls](#vertical-access-controls)
	- [Horizontal access controls](#horizontal-access-controls)
	- [Context-dependent access control](#context-dependent-access-control)
- [Unprotected admin functionality](#unprotected-admin-functionality)
- [Unprotected admin functionality with unpredictable URL](#unprotected-admin-functionality-with-unpredictable-url)
- [User role controlled by request parameter](#user-role-controlled-by-request-parameter)
- [User role can be modified in user profile](#user-role-can-be-modified-in-user-profile)
- [User ID controlled by request parameter](#user-id-controlled-by-request-parameter)
- [User ID controlled by request parameter, with unpredictable user IDs](#user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
- [User ID controlled by request parameter with data leakage in redirect](#user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
- [User ID controlled by request parameter with password disclosure](#user-id-controlled-by-request-parameter-with-password-disclosure)
- [Insecure direct object references](#insecure-direct-object-references)
- [URL-based access control can be circumvented](#url-based-access-control-can-be-circumvented)
- [Method-based access control can be circumvented](#method-based-access-control-can-be-circumvented)
- [Multi-step process with no access control on one step](#multi-step-process-with-no-access-control-on-one-step)
- [Referer-based access control](#referer-based-access-control)
- [Location-based access control](#location-based-access-control)



# What are Access control vulnerabilities?

Access control is the application that is authorized to perform actions or access resources. In web applications, access control is dependent on authentication and session management:

- **Authentication** confirms that the user is who they say they are.
- **Session management** identifies which subsequent HTTP requests are being made by that same user.
- **Access control** determines whether the user is allowed to carry out the action that they are attempting to perform.

**Broken access controls** are common and often present a critical security vulnerability.

## Vertical access controls

Vertical access controls are mechanism that restrict access to sensitive functionallity to **specific type of users**.

For example, an administrator might be able to modify or delete any user's account, but an ordinary user has not access to these action.

## Horizontal access controls

Horizontal access controls are mechanisms that restrict access to resources to **specific users**.

For exmaple, a banking application will allow a user to view transactions and make payments from their own accounts, but not the accounts of any other user.

## Context-dependent access control

Context-dependent access controls restrict access to functionality and resources based upon the state of the application or the user's interaction with it. Control prevent a user performing actions in the wrong order.

For example, a retail website might prevent users from modifying the contents of the ir shopping cart after they have made payment.

---

# Unprotected admin functionality

In this lab we have a [[#Vertical access controls]], a normal user can realize administrator tasks.

Sometimes administrators have pages like `/admin` which allows them to do privilage actions.

To find this type of pages we have two ways, search in other locations that may be disclosed like `robots.txt` or bruteforce the URL with some wordlist.

In this case, if we see the **robots.txt** file we found the **administrator-panel** page where doesn't check if the user is an administrator or not. Here, we can delete the Carlos account and solve the lab.

# Unprotected admin functionality with unpredictable URL

In some cases, some servers gives a less predictable URL. This is an example of **security by obscurity**. However, the obfuscated URL can be discover in a number of ways.

In this lab if you inspect the source code you find that the page is showing in JavaScript code.

```js
<script>
	var isAdmin = false;
	if (isAdmin) {
	    var topLinksTag = document.getElementsByClassName("top-links")[0];
	    var adminPanelTag = document.createElement('a');
	    adminPanelTag.setAttribute('href', '/admin-ceq4b9');
	    adminPanelTag.innerText = 'Admin panel';
	    topLinksTag.append(adminPanelTag);
	    var pTag = document.createElement('p');
	    pTag.innerText = '|';
	    topLinksTag.appendChild(pTag);
	}
</script>
```

# User role controlled by request parameter

Parameter-based access control methods.

Some applications determine the user's access rights or role at login, and then store this information in a user controllable location. This could be:
- A hidden field.
- A cookie.
- A preset query string parameter.

Some example in parameters:

```
https://insecure-website.com/login/home.jsp?admin=true 
https://insecure-website.com/login/home.jsp?role=1
```

In this if we login and see the request in HTTP History (Burp) we see two cookies:

```
Cookie: session=PCK3aAM2NBC79kyf1Qix1gwQ5qEcsGyr; Admin=false
```

If you send to the repeater and try to change the value of Admin cookie to true, it doesn't work, the first think is because have a csrf token that change for every login request. The next idea it would be to intercept the request, change the value, and then right click > Do intercept > Response to this request, forward, change the value of the admin cookie again in the redirect request, forward, forward the last request and in your browser you see a link "Admin Panel".

Other way to do it is that this cookies are dragged into the rest of the requests, for example in **/my-account** you can modify the admin cookie. You can use BurpSuite or the **Storage tool** of the browser to modify the value of the cookie and reload the page.

# User role can be modified in user profile

Parameter-based access control methods.

If intercept the request when change the email of wiener user, you see that in the response have different values like username, email, apikey and **roleid**.

In the description , it only tells us that the page /admin is accessible with a roleid of 2.

We can intercept the request, send to repeater and add the roleid.

```json
{"email":"test3@test.com",
"roleid":2}
```

We see that works and this value is stored, if we reload the website we see the admin panel.

# User ID controlled by request parameter

If you active the proxy for burp and login with wiener you will see this url:
https://0a64004c047c7eff80cc4e09006f0098.web-security-academy.net/my-account?id=wiener

In the response you see the API key what happens if you try yo change wiener to "carlos", you obtain a different API key, in this case the API key of carlos.

# User ID controlled by request parameter, with unpredictable user IDs

Now in place of the name we have GUID of a user, example: 946f1849-82d2-4bec-852d-0df9645125fa

Now how we have to discover the GUID of the Carlos users.
If we start exploring the page we see that in the comment section of some posts carlos write some comments. This link contain the userId that have the same format that the GUID of the user.

We can try to replace in `/my-account?id=x` and see if this works.

# User ID controlled by request parameter with data leakage in redirect

In this case if you try to change wiener with carlos user: `/my-account?id=carlos` the page will redirect to login page, but if you see the response in http history you will see that the redirect response is leaking the API Key of the user carlos and their personal information.

# User ID controlled by request parameter with password disclosure

In this when we log in with wiener:peter we see that have an input field of type password with the password in the input attribute, this means that we can see in plain text the password of the user.

What happens if we try to change the user carlos to administrator -> `/my-account?id=administrator`. We see another password, the password of the admin user. Now we can log in like admin user and delete the Carlos account.

# Insecure direct object references

In this case we have a live chat functionality and we can download the transcript. In HTTP history we see that we're downloading `/download-transcript/2.txt`, but what happens if we try to change two for one. We see other transcription(another conversation) that contain the password, we can try to log in with the user carlos. This is an IDOR vulnerability.

# URL-based access control can be circumvented

Some reverse proxies(nginx), WAFs or frameworks(ASP .NET, IIS) allow the header `X-Original-URL` or `X-Rewrite-URL, 

The original propose of this header is preserve the original URL that the client request before the proxy or middleware modify them.

This can ignore the original URL and use the new one in X-Original-URL.

I recommend you to remove the original path and use `/` to avoid confusion.

Example:
```http
GET / HTTP/2
Host: sitio.com
X-Original-URL: /notexist
```

If the response change to something like not found, it's a good signal, now, we can try to indicate the forbidden page.

```http
GET / HTTP/2
Host: sitio.com
X-Original-URL: /admin
```

In my case I'm using the intercept and the forward button.

The next step is to delete Carlos account, but if we click appear this message: "Access denied". This is because the header is not dragging.

Another problem is that this header can't use parameters, you have to use something like this:

```http
GET /?username=carlos HTTP/2
Host: 0a6500e304b9f4e68020d5dd000d0031.web-security-academy.net
X-Original-Url: /admin/delete
```

At the final you will see the error "Access denied" (because you don't include the header and you're in /admin page), but if you go to the main page you see that solve the lab.

# Method-based access control can be circumvented

If we access with the administrator we can upgrade or downgrade users, but this is possible because the admin user have the admin rol that allow to do this, if we try to do with another user doesn't work because doesn't have the necessary permissions.

If you try to change the username to wiener when upgrade the user you will see that doesn't work.

If you try to change the session cookie with the wiener cookie doesn't work either. This is because wiener is not an admin user.

Mmm we can try to bypass this restriction, one basic way is to change the request method, POST to GET, to check if the server check both methods.

Now with the session cookie, the username=wiener and GET method we can send the request:

```http
GET /admin-roles?username=wiener&action=upgrade
```

Now, we do a vertical privilege escalation and the user wiener is part of the administrators.

# Multi-step process with no access control on one step

Log in like administrator user, when we upgrade an user we do a post request, but have a button asking us if we are sure we want to do it. When click in that button do another POST request. 

The idea of the first POST request is to valid that the user is sure about upgrade or downgrade an user.

Then do a second request that do the action. But what happens if we do only the last request? The developer imagine that user have to do the both to complete the action, but we can only do the second request, thus avoiding the first.

If we send the requests to the repeater, log in with wiener user, obtain the session cookie, replace in the second request and check that the username that you're going to upgrade is wiener. Now we see that we solve the lab and we are an administrator.

# Referer-based access control

In this case the lab check that the Referer header point to /admin.
If you don't have this header the request in this case via GET is not going to work.

# Location-based access control

Some websites enforce access controls based on the user's geographical location. This can apply, for example, to banking applications or media services where state legislation or business restrictions apply. These access controls can often be circumvented by the use of web proxies, VPNs, or manipulation of client-side geolocation mechanisms.
