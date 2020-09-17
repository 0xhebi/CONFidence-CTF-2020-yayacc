<h2>Yet Another Yet Another Cat Challenge</h2>

<h3>Description</h3>

<blockquote>Yet Another Cat Challenge for you! Just another boring add/remove/share note app, but with 8 different cats!

http://yayacc.zajebistyc.tf</blockquote>



<p>One day when I was chatting with my friend <a href="https://gynvael.coldwind.pl/">Gynvael</a>, he told me about one hard XSS challenge from CONFidence CTF 2020 that he faced. So I decided to give it a try and see what is going on, two hints that he gave me were :</p>
<ol>
 <li>HTML injection</li> <li>Theme selection where XSS will happen.(Tho there is intended and unintended solution, we will cover both of them)</li></ol><br>
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

<p>First step is HTML injection which we can achieve by changing the <code>select</code> tag to either input or textbox for cat selection. <b>Note that both solution intended and uninteded will require those steps.</b> </p>